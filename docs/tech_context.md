# Technical Context

## Core Technologies

### Framework
- **Django 4.x** - Main web framework
- **Django REST Framework** - API development
- **Celery** - Asynchronous task processing
- **PostgreSQL** - Primary database (multiple databases: default, qa-crawler, platform-replica)

### Authentication & Authorization

#### Auth0 Integration (`auth0authorization` app)
- **Provider**: Auth0 OAuth2/OpenID Connect
- **Token Type**: JWT (JSON Web Tokens)
- **Algorithm**: RS256 (asymmetric) with JWKS validation
- **Token Validation**: 
  - Production: Fetches JWKS from Auth0 (cached 1 hour)
  - Test: HS256 with shared secret for faster tests

#### Authentication Flow
1. Frontend obtains JWT from Auth0
2. Client sends JWT in `Authorization: Bearer {token}` header
3. `rest_framework_jwt` middleware validates token
4. Custom `jwt_decode_token()` verifies signature via JWKS
5. Custom `jwt_get_username_from_payload_handler()` extracts user info
6. `SaveUserInfoRemoteUserBackend` creates/updates Django User
7. Request proceeds with authenticated user

#### User Provisioning
- Users auto-created from Auth0 tokens
- Username: Auth0 `sub` claim with pipe-to-dot transformation (`auth0|123` → `auth0.123`)
- Profile data: Email, first name, last name extracted from token claims

#### Permission System
- Django's built-in permission framework
- Custom app-level permissions (e.g., `qa_crawler.qa_crawler`, `digital_pr.digital_pr`)
- `FullDjangoModelPermissions`: Extends DRF's permissions to include view/read permissions
- Platform users: Special restrictions (no DELETE operations)

#### Multiple Auth Methods Supported
1. **JWT (Primary)**: Auth0 tokens for user authentication
2. **Token Auth**: DRF tokens for platform-to-platform API calls
3. **Session Auth**: Django sessions for admin interface
4. **Basic Auth**: Fallback for development/testing

### Development Setup

#### Docker-based Development
- `docker-compose.yml` orchestrates all services
- Hot reload enabled for code changes
- Separate test environment configuration

### External Integrations
#### Observability
- **OpenTelemetry**: Distributed tracing
- Performance monitoring on critical paths (JWT validation, JWKS fetching)
- Cache hit/miss tracking

#### Caching
- Django cache framework for JWKS (1-hour TTL)
- Reduces Auth0 API calls from ~every request to ~1/hour

### Technical Constraints

#### Authentication Requirements
- All API endpoints protected by JWT auth (except documented exceptions)
- Tokens must be obtained from Auth0
- Local development uses staging Auth0 application

#### Database Access
- Multi-database routing configured
- Some apps use dedicated databases (qa-crawler, platform-replica)
- Tests must declare all required databases in pytestmark

#### Environment-Specific Behavior
- Test environment uses simplified JWT validation (no external Auth0 calls)
- Production requires valid Auth0 configuration (AUTH_DOMAIN, AUTH_AUDIENCE)

### Dependencies

#### Key Python Packages
- `rest-framework-jwt` - JWT authentication for DRF
- `PyJWT` - JWT token encoding/decoding
- `requests` - HTTP client for JWKS fetching
- `opentelemetry-api` - Observability/tracing

#### Auth0 Configuration
Settings required:
- `AUTH_DOMAIN` - Auth0 tenant domain
- `AUTH_AUDIENCE` - API audience identifier
- `TEST_JWT_KEY` - Secret for test environment (HS256)

