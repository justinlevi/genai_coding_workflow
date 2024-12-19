### Development: Core Information and Guidelines

Project lives at:
[ FILL IN ABSOLUTE PATH]


## RULES

- use XXX
- don't use XXX

## Test-Driven Development Approach
Each phase must have passing tests before proceeding to the next phase. The testing strategy follows:

1. Write tests first
2. Implement minimum code to pass tests
3. Refactor while maintaining test coverage
4. Document test cases and coverage
5. Get PR review and approval before moving to next phase


## Guidelines and Best Practices

1. **Service Lifecycle Management**
   - Single source of truth for service instantiation
   - Proper startup/shutdown sequences
   - Resource cleanup on application shutdown
   - Controlled service initialization order

2. **Dependency Injection Architecture**
   - Loose coupling between components
   - Inversion of Control (IoC) principle
   - Easy testing through dependency substitution
   - Clear dependency graph
   - Avoid circular dependencies

3. **Configuration Management**
   - Type-safe settings using Pydantic
   - Environment-based configuration
   - Validation at startup
   - Single source of truth for settings

4. **Service Pattern Implementation**
   - Singleton services where appropriate
   - Factory pattern for service creation
   - Interface-based design for swappable implementations
   - Clear service boundaries and responsibilities

5. **Resource Management**
   - Database connection pooling
   - Proper connection lifecycle
   - Resource cleanup
   - Connection state management

6. **Logging Infrastructure**
   - Centralized logging configuration
   - Consistent log formatting
   - Request ID tracking
   - Structured logging

7. **Testing Support**
   - Easy mocking of dependencies
   - Isolated test environments
   - Test fixtures
   - Clear test data management

8. **Error Handling**
   - Consistent error responses
   - Proper error propagation
   - Error logging
   - Request tracing

- Always create modular and reusable code
- Always favor simplicity over complexity. The code should be easy to reason about and maintain.
- Composability over inheritance
- Document code to make it easier to understand
- Use meaningful variable and function names
- Use appropriate design patterns to make the code more readable and maintainable
- Use dependency injection to make the code more testable and maintainable
- Some design patterns which we should keep in mind are:
    - Dependency Injection
    - Inversion of Control
    - SOLID principles
    - KISS principle
    - YAGNI principle
    - DRY principle
    - Single Responsibility Principle
    - Open/Closed Principle
    - Liskov Substitution Principle
    - Interface Segregation Principle


## Core Testing Principles
Testing FastAPI applications requires a systematic approach focused on ensuring reliability and maintainability. Here are the fundamental principles for writing effective tests:

### Test Structure

#### Test Isolation
- Each test should be completely independent and not rely on the state of other tests
- This isolation makes tests easier to debug and maintain
- Use fixtures and setup/teardown to ensure clean test environments

#### Test Organization 
- Tests should be organized in separate files from the main application code
- Follow naming convention `test_*.py` for test files
- Group related tests into test classes or modules
- Use descriptive test file names that indicate what is being tested

### Essential Test Types

#### Unit Tests
Write focused tests for individual components and functions that:
- Verify specific functionality in isolation
- Test one concept per test function
- Use clear, descriptive test names
- Mock external dependencies
- Focus on testing business logic

#### Integration Tests
Create tests that validate:
- Interactions between components
- External service integrations 
- Complete request-response cycles
- Database operations
- API endpoint behaviors

The universally accepted structure for tests follows the "Arrange-Act-Assert" (AAA) pattern, which creates clear and maintainable test cases.

ALWAYS - make the minimum changes possible when refactoring, then re-test.

## Success Criteria

1. Functional Requirements:
   - [ ] Google OAuth login works end-to-end with secure cookie storage
   - [ ] Token refresh flow functions correctly with preemptive refresh
   - [ ] Protected routes are properly secured with guard system
   - [ ] Streaming endpoints maintain authentication

2. Non-Functional Requirements:
   - [ ] All tests pass with 95% coverage
   - [ ] Response times under 100ms for token validation
   - [ ] Proper error handling with recovery
   - [ ] Comprehensive logging of auth events