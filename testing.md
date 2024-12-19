# Test Suite Documentation

## Overview

This test suite implements a industry best practices and established patterns for maintainability, readability, and reliability.


## Test Organization

### Test Structure

All test suites follow a consistent structure:

1. Nested test classes for logical grouping
2. Comprehensive docstrings with Given-When-Then format
3. Fixtures for shared functionality
4. Clear test naming

Example:
```python
class TestAuthTokens:
    """Test suite for authentication tokens.
    
    Tests cover:
    - Token lifecycle management
    - Security measures
    - Refresh token rotation
    """
    
    @pytest.fixture
    def validate_auth_error(self):
        """Validate authentication error details."""
        def _validate(error, expected_status: int, expected_detail: str):
            assert error.status_code == expected_status
            assert expected_detail in error.detail
        return _validate
    
    class TestTokenRotation:
        """Tests for token rotation functionality."""
        
        async def test_refresh_token_rotation(self, token_service, test_user):
            """Test refresh token rotation on use.
            
            Given:
                - A valid refresh token
                - An authenticated user
            When:
                - The token is used for refresh
            Then:
                - A new token pair is issued
                - Old refresh token is invalidated
            """
            # Test implementation
```

### Best Practices

1. **Composition Over Inheritance**
   - Use fixtures for shared functionality
   - Keep test classes focused and independent
   - Avoid deep inheritance hierarchies

2. **Clear Documentation**
   - Comprehensive docstrings
   - Given-When-Then format
   - Clear test naming

3. **Test Isolation**
   - Each test is independent
   - Proper setup and teardown
   - No shared state between tests

4. **Error Validation**
   - Consistent error checking
   - Detailed error messages
   - Proper status codes

### Service-Specific Test Patterns

1. **Authentication Service Tests**
   - Token lifecycle management
   - OAuth integration
   - Error handling
   - Security validations

2. **Token Service Tests**
   - Token generation and validation
   - Refresh mechanisms
   - Token rotation
   - Cache management

3. **Audit Logging Tests**
   - Event capture
   - Log formatting
   - Security events
   - Log levels

4. **Cookie Security Tests**
   - Secure cookie handling
   - SameSite policies
   - Environment detection
   - Cookie expiration

5. **Middleware Tests**
   - Request processing
   - Token validation
   - Security headers
   - Error handling

## Running Tests

```bash
# Run all tests
cd /path/to/project/fastapi && poetry run pytest tests/ -v

# Run specific test file
poetry run pytest tests/**/*.py -v

# Run specific test
poetry run pytest tests/**/*.py::[SPECIFIC_TEST_NAME] -v

# Run with coverage
poetry run pytest --cov=src tests/ -v
```

## Test Categories

### 1. Unit Tests
- Service layer tests
- Utility function tests
- Model tests
- Schema validation tests

### 2. Integration Tests
- API route tests
- Database interaction tests
- OAuth integration tests
- Event handling tests

### 3. Security Tests
- Authentication flows
- Token security
- Cookie handling
- CORS policies
- Rate limiting

### 4. Audit Tests
- Event logging
- Security alerts
- Audit trail validation
- Log format verification

## Contributing

When contributing new tests:

1. Follow the established patterns
2. Use the provided base classes
3. Add appropriate documentation
4. Ensure test isolation
5. Run the full suite before submitting

### Test Documentation Template

```python
"""Test module description.

This module tests:
- Feature 1
- Feature 2
- Feature 3
"""

class TestFeature:
    """Test suite description.
    
    Tests cover:
    - Aspect 1
    - Aspect 2
    - Aspect 3
    """
    
    async def test_specific_feature(self):
        """Test description.
        
        Given:
            - Precondition 1
            - Precondition 2
        When:
            - Action is performed
        Then:
            - Expected result 1
            - Expected result 2
        """
```

## Maintenance

Regular maintenance tasks:

1. Keep test data up to date
2. Review and update documentation
3. Check for deprecated patterns
4. Monitor test coverage
5. Update dependencies

## Common Fixtures

1. **Database Fixtures**
   - `test_db`: Database session
   - `test_user`: Test user instance
   - `admin_user`: Admin user instance

2. **Authentication Fixtures**
   - `token_service`: Token service instance
   - `auth_service`: Auth service instance
   - `mock_google_auth`: Google OAuth mock

3. **Request Fixtures**
   - `client`: FastAPI test client
   - `mock_request`: Mock request object
   - `validate_auth_error`: Error validation helper

4. **Security Fixtures**
   - `cookie_service`: Cookie handling
   - `audit_logger`: Audit logging
   - `token_cache`: Token caching

## Test Coverage Goals

Maintain test coverage for:
1. All authentication flows
2. Token lifecycle events
3. Security mechanisms
4. Error scenarios
5. Audit logging
6. Cookie handling
7. OAuth integrations
