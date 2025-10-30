# Code Conventions

## File Organization

### Standard App Structure
Each Django app follows this file layout:
- `models.py` - Database models
- `serializers.py` - DRF serializers  
- `views.py` - ViewSets and API views
- `urls.py` - URL routing
- `permissions.py` - Custom permission classes
- `tasks.py` - Celery tasks
- `signals.py` - Django signals
- `admin.py` - Django admin registration
- `enums.py` - Django TextChoices enums
- `filters.py` - django-filter FilterSet classes
- `pagination.py` - DRF pagination classes
- `queries.py` - Reusable annotated querysets
- `startup.py` - post_migrate initialization functions
- `imports.py` - CSV import functions
- `exports.py` - CSV export functions
- `constants.py` - Module-level constants
- `helpers/` - App-specific utility functions
- `services/` - Business logic (scheduler, fetchers, publishers, etc)

## Naming Conventions

### Models
- All models inherit from `shared.models.BaseModel`
- Soft-deletable models also inherit from `shared.models.NonDeletable`
- Profile pattern: `UserProfile` and `ClientProfile` models (one-to-one with User/Website)

### Serializers
- Standard: `{Model}Serializer`
- Platform API (excludes sensitive fields): `{Model}PlatformSerializer`
- Admin-only fields: `{Model}AdminSerializer`
- Nested relationships (cross-app shared): `Short{Model}Serializer`
- Performance-optimized variant (same app): `Minimal{Model}Serializer`

### ViewSets & Views
- `{Model}ViewSet` for standard CRUD
- Use `APIView` for custom non-CRUD endpoints

### Permissions
- Format: `Has{Feature}Permission`
- Variants: `Has{Feature}AdminPermission`, `Has{Feature}FixerPermission`

### Celery Tasks
- Always use explicit name: `@shared_task(name="task_name")`
- Naming pattern: `verb_noun` (e.g., `trigger_qa_crawl`, `process_error`)

### URLs
- Router basenames: kebab-case (e.g., `"client-profiles"`, `"outreach-plans"`)
- Custom paths: kebab-case with slashes
- Analytics endpoints: prefix with `analytics/`

### Constants
- Use `SCREAMING_SNAKE_CASE` for module-level constants
- Define in `constants.py` file

## Pagination Classes

Standard pagination sizes duplicated per app:
- `LargeResultsSetPagination` - 1000 items (max 10000)
- `StandardResultsSetPagination` - 100 items (max 1000)  
- `SmallResultsSetPagination` - 10 items (max 50)

All use `page_size_query_param = "page_size"` for client override.

## Startup Functions

Pattern for `startup.py`:
```python
def create_resources(apps, app_config, **kw):
    # Initialization logic
    pass
```

- Functions accept `(apps, app_config, **kw)` signature
- Common names: `create_{resource}`, `schedule_{task}`, `set_{property}`
- Connected in `apps.py` `ready()` method via `post_migrate.connect()`

## Signal Registration

Signals must be imported in `apps.py`:
```python
def ready(self):
    from . import signals  # Import triggers registration
```

## Custom Actions

Use explicit `url_path` for ViewSet custom actions:
```python
@action(detail=False, methods=["post"], url_path="import-csv")
def import_csv(self, request):
    ...
```

## Profile Pattern

Feature-specific user/client data uses Profile models:
- `UserProfile` - one-to-one with Django User
- `ClientProfile` - one-to-one with Website
- Created via signals on User/Website creation
- Initialization via `startup.py` for existing records

## Import/Export Functions

- Define in standalone `imports.py` / `exports.py` files
- Use `csv.QUOTE_ALL` for CSV operations
- Import functions return Response with errors list
- Export functions return HttpResponse with CSV attachment

## Hierarchical Tree Endpoints

**Pattern for practice area trees**:
```python
# In ViewSet
tree_node_serializer_class = TreeNode{Model}Serializer

@action(detail=False, methods=["get"], url_path="tree", pagination_class=None)
def tree(self, request, *args, **kwargs):
    queryset = self.filter_queryset(self.get_queryset())
    queryset = queryset.order_by("practice_area_depth")
    tree_dict = build_page_like_tree(queryset, self.tree_node_serializer_class)
    return Response(tree_dict)
```

**Requirements**:
- Model must inherit from `PageLike` abstract model
- Base query in `queries.py` must include `practice_area_depth` annotation
- Tree node serializer includes minimal fields only
- No pagination on tree endpoints (returns full hierarchy)
- All standard filters still apply via `filter_queryset()`

**Naming**:
- ViewSet attribute: `tree_node_serializer_class`
- Serializer: `TreeNode{Model}Serializer`
- URL path: `"tree"` → `/api/{model}/tree/`
