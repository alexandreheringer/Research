---
trigger: always_on
---

# Python Development Agent Skills

This guide provides directive rules and best practices for AI agents developing Python projects. Each section includes clear priority levels for decision-making.

---

## Priority Levels

- 🔴 **CRITICAL / MUST**: Mandatory rules - never violate
- 🟡 **SHOULD**: Strongly recommended - follow unless specific reason not to
- 🟢 **MAY / OPTIONAL**: Contextual recommendations - use judgment
- ⛔ **NEVER / AVOID**: Anti-patterns - do not use

---

## Table of Contents

1. [Core Principles & Rules](#1-core-principles--rules)
2. [Project Setup & Structure](#2-project-setup--structure)
3. [Code Writing Standards](#3-code-writing-standards)
4. [Testing Requirements](#4-testing-requirements)
5. [Documentation Standards](#5-documentation-standards)
6. [Security Practices](#6-security-practices)
7. [Performance Guidelines](#7-performance-guidelines)
8. [Error Handling & Logging](#8-error-handling--logging)
9. [Dependency & Environment Management](#9-dependency--environment-management)
10. [Deployment & CI/CD](#10-deployment--cicd)
11. [Common Pitfalls to Avoid](#11-common-pitfalls-to-avoid)
12. [Tooling & Configuration](#12-tooling--configuration)

---

## Library-Specific Rules

This guide covers general Python development. **Load additional rule files** based on libraries used:

| Library/Tool                                         | Load Additional File     | When to Load                              |
| ---------------------------------------------------- | ------------------------ | ----------------------------------------- |
| **Pandas, NumPy, Matplotlib, Seaborn, Scikit-learn** | `python-data-science.md` | Data analysis, visualization, or ML tasks |
| **Jupyter Notebook**                                 | `python-jupyter.md`      | Working in Jupyter notebooks              |

**Loading Priority:**
1. **Always load this file first** (`python-agent.md`) - contains base Python rules
2. **Then load library-specific files** as needed - contains specialized rules
3. **In case of conflict**, rules in this file take precedence

---

## 1. Core Principles & Rules

### 🔴 CRITICAL - Always Follow

1. **Never commit secrets or credentials**
   - Use environment variables for all sensitive data
   - Never hardcode API keys, passwords, or tokens

2. **Always use type hints**
   - Every function must have parameter and return type annotations
   - Use `typing` module for complex types

3. **Always use virtual environments**
   - Isolate project dependencies
   - Never install packages globally

4. **Follow PEP 8**
   - Code style consistency is mandatory
   - Use automated formatters

5. **Test coverage must be ≥ 80%**
   - Write tests for all new code
   - Maintain existing coverage

### 🟡 SHOULD - Strong Recommendations

1. **Use pytest as the testing framework**
   - Avoid unittest module (except for unittest.mock)
   - Use pytest fixtures and parametrization

2. **Implement structured logging**
   - Use logging module with proper configuration
   - Consider structlog for complex applications

3. **Use dependency injection**
   - Improves testability
   - Reduces coupling

4. **Follow Test-Driven Development (TDD)**
   - Write tests before implementation when possible
   - Clarifies requirements

### 🟢 MAY - Contextual Recommendations

1. **Consider design patterns**
   - Factory, Strategy, Observer, etc.
   - Use when they simplify code

2. **Use async/await for I/O-bound operations**
   - Improves performance for concurrent operations
   - Not needed for CPU-bound tasks

3. **Implement caching**
   - Use `functools.lru_cache` for expensive computations
   - Consider Redis/Memcached for distributed systems

---

## 2. Project Setup & Structure

### 🔴 CRITICAL - Required Structure

When creating a new Python project, MUST use this structure:

```
project_name/
├── src/
│   └── package_name/
│       ├── __init__.py          # MUST exist
│       ├── models/
│       │   └── __init__.py      # MUST exist
│       ├── services/
│       │   └── __init__.py      # MUST exist
│       ├── controllers/
│       │   └── __init__.py      # MUST exist
│       ├── utils/
│       │   └── __init__.py      # MUST exist
│       └── main.py
├── tests/
│   ├── __init__.py              # MUST exist
│   ├── test_models/
│   ├── test_services/
│   └── test_controllers/
├── docs/
├── config/
├── .env.example                 # MUST provide
├── .gitignore                   # MUST include .env
├── pyproject.toml
├── README.md
└── requirements.txt
```

**Critical Rules:**
- ✅ All Python packages MUST have `__init__.py`
- ✅ Source code MUST be in `src/` directory
- ✅ Tests MUST be in separate `tests/` directory
- ✅ Tests MUST mirror source structure
- ✅ MUST provide `.env.example` (never commit `.env`)

### 🟡 SHOULD - File Naming

- Modules: `lowercase_with_underscores.py`
- Packages: `lowercase` (avoid underscores)
- Tests: `test_module_name.py`
- Classes: `PascalCase`
- Functions/Variables: `snake_case`
- Constants: `UPPER_CASE`

### 🟢 MAY - Architecture Patterns

Choose based on project complexity:
- Simple projects: Basic modular structure
- Medium projects: Layered architecture
- Large projects: Hexagonal/Clean architecture
- Distributed systems: Microservices

---

## 3. Code Writing Standards

### 🔴 CRITICAL - Type Hints

**MUST add type hints to ALL code:**

```python
from typing import List, Optional, Dict, Union

def process_users(
    user_ids: List[int],
    limit: Optional[int] = None,
    filters: Optional[Dict[str, str]] = None
) -> Dict[int, str]:
    """Process user IDs and return mapping of ID to name."""
    pass
```

**For TYPE_CHECKING in tests:**

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from _pytest.capture import CaptureFixture
    from _pytest.fixtures import FixtureRequest
    from _pytest.logging import LogCaptureFixture
    from _pytest.monkeypatch import MonkeyPatch
    from pytest_mock.plugin import MockerFixture
```

### 🔴 CRITICAL - Docstrings

**MUST add docstrings to ALL functions and classes:**

```python
def calculate_total(items: List[float], tax_rate: float = 0.1) -> float:
    """
    Calculate the total price including tax.

    Args:
        items: List of item prices
        tax_rate: Tax rate to apply (default: 0.1 for 10%)

    Returns:
        Total price including tax

    Raises:
        ValueError: If tax_rate is negative
    """
    if tax_rate < 0:
        raise ValueError("Tax rate cannot be negative")
    return sum(items) * (1 + tax_rate)
```

**Follow PEP 257 conventions:**
- One-line docstrings for simple functions
- Multi-line docstrings with Args, Returns, Raises for complex functions
- Class docstrings explain purpose and usage

### 🔴 CRITICAL - Code Style

**MUST use Ruff exclusively:**
- **Ruff**: Fast, comprehensive linter and formatter
- No alternatives permitted (black, flake8, pylint, etc.)
- **mypy** or **pyright** for static type checking

**MUST follow PEP 8:**
- Line length: 88-100 characters
- 4 spaces for indentation (never tabs)
- 2 blank lines between top-level definitions
- 1 blank line between method definitions

### 🟡 SHOULD - Import Order

```python
# 1. Standard library
import os
import sys
from pathlib import Path

# 2. Third-party packages
import requests
from flask import Flask

# 3. Local imports
from .models import User
from .services import UserService
```

### 🟡 SHOULD - Use Named Constants

```python
# Good
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"

def fetch_data():
    for attempt in range(MAX_RETRIES):
        response = requests.get(API_BASE_URL, timeout=DEFAULT_TIMEOUT)
```

### ⛔ NEVER

- ❌ Use mutable default arguments
- ❌ Use wildcard imports (`from module import *`)
- ❌ Use global variables for mutable state
- ❌ Use bare `except:` clauses
- ❌ Ignore type checker warnings

---

## 4. Testing Requirements

### 🔴 CRITICAL - Testing Framework

**MUST use TestSprite for all testing:**

- **DO NOT** manually create test files or configure test frameworks
- **DO NOT** write test code directly
- **DELEGATE** all testing tasks to the specialized testing agent

**Testing Agent Integration:**
- **Agent Name**: `XXXX` (specialized testing agent)
- **Skills**: `yyyyyy` (TestSprite integration skills)

**When testing is required:**
1. Invoke the `XXXX` testing agent
2. The agent will use TestSprite MCP server to:
   - Generate appropriate tests
   - Execute tests
   - Analyze results
   - Generate reports
3. Ensure test coverage ≥ 80%

**TestSprite handles:**
- Test framework selection
- Test structure and organization
- Test execution
- Coverage reporting

### 🔴 CRITICAL - Test Requirements

**Every test MUST have:**
1. Type annotations
2. Docstring explaining what is tested
3. Descriptive name starting with `test_`

```python
def test_divide_by_zero_raises_value_error() -> None:
    """Test that dividing by zero raises ValueError with correct message."""
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)
```

### 🔴 CRITICAL - Test Coverage

**MUST maintain ≥ 80% test coverage:**
- Use `pytest --cov=src --cov-report=term-missing --cov-fail-under=80`
- Cover edge cases and error conditions
- Test boundary conditions

### 🟡 SHOULD - Test Organization

```
tests/
├── __init__.py
├── conftest.py          # Shared fixtures
├── test_models/
│   ├── __init__.py
│   └── test_user.py
├── test_services/
│   ├── __init__.py
│   └── test_user_service.py
└── test_integration/
    └── test_api.py
```

### 🟡 SHOULD - Use Fixtures

```python
# conftest.py
import pytest

@pytest.fixture
def sample_user() -> dict:
    """Provide a sample user for testing."""
    return {"id": 1, "name": "John", "email": "john@example.com"}

# test_user.py
def test_user_processing(sample_user: dict) -> None:
    """Test user data processing."""
    result = process_user(sample_user)
    assert result is not None
```

### 🟡 SHOULD - Parametrize Tests

```python
@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
    (3, 6),
])
def test_double(input: int, expected: int) -> None:
    """Test that double function returns correct values."""
    assert double(input) == expected
```

### 🟡 SHOULD - Mock External Dependencies

```python
def test_api_call(mocker: MockerFixture) -> None:
    """Test that API is called with correct parameters."""
    mock_get = mocker.patch("requests.get")
    mock_get.return_value.json.return_value = {"status": "ok"}

    result = fetch_data()
    assert result["status"] == "ok"
    mock_get.assert_called_once()
```

### ⛔ NEVER

- ❌ Skip tests without good reason
- ❌ Write tests that depend on external services (mock them)
- ❌ Write tests that depend on test execution order
- ❌ Use time.sleep() in tests (use mocking)

---

## 5. Documentation Standards

### 🔴 CRITICAL - Docstring Requirements

**MUST add docstrings to:**
- All public functions
- All public classes
- All public methods
- All modules (at the top of file)

```python
"""
User management module.

This module provides functionality for creating, updating,
and deleting user accounts.
"""

from typing import Optional

class UserService:
    """
    Service for managing user operations.

    Handles user CRUD operations and business logic.
    """

    def create_user(self, email: str, name: str) -> int:
        """
        Create a new user account.

        Args:
            email: User's email address (must be unique)
            name: User's full name

        Returns:
            The ID of the newly created user

        Raises:
            ValueError: If email is invalid or already exists
        """
        pass
```

### 🔴 CRITICAL - README Requirements

**Every project MUST have a README.md with:**
1. Project description
2. Installation instructions
3. Usage examples
4. Configuration guide
5. Testing instructions

### 🟡 SHOULD - Update Existing Docstrings

When modifying code:
- Update docstrings to reflect changes
- Keep comments in sync with code
- Remove obsolete comments

### 🟡 SHOULD - Add Comments for Complex Logic

```python
def calculate_shipping(weight: float, distance: float) -> float:
    """Cal