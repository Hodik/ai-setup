# System Patterns

## Architecture Overview

This is a Django REST Framework API serving as the backend for Hennessey Dashboard 2.0. The system follows a modular app-based architecture with shared authentication and permissions.

### Core Apps Structure
- **auth0authorization** - Authentication and user management
- **client_content** - Client content management
- **competitor_content** - Competitor analysis
- **digital_pr** - Digital PR campaigns
- **qa_crawler** - Quality assurance crawling
- **content_reports** - Content analytics and reporting
- **platform_management** - Platform-level configuration
- **shared** - Shared utilities, permissions, and mixins

## Authentication Patterns

### JWT Authentication Flow

**Primary Pattern**: Hybrid Auth0 + Django authentication

```python
# Token arrives from frontend
Authorization: Bearer <auth0_jwt_token>

# DRF JWT middleware processes token
rest_framework_jwt.authentication.JSONWebTokenAuthentication

# Custom decoder validates with JWKS
auth0authorization.utils.jwt_decode_token(token)
  → Fetches JWKS from Auth0 (cached)
  → Verifies RS256 signature
  → Returns decoded payload

# Custom payload handler provisions user
auth0authorization.utils.jwt_get_username_from_payload_handler(payload)
  → Transforms Auth0 sub claim (pipe → dot)
  → Calls authenticate() with user attributes
  
# Backend creates/updates Django user
auth0authorization.backends.SaveUserInfoRemoteUserBackend
  → get_or_create User with username
  → Updates profile fields (email, first_name, last_name)
  → Returns authenticated user
```

### User Auto-Provisioning

**Pattern**: Just-in-time user creation from Auth0 tokens

**Key Implementation**:
```python
# backends.py
def configure_user(self, request, user, created=True, **user_attrs):
    if user_attrs:
        for k, v in user_attrs.items():
            setattr(user, k, v)
        user.save()
    return user
```

**Why**: Eliminates need for separate user registration flow. Auth0 is source of truth for identity, Django manages authorization.

## Permission Patterns

### App-Level Custom Permissions

**Pattern**: Each feature module defines custom permissions checked via `user.has_perm()`

**Examples**:
```python
# qa_crawler/permissions.py
class HasQACrawlerPermission(BasePermission):
    def has_permission(self, request, view):
        return request.user.has_perm("qa_crawler.qa_crawler")

# digital_pr/permissions.py  
class HasDigitalPRPermission(BasePermission):
    def has_permission(self, request, view):
        return request.user.has_perm("digital_pr.digital_pr")
```

**Why**: Provides coarse-grained feature access control. Admins grant permissions via Django admin to control which features users can access.

### Full Model Permissions

**Pattern**: Extended DRF model permissions including view/read operations

```python
# shared/permissions.py
class FullDjangoModelPermissions(DjangoModelPermissions):
    perms_map = {
        "GET": ["%(app_label)s.view_%(model_name)s"],
        "OPTIONS": ["%(app_label)s.view_%(model_name)s"],
        "HEAD": ["%(app_label)s.view_%(model_name)s"],
        "POST": ["%(app_label)s.add_%(model_name)s"],
        "PUT": ["%(app_label)s.change_%(model_name)s"],
        "PATCH": ["%(app_label)s.change_%(model_name)s"],
        "DELETE": ["%(app_label)s.delete_%(model_name)s"],
    }
```

**Why**: DRF's default `DjangoModelPermissions` doesn't require permissions for GET requests. This pattern ensures read access is also permission-controlled.

### Platform User Pattern

**Pattern**: Special user type for platform-to-platform API communication

**Implementation**:
```python
# Identified by email header
platform_email = request.headers.get('X-Platform-Email')

# Special permissions
class PlatformWebsiteAccessPermission(BasePermission):
    """Platform users can only access their assigned websites"""

class PlatformMethodPermission(BasePermission):  
    """Platform users cannot DELETE resources"""
```

**Flow**:
1. Platform user authenticates with DRF Token (not JWT)
2. Request includes `X-Platform-Email` header
3. Middleware stores email in thread-local storage
4. Custom permissions enforce website-level access control
5. DELETE operations blocked for platform users

**Why**: Enables external platforms to integrate with limited, scoped access. Website-level isolation prevents cross-client data access.

## Serializer Patterns

### User Serializers

**Two serializer pattern for users**:

```python
# Full user with permissions (for auth endpoints)
class UserSerializer(serializers.ModelSerializer):
    permissions_names = serializers.SerializerMethodField()
    # Returns: ["app_label | permission_name", ...]

# Minimal user (for nested relationships)  
class ShortUserSerializer(serializers.ModelSerializer):
    # Just: id, first_name, last_name, email
```

**Usage**: `ShortUserSerializer` widely imported for showing user info in related objects (created_by, assigned_to, etc.)

**Why**: Avoids N+1 queries and massive payloads when serializing lists of objects with user relationships.

### Minimal Serializers for Performance

**Pattern**: Query-parameter-driven serializer selection for high-volume endpoints

```python
# ViewSet with minimal serializer variant
class RecommendationViewSet(ModelViewSet):
    serializer_class = RecipeCardRecommendationSerializer  # Full data
    minimal_serializer_class = MinimalRecipeCardRecommendationSerializer
    
    def get_serializer_class(self):
        variant = self.request.query_params.get("variant", "full")
        if variant == "minimal":
            return self.minimal_serializer_class
        return super().get_serializer_class()

# Full serializer (22+ fields, nested relationships)
class RecipeCardRecommendationSerializer(BaseRecipeCardRecommendationSerializer):
    crawl_result = RecipeCardCrawlResultSerializer(required=False)  # 70+ fields
    class Meta:
        fields = "__all__"

# Minimal serializer (5 essential fields only)
class MinimalRecipeCardRecommendationSerializer(BaseRecipeCardRecommendationSerializer):
    crawl_result = MinimalRecipeCardCrawlResultSerializer(required=False)  # 4 fields
    class Meta:
        fields = ["id", "new_url", "new_title", "new_h1", "crawl_result"]
```

**Usage**: Clients request via `?variant=minimal` for list views, `?variant=full` (default) for detail views.

**Why**: 
- Reduces payload size by ~85-90% for large paginated lists (1000+ items)
- Paired with `LargeResultsSetPagination` for bulk data transfer
- Shared validation logic in base serializer prevents duplication
- Client-controlled vs server-enforced (unlike `PlatformSerializerMixin`)

**Naming Convention**: Use `Minimal{Model}Serializer` prefix (vs `Short{Model}Serializer` for cross-app shared serializers)

### Permission Aggregation

**Pattern**: Single method to aggregate user + group permissions

```python
def get_permissions_names(self, obj):
    if obj.is_superuser:
        permissions = Permission.objects.all()
    else:
        permissions = Permission.objects.filter(
            Q(user=obj) | Q(group__user=obj)
        ).distinct()
    return [f"{p.content_type.app_label} | {p.name}" for p in permissions]
```

**Why**: Frontend needs to know what features to show/hide. Single query with `.distinct()` handles both direct user permissions and group-based permissions efficiently.

## Middleware Patterns

### Request Context Middleware

**Pattern**: Store request in thread-local storage for access in signals/services

```python
# dashboard/middlewares.py
class CurrentUserAndPlatformEmailMiddleware:
    def process_request(self, request):
        _requests[current_thread().ident] = request
        _requests["platform_email"] = request.headers.get("X-Platform-Email")
        
    def process_response(self, request, response):
        _requests.pop(current_thread().ident, None)
        
# Accessed via helpers
def current_request_user() -> AbstractUser:
    return current_request().user
```

**Why**: Signals and background services need user context but don't receive request objects. Thread-local storage provides global access while maintaining thread safety in Django's request-per-thread model.

## Testing Patterns

### Multi-Database Test Marking

**Required Pattern**:
```python
pytestmark = pytest.mark.django_db(
    databases=["default", "qa-crawler", "platform-replica"]
)
```

**Why**: System uses multiple databases. Tests must declare all DBs they touch, otherwise transactions fail.

### Authenticated Test Client

**Pattern**: Conftest provides authenticated client fixture

```python
# Tests use client fixture which is pre-authenticated
def test_api_endpoint(client):
    resp = client.get("/api/resource/")
    assert resp.status_code == 200
```

**Why**: Most endpoints require authentication. Default fixture eliminates auth boilerplate in every test.

### Simplified JWT for Tests

**Pattern**: Test environment uses HS256 instead of RS256

```python
# dashboard/settings/test.py
JWT_AUTH = {
    "JWT_DECODE_HANDLER": "auth0authorization.utils.test_jwt_decode_token",
    "JWT_ALGORITHM": "RS256",  
}

# auth0authorization/utils.py
def test_jwt_decode_token(token):
    return jwt.decode(token, key=settings.TEST_JWT_KEY, algorithms=["HS256"])
```

**Why**: Avoids external Auth0 API calls during tests. Tests run faster and don't depend on Auth0 availability.

## Key Architectural Decisions

### Why Auth0 + Django Hybrid?

**Decision**: Use Auth0 for authentication, Django for authorization

**Trade-offs**:
- ✅ Auth0 handles identity, MFA, social login complexity
- ✅ Django handles fine-grained permissions via admin UI
- ✅ Auto-provision users on first login (no registration flow)
- ⚠️ Two systems to manage (Auth0 users + Django permissions)
- ⚠️ Auth0 costs scale with user count

### Why JWKS Caching?

**Decision**: Cache JWKS for 1 hour despite security recommendation for fresh fetch

**Trade-offs**:
- ✅ Massive performance gain (~200ms per request → amortized)
- ✅ Reduced Auth0 API usage (cost savings)
- ⚠️ Key rotations take up to 1 hour to propagate
- ✅ Auth0's best practice: 1 hour is acceptable cache duration

### Why Multiple Authentication Methods?

**Decision**: Support JWT, Token, Session, and Basic auth simultaneously

**Trade-offs**:
- ✅ JWT for frontend SPA authentication
- ✅ Token auth for platform-to-platform APIs
- ✅ Session auth for Django admin interface
- ✅ Basic auth for development/debugging
- ⚠️ More attack surface to secure
- ⚠️ More authentication code paths to maintain

## Data Visualization Patterns

### Hierarchical Tree Builder

**Pattern**: Generic utility for building practice area trees from PageLike models

**Implementation**:
```python
# recipe_card/helpers/tree_builder.py
def build_page_like_tree(
    queryset: QuerySet[PageLike], 
    serializer_class: ModelSerializer
) -> Dict[str, Dict]:
    """
    Returns nested dict keyed by practice area + city:
    {
        "Personal Injury:New York": {
            "id": ...,
            "city": "New York", 
            "children": {
                "Personal Injury||Car Accident:New York": {
                    "id": ...,
                    "children": {}
                }
            }
        }
    }
    """
```

**Usage in ViewSets**:
```python
class RecommendationViewSet(ModelViewSet):
    tree_node_serializer_class = TreeNodeRecipeCardRecommendationSerializer
    
    @action(detail=False, methods=["get"], url_path="tree", pagination_class=None)
    def tree(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())
        queryset = queryset.order_by("practice_area_depth")
        tree_dict = build_page_like_tree(queryset, self.tree_node_serializer_class)
        return Response(tree_dict)
```

**Key Requirements**:
1. QuerySet must contain `practice_area_depth` annotation (computed in queries.py)
2. Models must inherit from `PageLike` abstract base
3. Queryset ordered by `practice_area_depth` ascending (depth 1 first)
4. Tree serializer class provides minimal fields for efficiency

**Why Dict Structure**:
- Frontend tree components consume nested dict format naturally
- O(1) key lookups during tree building (vs O(n) array traversal)
- Automatic duplicate handling via `num_duplicates` counter
- Creates empty parent nodes when children exist but parent doesn't

**Apps Using This Pattern**:
- `recipe_card`: Recommendations tree by practice area + city
- `content_gap`: Pages tree by practice area + city (same taxonomy)

## Component Relationships

### Critical Authentication Path

```
Client Request → MIDDLEWARE:
  1. SessionMiddleware (sessions)
  2. CorsMiddleware (CORS headers)
  3. AuthenticationMiddleware (Django auth)
  4. CurrentUserAndPlatformEmailMiddleware (thread-local storage)

→ DRF VIEW → AUTHENTICATION CLASSES:
  1. JSONWebTokenAuthentication (primary)
  2. TokenAuthentication (platform users)
  3. SessionAuthentication (admin)
  4. BasicAuthentication (fallback)

→ PERMISSION CLASSES:
  - IsAuthenticated (default global)
  - Custom app permissions (per-view)
  - FullDjangoModelPermissions (per-view)
  - PlatformWebsiteAccessPermission (platform users)

→ View Logic → Response
```

### Cross-App Dependencies

**`ShortUserSerializer` Usage**:
- `client_content.serializers`
- `digital_pr.serializers`
- `qa_crawler.serializers`
- `recipe_card.serializers`
- `competitor_content.serializers`

**Pattern**: Widely shared serializer reduces duplication but creates tight coupling to auth0authorization app.

**`PageLike` Abstract Model**:
- Defined in `recipe_card.models`
- Inherited by `recipe_card.Recommendation` and `content_gap.Page`
- Provides common interface: practice area hierarchy (4 levels) + location (city, state)
- Enables generic tree builder utility to work across both models

**Pattern**: Abstract model in feature app (not shared/) for small cross-app interfaces. Creates coupling but avoids over-engineering.

### Permission Service Layer

```
shared/services/platform_permissions.py
  ↓
shared/permissions.py (PlatformWebsiteAccessPermission)
  ↓
View mixins (PlatformPermissionMixin)
  ↓
Individual views
```

**Pattern**: Centralized permission logic in service layer, consumed by permission classes, applied via mixins.

