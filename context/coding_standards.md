# Coding Standards & Rules

> [!IMPORTANT]
> This document defines the coding standards that **all agents MUST follow** when generating or modifying code. The Fix Agent and Reasoning Agent should reference these standards before implementing any changes.

---

## General Principles (Language Agnostic)

### 1. Code Quality Standards

- **Readability First**: Code should be written for humans to read, with machines to execute second
- **DRY Principle**: Don't Repeat Yourself - extract common logic into reusable functions
- **KISS Principle**: Keep It Simple, Stupid - avoid over-engineering
- **YAGNI Principle**: You Aren't Gonna Need It - don't add features "just in case"
- **Single Responsibility**: Each function/class should do one thing well

### 2. Naming Conventions

| Type            | Convention                                                  | Examples                                |
| --------------- | ----------------------------------------------------------- | --------------------------------------- |
| Variables       | `snake_case` (Python), `camelCase` (JS/TS)                  | `user_count`, `getUserData`             |
| Constants       | `UPPER_SNAKE_CASE`                                          | `MAX_RETRY_COUNT`, `API_BASE_URL`       |
| Functions       | `snake_case` (Python), `camelCase` (JS/TS)                  | `calculate_total()`, `calculateTotal()` |
| Classes         | `PascalCase`                                                | `UserController`, `DatabaseManager`     |
| Files           | `snake_case` (Python), `kebab-case` or `PascalCase` (JS/TS) | `user_service.py`, `user-service.ts`    |
| Private members | `_leading_underscore`                                       | `_internal_method`, `privateField`      |

### 3. Code Formatting

- **Indentation**: 2 spaces (JS/TS/HTML/CSS) or 4 spaces (Python)
- **Line Length**: Maximum 80-100 characters per line
- **Trailing Whitespace**: Never allowed
- **Semicolons**: Required in JavaScript/TypeScript
- **Quotes**: Prefer single quotes `'` for strings, use double `"` only when string contains single quotes

### 4. Documentation Standards

- **Function Documentation**: Every function must have a docstring/comment describing:
  - Purpose (what it does)
  - Parameters (with types)
  - Return value (with type)
  - Exceptions/Errors it may throw

```javascript
/**
 * Fetches user data from the API by user ID.
 * @param {string} userId - The unique identifier of the user
 * @returns {Promise<User>} A promise resolving to the user object
 * @throws {ApiError} When the user is not found or API is unreachable
 */
async function fetchUser(userId) { ... }
```

```python
def fetch_user(user_id: str) -> User:
    """
    Fetches user data from the API by user ID.

    Args:
        user_id: The unique identifier of the user

    Returns:
        The user object containing user data

    Raises:
        ApiError: When the user is not found or API is unreachable
    """
    ...
```

### 5. Error Handling Standards

- **Never ignore errors**: All exceptions must be explicitly handled or propagated
- **Specific exceptions**: Catch specific exception types, not generic `Exception`
- **Fail fast**: Validate inputs early and fail with clear error messages
- **Logging**: Log errors with context (what happened, why, and how to fix)

```javascript
// BAD
try {
  await riskyOperation();
} catch (e) {
  // ignore
}

// GOOD
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof NetworkError) {
    logger.error(`Network failure: ${error.message}`, { url, retries });
    throw new ApiError("Service unavailable", { cause: error });
  }
  throw error;
}
```

### 6. Security Standards

- **Input Validation**: Always validate and sanitize user inputs
- **No Hardcoded Secrets**: Never commit API keys, passwords, or tokens
- **SQL Injection**: Always use parameterized queries
- **XSS Prevention**: Escape user-generated content before rendering
- **Authentication**: Verify permissions before accessing protected resources

---

## Language-Specific Standards

### JavaScript / TypeScript

#### Style Guide: Google JavaScript Style Guide

1. **Use `const` by default, `let` only if reassignment is needed**

   ```javascript
   // GOOD
   const API_URL = "https://api.example.com";
   let counter = 0;

   // BAD
   var data = []; // Never use var
   ```

2. **Use template literals instead of string concatenation**

   ```javascript
   // GOOD
   const message = `Hello ${name}, you have ${count} messages`;

   // BAD
   const message = "Hello " + name + ", you have " + count + " messages";
   ```

3. **Use arrow functions for callbacks**

   ```javascript
   // GOOD
   users.map((user) => user.id);
   users.filter((user) => user.isActive);

   // BAD
   users.map(function (user) {
     return user.id;
   });
   ```

4. **Use async/await over raw Promises**

   ```javascript
   // GOOD
   async function fetchData() {
     try {
       const response = await fetch(url);
       return await response.json();
     } catch (error) {
       handleError(error);
     }
   }

   // AVOID
   function fetchData() {
     return fetch(url)
       .then((response) => response.json())
       .catch(handleError);
   }
   ```

5. **TypeScript Specific:**
   - **Always specify explicit return types on exported functions**
   - **Use `interface` for object shapes, `type` for unions**
   - **Prefer `unknown` over `any`** for untyped data
   - **Use strict null checks**: Enable `strictNullChecks` in tsconfig.json

   ```typescript
   // GOOD
   interface User {
     id: string;
     name: string;
     email: string;
   }

   async function getUser(id: string): Promise<User | null> {
     const response = await fetch(`/api/users/${id}`);
     if (!response.ok) return null;
     return response.json() as Promise<User>;
   }

   // BAD
   async function getUser(id: any): any { ... }
   ```

6. **Import Order:** (as per Google Style Guide)

   ```javascript
   // 1. Node.js built-in modules
   import fs from 'fs';
   import path from 'path';

   // 2. External packages (npm)
   import express from 'express';
   import lodash from 'lodash';

   // 3. Internal imports (from your project)
   import { UserService } from './services/user.service';
   import { APIConfig } from '../config/api.config';

   // 4. Type-only imports (TypeScript)
   import type { User } from './types';
   ```

### Python

#### Style Guide: Google Python Style Guide (PEP 8 + Google enhancements)

1. **Docstrings**: Use Google-style docstrings

   ```python
   def fetch_user(user_id: str) -> Optional[User]:
       """Fetches a user from the database by ID.

       Args:
           user_id: The unique identifier of the user.

       Returns:
           The User object if found, None otherwise.

       Raises:
           ValueError: If user_id is empty or invalid.
       """
       ...
   ```

2. **Type Hints**: Always use type hints for function signatures

   ```python
   # GOOD
   from typing import List, Optional

   def process_items(items: List[str], limit: Optional[int] = None) -> int:
       ...

   # BAD
   def process_items(items, limit=None):
       ...
   ```

3. **Imports**: Group and sort imports (isort format)

   ```python
   # 1. Standard library imports
   import os
   import sys
   from typing import Dict, List

   # 2. Third-party imports
   import requests
   from fastapi import FastAPI

   # 3. Local imports
   from app.models import User
   from app.services import UserService
   ```

4. **String Formatting**: Use f-strings

   ```python
   # GOOD
   message = f"Hello {name}, you have {count} messages"

   # BAD
   message = "Hello {}, you have {} messages".format(name, count)
   message = "Hello " + name + ", you have " + str(count) + " messages"
   ```

### Go (if applicable)

#### Style Guide: Go Code Review Comments + Effective Go

1. **Package Names**: Lowercase, single word, no underscores

   ```go
   package user  // GOOD
   package user_service  // BAD
   ```

2. **Error Handling**: Always check and handle errors immediately

   ```go
   // GOOD
   resp, err := http.Get(url)
   if err != nil {
       return fmt.Errorf("failed to fetch: %w", err)
   }
   defer resp.Body.Close()

   // BAD
   resp, err := http.Get(url)
   // process resp without checking err
   ```

3. **Interface Names**: `-er` suffix for single-method interfaces

   ```go
   type Reader interface {
       Read(p []byte) (n int, err error)
   }

   type FileWriter interface {
       Write(p []byte) (n int, err error)
       Close() error
   }
   ```

---

## Testing Standards

### 1. Test Naming

- **Descriptive names**: `test_<function>_<scenario>_<expected_result>`
- Use `should` or `when` for readability

```javascript
// GOOD
describe("UserService", () => {
  it("should return user when valid ID is provided", () => {});
  it("should throw error when user ID is empty", () => {});
  it("should return null when user does not exist", () => {});
});

// BAD
describe("UserService", () => {
  it("works", () => {});
  it("test1", () => {});
});
```

```python
# GOOD
def test_fetch_user_returns_user_when_valid_id_provided():
    pass

def test_fetch_user_raises_error_when_id_is_empty():
    pass

# BAD
def test_user_1():
    pass
```

### 2. Test Structure (AAA Pattern)

- **Arrange**: Set up test data and mocks
- **Act**: Execute the function being tested
- **Assert**: Verify expected outcomes

```javascript
it("should calculate discount correctly", () => {
  // Arrange
  const price = 100;
  const discount = 0.2;

  // Act
  const result = calculateDiscount(price, discount);

  // Assert
  expect(result).toBe(80);
});
```

### 3. Test Coverage Standards

- **Minimum coverage**: 80% for new code
- **Critical paths**: 100% coverage for authentication, payments, security
- **Edge cases**: Always test null/undefined, empty inputs, boundary values

---

## Git Commit Standards

### Commit Message Format (Conventional Commits)

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `docs`: Documentation only changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples

```
feat(auth): add OAuth2 login support

- Implement Google OAuth2 flow
- Add user session management
- Update login UI

Closes #123
```

```
fix(api): handle null response from user endpoint

Previously, the API would crash when receiving a null response.
Now it returns a 404 with appropriate error message.

Fixes #456
```

---

## Automated Linting & Formatting Tools

The Code Standards Validator Agent should use these tools when available in the target project:

### JavaScript / TypeScript

| Tool                    | Purpose                  | Command                                  | Priority        |
| ----------------------- | ------------------------ | ---------------------------------------- | --------------- |
| **ESLint**              | Linting & style checking | `eslint [file]` or `eslint --fix [file]` | Primary         |
| **Prettier**            | Code formatting          | `prettier --write [file]`                | Secondary       |
| **TypeScript Compiler** | Type checking            | `tsc --noEmit`                           | Required for TS |

### Python

| Tool       | Purpose         | Command         | Priority    |
| ---------- | --------------- | --------------- | ----------- |
| **Black**  | Code formatting | `black [file]`  | Primary     |
| **Flake8** | Linting (PEP 8) | `flake8 [file]` | Secondary   |
| **pylint** | Code analysis   | `pylint [file]` | Optional    |
| **mypy**   | Type checking   | `mypy [file]`   | Recommended |

### Go

| Tool       | Purpose         | Command           | Priority    |
| ---------- | --------------- | ----------------- | ----------- |
| **gofmt**  | Code formatting | `gofmt -w [file]` | Required    |
| **go vet** | Static analysis | `go vet [file]`   | Required    |
| **golint** | Style checking  | `golint [file]`   | Recommended |

### Rust

| Tool        | Purpose         | Command          | Priority |
| ----------- | --------------- | ---------------- | -------- |
| **rustfmt** | Code formatting | `rustfmt [file]` | Required |
| **clippy**  | Linting         | `cargo clippy`   | Required |

### Agent Usage Instructions

**For the Code Standards Validator Agent:**

1. **Detect available tools** by checking for configuration files:
   - JavaScript/TypeScript: `.eslintrc.*`, `prettier.config.*`, `tsconfig.json`
   - Python: `.black`, `setup.cfg`, `pyproject.toml`, `.flake8`
   - Go: `go.mod` (implies gofmt is available)
   - Rust: `rustfmt.toml`, `.clippy.toml`

2. **Run tools in order of priority**:
   - First run formatters (e.g., `prettier --write`, `black`)
   - Then run linters (e.g., `eslint`, `flake8`)
   - Finally run type checkers (e.g., `tsc`, `mypy`)

3. **Handle missing tools gracefully**:
   - If a tool is not available, fall back to manual validation
   - Document which manual checks were performed

---

## Security Checklist (Must Verify Before PR)

- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All user inputs are validated and sanitized
- [ ] SQL queries use parameterized statements
- [ ] Error messages don't leak sensitive information
- [ ] Authentication is checked on protected endpoints
- [ ] Sensitive data is encrypted at rest
- [ ] Dependencies are up-to-date (no known vulnerabilities)

---

## Performance Standards

1. **Database Queries**
   - Use indexed columns in WHERE clauses
   - Avoid N+1 queries (use eager loading)
   - Set query timeouts

2. **API Calls**
   - Implement request timeouts
   - Use connection pooling
   - Cache frequently accessed data

3. **Memory Management**
   - Close file handles and connections
   - Clean up event listeners
   - Avoid memory leaks in closures

---

## Agent Enforcement Checklist

**Before generating code, the Fix Agent MUST:**

1. Review the existing codebase style and match it
2. Apply the appropriate language-specific standards from this document
3. Ensure all new functions have proper documentation
4. Include type hints/annotations where applicable
5. Add appropriate error handling
6. Follow the security checklist

**The Reasoning Agent MUST include in its FIX_PLAN:**

- Coding standards to follow for this fix
- Any specific patterns to match existing code
- Security considerations
- Test requirements
