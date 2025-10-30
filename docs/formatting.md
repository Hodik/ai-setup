# Code Formatting Guide

## Python Version

**Python 3.12** - Use modern Python 3.12+ syntax and features

## Code Formatter

### Black 22.12.0

**Enforced**: All Python code must be formatted with **Black 22.12.0**

```bash
black .
```

**Black configuration:**
- Line length: **88 characters**
- String quotes: **Double quotes**
- Trailing commas in multi-line structures
- 4 spaces for indentation (no tabs)

Black automatically handles all formatting - no manual formatting needed.

## Import Organization

### Import Order (PEP 8)

Organize imports in **4 groups** separated by blank lines:

1. **Standard library** imports
2. **Django** core imports  
3. **Third-party** library imports
4. **Local application** imports
