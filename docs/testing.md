# Testing Guidelines

## Writing Tests

**ALWAYS include:**

```python
pytestmark = pytest.mark.django_db(
    databases=["default", "qa-crawler", "platform-replica"]
)
```

**Structure:**

- Unit tests: `tests/unit/test_*.py` - isolated logic, mock all dependencies
- Integration tests: `tests/integration/test_*.py` - full API workflows
- Small apps: flat `tests/test_*.py` structure is acceptable

## Unit vs Integration Tests

### Unit Tests

- Purpose: test a single function, method, or component in isolation
- Speed: should run instantly (< 100ms per test)
- Database: use `pytest.mark.django_db` transactional database when needed
- Mocking: mock all external services and side effects (APIs, HTTP, Google, Asana, WordPress)
- Location: `tests/unit/test_<module>.py`

### Integration Tests

- Purpose: test full workflows through API endpoints
- Speed: slower; uses real database and realistic flows
- Mocking: only mock external services (e.g., Google, Asana, WordPress)
- Location: `tests/integration/test_<feature>.py`
- What to test:
  - Complete CRUD workflows
  - Authentication and permissions
  - Side effects (history events, signals, tasks)
  - Related object creation/updates
  - Filters, ordering, pagination

**Required imports:**

```python
import pytest
from faker import Faker

fake = Faker()
```

**Fixtures:**

- Use `client` fixture for authenticated API testing (auto-injected from conftest.py)
- Use `user` fixture for authenticated user
- Use `monkeypatch` for mocking functions
- Create local fixtures for test-specific setup

**Mocking external services:**

```python
# Always mock: Google Docs/Drive, Asana, WordPress, external APIs
def test_example(monkeypatch):
    mock_fn = MagicMock()
    mock_fn.return_value = expected_value
    monkeypatch.setattr(module, 'function_name', mock_fn)
```

**Testing async tasks:**

```python
def test_celery_task(monkeypatch):
    background_tasks = []
    def mock_delay(*args, **kwargs):
        background_tasks.append((task_fn, args, kwargs))
    monkeypatch.setattr(task_fn, 'delay', mock_delay)

    # ... trigger action ...

    # Execute synchronously
    for task, args, kwargs in background_tasks:
        task(*args, **kwargs)
```

## Running Tests

be efficient with running tests:

- If the test is passing and you haven't changed it - no need to rerun it,

**Run specific app:**

```bash
docker exec dashboard-2-app-1 sh -c "DJANGO_SETTINGS_MODULE=dashboard.settings.test pytest <app_name> --verbose -x"
```

**Run specific test file:**

```bash
docker exec dashboard-2-app-1 sh -c "DJANGO_SETTINGS_MODULE=dashboard.settings.test pytest <app_name>/tests/test_file.py --verbose -x"
```

**Run specific test function:**

```bash
docker exec dashboard-2-app-1 sh -c "DJANGO_SETTINGS_MODULE=dashboard.settings.test pytest <app_name>/tests/test_file.py::test_function_name --verbose -x"
```

add TEST_ENV=release to the start of testing command like so: docker exec dashboard-2-app-1 sh -c "TEST_ENV=release DJANGO_SETTINGS_MODULE=dashboard.settings.test ..." only if need to run specific skipped tests. Otherwise run without it.

**Flags:**

- `--verbose` - detailed output
- `-x` - stop on first failure
- `--reuse-db` - reuse database (already in pytest.ini)

## Coverage Checklist

For each test suite, ensure you test:

- Happy path (valid inputs, expected behavior)
- Error cases (invalid inputs, constraints)
- Edge cases (null values, empty lists, boundaries)
- Permissions (different user roles)
- Side effects (database state, async tasks)
