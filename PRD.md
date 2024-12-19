# EXAMPLE: Database Layer Implementation - Product Requirements Document (PRD)  

## Overview
This document outlines the implementation of a robust, testable database layer for our FastAPI application. The implementation follows test-driven development principles and focuses on creating a maintainable, efficient, and type-safe database interface.

## Goals
- Create a reliable and efficient database connection management system
- Ensure proper resource lifecycle management
- Implement comprehensive error handling and validation
- Provide clear testing patterns and infrastructure
- Enable easy monitoring and debugging

## Core Principles
- Test-driven development
- Type safety
- Resource efficiency
- Error resilience
- Maintainable code structure

## Implementation Phases

### Phase 1: Database Configuration Management
**Why?** Configuration errors are a common source of production issues  
**Why?** Type-safe configuration prevents runtime errors  
**Why?** Validation at startup catches issues early  
**Why?** Centralized configuration improves maintainability  
**Why?** Strong typing enables better IDE support and catches errors during development

#### Requirements
1. Type-safe database settings using Pydantic  
2. URL validation with regex patterns  
3. Pool configuration validation  
4. Environment-based configuration  
5. Configuration error handling

#### Detailed Steps
1. **Create Pydantic Model**  
2. **URL Validation**  
3. **Pool Configuration Validation**  
4. **Environment-Based Configuration**  
5. **Configuration Error Handling**  
6. **Potential Gotchas**  
7. **Best Practices**

#### Example Configuration Model
```python
class DatabaseSettings(BaseModel):
    DB_URL: str = Field(
        description="Database connection URL",
        pattern=r"^postgresql\+asyncpg://.*$"
    )
    POOL_SIZE: int = Field(
        default=5,
        ge=1,
        le=20,
        description="Database connection pool size"
    )
    # Additional settings...
```

#### Test Cases
1. Valid configuration validation  
2. Invalid URL detection  
3. Pool size bounds checking  
4. Configuration error handling  
5. Environment variable loading

====================================================

### Phase 2: Connection Management
**Why?** Connections are expensive resources  
**Why?** Connection pooling improves performance  
**Why?** Proper lifecycle management prevents resource leaks  
**Why?** Connection verification ensures reliability  
**Why?** Error handling enables graceful degradation

#### Requirements
1. Async connection pool setup  
2. Connection verification  
3. Connection lifecycle management  
4. Error handling and recovery  
5. Resource cleanup

#### Detailed Steps
1. **Initialize Async Engine**:  
   - Use `create_async_engine` from SQLAlchemy with the validated URL and pool configuration.  
   - Ensure that `pool_size` and other parameters match the configured values from Phase 1.  
   - Disable unnecessary logging in production (e.g., `echo=False`).

2. **Connection Pool Setup**:  
   - Confirm that the engine’s connection pool is created asynchronously.  
   - Validate that the pool size, timeout, and overflow settings align with the environment-based configurations.

3. **Connection Verification**:  
   - Implement a verification step (e.g., `verify_connection(engine)`) that issues a simple `SELECT 1` query.  
   - If verification fails, raise a `DatabaseConfigError` and fail fast on startup.

4. **Lifecycle Management**:  
   - Ensure the engine and any associated resources are properly cleaned up during application shutdown.  
   - Call `await engine.dispose()` in the FastAPI shutdown event handler to close all connections.

5. **Error Handling & Recovery**:  
   - On transient connection failures, implement a retry mechanism with exponential backoff.  
   - Integrate a circuit breaker to temporarily halt DB requests if repeated failures occur beyond the defined threshold.

6. **Resource Cleanup**:  
   - Confirm that no open connections remain on application shutdown.  
   - Use finalizers in tests to ensure the engine is disposed of and any async connections are closed.

7. **Potential Gotchas**:  
   - Ensure that async contexts are properly awaited—forgetting `await` can cause resource leaks.  
   - Incorrect pool configuration can lead to performance bottlenecks or too many idle connections.

8. **Best Practices**:
   - **Mocking Async Context Managers**:  
     - When writing unit tests, especially those that verify connection creation or verification, use mocking frameworks like `unittest.mock` or `pytest-asyncio` fixtures.  
     - For async context managers (e.g., `async with engine.connect():`), ensure that you mock both the `connect()` method and the resulting connection object's async methods.  
     - Return `async context managers` from your mock functions. For example, create a simple async context manager mock:
       ```python
       from contextlib import asynccontextmanager
       from unittest.mock import AsyncMock

       @asynccontextmanager
       async def mock_engine_connect():
           mock_conn = AsyncMock()
           yield mock_conn
       ```
       Then patch `engine.connect` to return `mock_engine_connect()` so that `async with engine.connect():` works as intended in tests.
     - Make sure to verify that tests run with `pytest-asyncio` or another async testing framework so that async mocks are properly awaited.

   - **Isolate Tests from the Real Database**:  
     - Use dependency injection to pass a mock engine or connection into your code paths, allowing you to test connection logic without hitting a real database.
     - Validate that verification, retry logic, and circuit breaker behavior can be triggered by manipulating mock return values and exceptions.

   - **Logging and Observability**:  
     - Log each connection attempt and its result. This helps debug connection pool issues in production.  
     - Use structured logging (e.g., `structlog`) to ensure logs are parseable and can be correlated with request traces.

   - **Performance Considerations**:  
     - Avoid overloading the database with too-frequent connection verifications. Perform them once at startup and rely on application-level health checks during runtime.  
     - Monitor connection pool utilization metrics to inform configuration changes.

#### Test Cases
1. Successful connection creation  
2. Connection verification  
3. Connection failure handling (including retries and circuit breaker triggering)  
4. Pool configuration verification (testing min/max pool sizes)  
5. Resource cleanup verification (ensuring engine is disposed and async contexts are closed)

#### Implementation Considerations

1. **Async Context Manager Patterns**
   - All database connections MUST be handled using async context managers
   - Connection verification should use `async with` pattern
   - Ensure proper cleanup in both success and failure cases
   - Example pattern:
     ```python
     async with engine.connect() as conn:
         result = await conn.execute(query)
         value = await result.scalar()
     ```

2. **Pool Configuration Validation**
   - Do NOT rely on direct pool size inspection for validation
   - Pool size is initialized lazily and may not reflect configured values immediately
   - Instead, validate pool behavior through:
     - Connection acquisition limits
     - Maximum concurrent connections
     - Pool overflow behavior

3. **Error Handling Hierarchy**
   - Implement error handling in this specific order:
     1. SQLAlchemy-specific exceptions (most specific)
     2. Database-specific exceptions
     3. General exceptions (least specific)
   - Each layer should wrap lower-level exceptions with contextual information
   - Maintain error context through the stack

4. **Testing Considerations**
   - Mock async context managers properly:
     ```python
     @asynccontextmanager
     async def mock_connect():
         yield mock_connection  # Don't raise here for connection simulation
     ```
   - Simulate failures at the correct layer:
     - Connection failures: Mock the context manager
     - Query failures: Mock the connection's execute method
     - Result failures: Mock the result's scalar/fetchall methods
   - Test both immediate and delayed failures

5. **Resource Lifecycle**
   - Implement explicit startup sequence:
     1. Engine creation
     2. Connection verification
     3. Pool initialization
   - Implement explicit shutdown sequence:
     1. Wait for active queries (with timeout)
     2. Close active connections
     3. Dispose of engine
   - Log all lifecycle events

6. **Connection Verification Strategy**
   - Implement progressive verification:
     1. Basic connectivity (`SELECT 1`)
     2. Permissions check (if applicable)
     3. Pool configuration verification
   - Define retry strategy:
     - Maximum retry attempts
     - Backoff intervals
     - Circuit breaker conditions

7. **Monitoring and Diagnostics**
   - Track and expose:
     - Current pool utilization
     - Failed connection attempts
     - Connection lifecycle durations
     - Query execution times
   - Define health check criteria

#### Common Pitfalls to Avoid

1. **Async Context Management**
   - DON'T use synchronous context managers with async code
   - DON'T mix async and sync database operations
   - DON'T forget to await connection operations

2. **Pool Management**
   - DON'T validate pool configuration through direct attribute access
   - DON'T assume immediate pool initialization
   - DON'T ignore pool overflow conditions

3. **Error Handling**
   - DON'T catch generic exceptions before specific ones
   - DON'T lose original exception context
   - DON'T ignore transient failures without retry logic

4. **Testing**
   - DON'T mock at the wrong layer (e.g., engine vs. connection)
   - DON'T use sync mocks for async operations
   - DON'T forget to test cleanup paths

#### Validation Checklist

Before considering Phase 2 complete:

1. [ ] All database operations use async context managers
2. [ ] Connection pool behavior is validated through functional tests
3. [ ] Error handling covers all specified exception types
4. [ ] Resource cleanup is verified in all scenarios
5. [ ] Monitoring hooks are in place
6. [ ] All tests follow async testing best practices

====================================================

### Phase 3: Session Management
**Why?** Sessions manage transaction boundaries  
**Why?** Proper session handling prevents data corruption  
**Why?** Session context managers ensure cleanup  
**Why?** Session factories enable dependency injection  
**Why?** Session lifecycle management prevents resource leaks

#### Requirements
1. Async session factory  
2. Session context managers  
3. Transaction management  
4. Session cleanup  
5. Error handling
6. Session timeout handling
7. Transaction isolation configuration
8. Session pooling strategy
9. Cleanup guarantees

#### Implementation Considerations

1. **Session Factory Configuration**
   - Configuration must be done through factory creation, not post-creation
   - Required settings:
     ```python
     async_sessionmaker(
         bind=engine,
         class_=AsyncSession,
         expire_on_commit=False,
         autoflush=False,
         # New settings:
         pool_size=5,
         pool_timeout=30,
         pool_recycle=3600
     )
     ```

2. **Error Handling Hierarchy**
   ```
   SQLAlchemyError
   ├── DatabaseConnectionError (session creation only)
   ├── TransactionError (commit/rollback)
   └── OperationalError (queries/operations)
   ```
   - Only wrap connection errors during session creation
   - Preserve original error context for debugging
   - Ensure cleanup runs in correct order

3. **Session Lifecycle States**
   ```
   Created → Initialized → Active → Cleaning → Closed
                ↓           ↓
             Failed     Failed (with cleanup)
   ```
   - Define valid state transitions
   - Specify cleanup requirements for each state
   - Document retry policies per state

4. **Transaction Management**
   - Maximum nesting depth: 3 levels
   - Parent transaction state preservation rules:
     ```python
     async with session.begin():      # Level 1
         async with session.begin_nested():  # Level 2
             # Failure here doesn't affect Level 1
         # Level 1 still valid
     ```
   - Explicit savepoint management
   - Rollback behavior documentation

5. **Resource Cleanup Guarantees**
   - Cleanup must run exactly once
   - Order of operations:
     1. Rollback any active transaction
     2. Close any active cursors
     3. Return session to pool
     4. Release connection
   - Timeout handling during cleanup

#### Validation Requirements

1. **Session Creation**
   - Verify factory configuration
   - Test connection establishment
   - Validate session settings
   - Check pool integration

2. **Transaction Handling**
   - Test nested transaction isolation
   - Verify savepoint creation/rollback
   - Confirm cleanup order
   - Validate state transitions

3. **Error Scenarios**
   - Test partial initialization
   - Verify cleanup in all error paths
   - Validate error wrapping rules
   - Check resource cleanup ordering

4. **Performance Criteria**
   - Session creation: < 50ms
   - Transaction begin: < 10ms
   - Cleanup completion: < 100ms
   - Pool reuse rate: > 90%

#### Testing Considerations

1. **Mock Requirements**
   ```python
   @pytest.fixture
   def mock_session():
       session = AsyncMock(spec=AsyncSession)
       session.begin = asynccontextmanager(
           async def _begin():
               yield AsyncMock()
       )
       return session
   ```

2. **State Verification**
   - Track cleanup calls
   - Verify transaction boundaries
   - Monitor resource usage
   - Log state transitions

3. **Error Injection**
   - Session creation failures
   - Transaction boundary errors
   - Cleanup failures
   - Pool exhaustion

#### Common Pitfalls to Avoid

1. **Session Management**
   - DON'T reuse sessions across requests
   - DON'T mix sync and async session operations
   - DON'T leave transactions open
   - DON'T catch exceptions without proper cleanup

2. **Transaction Boundaries**
   - DON'T rely on implicit transactions
   - DON'T nest transactions without proper error handling
   - DON'T commit/rollback outside transaction context
   - DON'T forget to handle nested transaction failures

3. **Resource Management**
   - DON'T create sessions without context managers
   - DON'T leave sessions open after exceptions
   - DON'T ignore cleanup in error paths
   - DON'T forget to dispose of engine references

4. **Testing**
   - DON'T mock at wrong abstraction level
   - DON'T ignore transaction boundaries in tests
   - DON'T forget to test rollback scenarios
   - DON'T mix real and mock sessions in tests

#### Example Session Factory
```python
def create_session_factory(engine: AsyncEngine) -> async_sessionmaker:
    return async_sessionmaker(
        engine,
        class_=AsyncSession,
        expire_on_commit=False,
        autoflush=False
    )
```

#### Test Cases
1. Session creation and cleanup
2. Transaction management:
   - Successful commits
   - Rollback scenarios
   - Nested transactions
3. Error handling:
   - Transaction failures
   - Session cleanup after errors
   - Connection recovery
4. Resource management:
   - Session disposal
   - Connection cleanup
5. Context manager behavior:
   - Proper cleanup in success case
   - Proper cleanup in error case
   - Nested context handling

#### Validation Checklist

Before considering Phase 3 complete:

1. [ ] All sessions use async context managers
2. [ ] Transaction boundaries are explicit and tested
3. [ ] Session factory is properly configured
4. [ ] Error handling includes proper cleanup
5. [ ] Lifecycle events are logged
6. [ ] Resource cleanup is verified
7. [ ] All test scenarios are covered:
   - [ ] Successful transactions
   - [ ] Failed transactions
   - [ ] Nested transactions
   - [ ] Session cleanup
   - [ ] Resource disposal

#### Monitoring Requirements

1. **Session Metrics**
   - Active sessions count
   - Transaction duration
   - Rollback frequency
   - Session lifecycle events

2. **Transaction Metrics**
   - Transaction success rate
   - Average transaction duration
   - Nested transaction usage
   - Rollback reasons

3. **Resource Usage**
   - Session creation rate
   - Session cleanup timing
   - Transaction concurrency
   - Resource leak detection

#### Health Checks

1. **Session Health**
   - Verify session creation
   - Check transaction capabilities
   - Monitor session cleanup
   - Track session leaks

2. **Transaction Health**
   - Verify transaction isolation
   - Check savepoint support
   - Monitor transaction timeouts
   - Track long-running transactions

====================================================

### Phase 4: Error Handling and Recovery
**Why?** Errors are inevitable in distributed systems  
**Why?** Proper error handling improves reliability  
**Why?** Error categorization enables appropriate responses  
**Why?** Error recovery prevents cascading failures  
**Why?** Error logging aids debugging

#### Requirements
1. Comprehensive error categorization and handling
2. Recovery strategies with retry mechanisms
3. Structured error logging with correlation IDs
4. Circuit breaker implementation
5. Error monitoring and alerting
6. Resource cleanup verification
7. Integration test coverage for error scenarios

#### Implementation Considerations

1. **Error Categorization Hierarchy**
   ```python
   class DatabaseError(Exception):
       """Base class for all database-related errors."""
       def __init__(
           self, 
           message: str, 
           original_error: Optional[Exception] = None,
           correlation_id: Optional[str] = None,
           retry_allowed: bool = True
       ):
           super().__init__(message)
           self.original_error = original_error
           self.correlation_id = correlation_id
           self.retry_allowed = retry_allowed
           self.timestamp = datetime.utcnow()
   
   class ConnectionError(DatabaseError):
       """Connection establishment and pool errors."""
       retry_allowed = True
   
   class TransactionError(DatabaseError):
       """Transaction-related errors."""
       retry_allowed = True
   
   class DataError(DatabaseError):
       """Data validation and constraint errors."""
       retry_allowed = False
   ```

2. **Recovery Strategy Implementation**
   ```python
   class RetryStrategy:
       max_attempts: int = 3
       base_delay: float = 0.1
       max_delay: float = 1.0
       
       async def execute_with_retry(
           self,
           operation: Callable,
           *args,
           **kwargs
       ) -> Any:
           attempt = 0
           last_error = None
           
           while attempt < self.max_attempts:
               try:
                   return await operation(*args, **kwargs)
               except DatabaseError as e:
                   if not e.retry_allowed:
                       raise
                   last_error = e
                   attempt += 1
                   if attempt < self.max_attempts:
                       delay = min(
                           self.base_delay * (2 ** attempt),
                           self.max_delay
                       )
                       await asyncio.sleep(delay)
           
           raise MaxRetriesExceededError(
               f"Operation failed after {attempt} attempts",
               original_error=last_error
           )
   ```

3. **Circuit Breaker Pattern**
   ```python
   class CircuitBreaker:
       def __init__(
           self,
           failure_threshold: int = 5,
           reset_timeout: float = 60.0
       ):
           self.failure_count = 0
           self.failure_threshold = failure_threshold
           self.reset_timeout = reset_timeout
           self.last_failure_time = None
           self.state = CircuitState.CLOSED
   
       async def execute(
           self,
           operation: Callable,
           *args,
           **kwargs
       ) -> Any:
           if self.state == CircuitState.OPEN:
               if self._should_reset():
                   self.state = CircuitState.HALF_OPEN
               else:
                   raise CircuitOpenError()
           
           try:
               result = await operation(*args, **kwargs)
               if self.state == CircuitState.HALF_OPEN:
                   self.state = CircuitState.CLOSED
                   self.failure_count = 0
               return result
           except DatabaseError as e:
               self._handle_failure()
               raise
   ```

4. **Structured Logging**
   ```python
   class DatabaseLogger:
       def __init__(self, logger: Logger):
           self.logger = logger
       
       def log_error(
           self,
           error: DatabaseError,
           context: Dict[str, Any]
       ) -> None:
           self.logger.error(
               "Database operation failed",
               error_type=type(error).__name__,
               correlation_id=error.correlation_id,
               timestamp=error.timestamp,
               retry_allowed=error.retry_allowed,
               original_error=str(error.original_error),
               **context
           )
   ```

#### Testing Requirements

1. **Error Scenario Coverage**
   - Test each error category
   - Verify retry behavior
   - Validate circuit breaker transitions
   - Check cleanup after errors
   - Verify logging output

2. **Integration Test Scenarios**
   ```python
   @pytest.mark.asyncio
   async def test_connection_retry():
       """Test connection retry behavior."""
       async with create_test_engine() as engine:
           # Simulate network issues
           with network_failure_simulation():
               with pytest.raises(MaxRetriesExceededError):
                   await engine.connect()
   
   @pytest.mark.asyncio
   async def test_circuit_breaker():
       """Test circuit breaker state transitions."""
       breaker = CircuitBreaker()
       # Test state transitions
       for _ in range(5):
           with pytest.raises(DatabaseError):
               await breaker.execute(failing_operation)
       assert breaker.state == CircuitState.OPEN
   ```

3. **Property-Based Testing**
   ```python
   @given(
       failure_threshold=st.integers(min_value=1, max_value=10),
       reset_timeout=st.floats(min_value=0.1, max_value=60.0)
   )
   async def test_circuit_breaker_properties(
       failure_threshold: int,
       reset_timeout: float
   ):
       """Test circuit breaker with various configurations."""
       breaker = CircuitBreaker(
           failure_threshold=failure_threshold,
           reset_timeout=reset_timeout
       )
       # Property: Circuit should open after threshold failures
       for _ in range(failure_threshold):
           await simulate_failure(breaker)
       assert breaker.state == CircuitState.OPEN
   ```

#### Monitoring Requirements

1. **Error Metrics**
   - Error rate by category
   - Retry attempt distribution
   - Circuit breaker state changes
   - Average recovery time
   - Resource cleanup success rate

2. **Alerting Thresholds**
   - Error rate > 1% of operations
   - Circuit breaker open > 5 minutes
   - Max retries exceeded > 3 times/minute
   - Cleanup failures > 0

#### Validation Checklist

Before considering Phase 4 complete:

1. [ ] Error hierarchy implemented and tested
2. [ ] Retry mechanism verified with integration tests
3. [ ] Circuit breaker behavior validated
4. [ ] Logging system implemented and verified
5. [ ] Monitoring metrics exposed
6. [ ] Alert thresholds configured
7. [ ] Resource cleanup verified in error scenarios
8. [ ] All test scenarios covered:
   - [ ] Connection failures
   - [ ] Transaction failures
   - [ ] Data validation errors
   - [ ] Resource cleanup errors
   - [ ] Circuit breaker transitions
   - [ ] Retry exhaustion

#### Common Pitfalls to Avoid

1. **Error Handling**
   - DON'T catch generic exceptions
   - DON'T lose original error context
   - DON'T retry non-retryable errors
   - DON'T ignore cleanup errors

2. **Recovery Mechanisms**
   - DON'T implement infinite retries
   - DON'T retry without backoff
   - DON'T ignore circuit breaker state
   - DON'T reset circuit breaker without cooldown

3. **Resource Management**
   - DON'T leave resources open after errors
   - DON'T ignore cleanup in error paths
   - DON'T forget to log cleanup failures
   - DON'T mix error handling levels

4. **Testing**
   - DON'T mock what you should integrate
   - DON'T ignore edge cases
   - DON'T skip cleanup verification
   - DON'T assume happy path in tests

====================================================

### Phase 5: Monitoring and Metrics
**Why?** Monitoring enables proactive maintenance  
**Why?** Metrics guide optimization  
**Why?** Performance tracking ensures SLAs  
**Why?** Usage patterns inform scaling decisions  
**Why?** Anomaly detection prevents outages

#### Requirements
1. Error and Recovery Metrics
   - Error rates by category (Connection, Transaction, Data)
   - Retry attempt distribution
   - Circuit breaker state changes
   - Recovery success rates
   - Correlation ID tracking

2. Performance Metrics
   - Connection acquisition timing
   - Query execution duration
   - Transaction lifecycle timing
   - Pool utilization rates
   - Resource cleanup timing

3. Resource Usage Metrics
   - Active connections count
   - Session lifecycle duration
   - Pool saturation levels
   - Connection leak detection
   - Cleanup verification stats

4. Health Check Integration
   - Component status verification
   - Dependency health monitoring
   - Resource availability checks
   - Performance threshold monitoring

#### Implementation Considerations

1. **Metric Collection Strategy**
   ```python
   class DatabaseMetrics:
       def __init__(self):
           self.error_counts: Dict[str, Counter] = defaultdict(Counter)
           self.timing_stats: Dict[str, Histogram] = defaultdict(Histogram)
           self.resource_gauges: Dict[str, Gauge] = defaultdict(Gauge)
           self.state_tracking: Dict[str, StateTracker] = defaultdict(StateTracker)

       async def record_operation_timing(
           self,
           operation_type: str,
           duration: float,
           context: Dict[str, Any]
       ) -> None:
           """Record operation timing with context."""
           self.timing_stats[operation_type].observe(
               duration,
               labels=context
           )

       async def track_resource_usage(
           self,
           resource_type: str,
           count: int,
           metadata: Dict[str, Any]
       ) -> None:
           """Track resource usage with metadata."""
           self.resource_gauges[resource_type].set(
               count,
               labels=metadata
           )

       async def record_error(
           self,
           error: DatabaseError,
           context: Dict[str, Any]
       ) -> None:
           """Record error with context and correlation."""
           error_type = type(error).__name__
           self.error_counts[error_type].inc(
               labels={
                   "correlation_id": error.correlation_id,
                   "retry_allowed": error.retry_allowed,
                   **context
               }
           )
   ```

2. **Integration Points**
   ```python
   class MetricsMiddleware:
       def __init__(self, metrics: DatabaseMetrics):
           self.metrics = metrics

       async def __call__(
           self,
           operation: Callable,
           *args,
           **kwargs
       ) -> Any:
           start_time = time.monotonic()
           try:
               result = await operation(*args, **kwargs)
               await self.metrics.record_operation_timing(
                   operation.__name__,
                   time.monotonic() - start_time,
                   context={"status": "success"}
               )
               return result
           except DatabaseError as e:
               await self.metrics.record_error(e, {
                   "operation": operation.__name__
               })
               raise
   ```

3. **Health Check Implementation**
   ```python
   class DatabaseHealth:
       def __init__(
           self,
           metrics: DatabaseMetrics,
           thresholds: Dict[str, float]
       ):
           self.metrics = metrics
           self.thresholds = thresholds

       async def check_health(self) -> Dict[str, Any]:
           """Perform comprehensive health check."""
           return {
               "status": await self._get_overall_status(),
               "components": {
                   "connection_pool": await self._check_pool_health(),
                   "error_rates": await self._check_error_rates(),
                   "performance": await self._check_performance(),
                   "resources": await self._check_resources()
               },
               "timestamp": datetime.now(UTC)
           }

       async def _check_error_rates(self) -> Dict[str, Any]:
           """Check error rates against thresholds."""
           error_rates = {}
           for error_type, count in self.metrics.error_counts.items():
               rate = count.rate()
               error_rates[error_type] = {
                   "rate": rate,
                   "status": "healthy" if rate < self.thresholds["error_rate"] else "degraded"
               }
           return error_rates
   ```

#### Testing Requirements

1. **Metric Accuracy Tests**
   ```python
   @pytest.mark.asyncio
   async def test_operation_timing_accuracy():
       """Test operation timing measurement accuracy."""
       metrics = DatabaseMetrics()
       operation = AsyncMock()
       middleware = MetricsMiddleware(metrics)

       # Simulate operation with known duration
       operation.side_effect = lambda: asyncio.sleep(0.1)
       await middleware(operation)

       timing = metrics.timing_stats[operation.__name__].get()
       assert 0.09 <= timing <= 0.11
   ```

2. **Resource Tracking Tests**
   ```python
   @pytest.mark.asyncio
   async def test_resource_usage_tracking():
       """Test resource usage tracking accuracy."""
       metrics = DatabaseMetrics()
       
       # Simulate pool usage
       await metrics.track_resource_usage(
           "connection_pool",
           active_count=5,
           metadata={"pool_size": 10}
       )
       
       usage = metrics.resource_gauges["connection_pool"].get()
       assert usage == 5
   ```

3. **Error Rate Tests**
   ```python
   @pytest.mark.asyncio
   async def test_error_rate_calculation():
       """Test error rate calculation accuracy."""
       metrics = DatabaseMetrics()
       
       # Simulate errors
       for _ in range(10):
           await metrics.record_error(
               ConnectionError("test"),
               {"context": "test"}
           )
       
       rate = metrics.error_counts["ConnectionError"].rate()
       assert rate == pytest.approx(10 / 60.0)  # 10 errors per minute
   ```

#### Monitoring Requirements

1. **Metric Collection**
   - Sample rates:
     - Error counts: Real-time
     - Timing stats: Every operation
     - Resource usage: Every 5 seconds
     - Health status: Every 30 seconds

2. **Alerting Thresholds**
   - Error rates > 1% per minute
   - Connection acquisition time > 100ms
   - Pool utilization > 80%
   - Circuit breaker open > 5 minutes
   - Failed cleanup operations > 0

3. **Metric Retention**
   - Error logs: 30 days
   - Timing metrics: 7 days
   - Resource usage: 24 hours
   - Health status: 72 hours

#### Validation Checklist

Before considering Phase 5 complete:

1. [ ] Metric collection implemented and tested
2. [ ] Integration with error handling verified
3. [ ] Resource tracking accuracy validated
4. [ ] Health checks implemented
5. [ ] Alerting thresholds configured
6. [ ] Metric retention policies set
7. [ ] Performance impact measured
8. [ ] Documentation updated

#### Common Pitfalls to Avoid

1. **Metric Collection**
   - DON'T collect metrics without context
   - DON'T ignore metric collection errors
   - DON'T collect high-cardinality labels
   - DON'T sample critical metrics

2. **Performance Impact**
   - DON'T block operations for metric collection
   - DON'T collect metrics too frequently
   - DON'T store raw values indefinitely
   - DON'T ignore collection overhead

3. **Integration**
   - DON'T duplicate metric collection
   - DON'T miss error contexts
   - DON'T ignore correlation IDs
   - DON'T collect metrics without labels

4. **Testing**
   - DON'T skip metric accuracy tests
   - DON'T ignore timing precision
   - DON'T mock critical metrics
   - DON'T test without load simulation

====================================================

### Phase 6: Testing Infrastructure
**Why?** Testing ensures reliability  
**Why?** Test fixtures enable consistent testing  
**Why?** Mocking enables isolated testing  
**Why?** Test categories guide development  
**Why?** Test coverage prevents regressions

#### Requirements
1. Comprehensive Test Fixtures
   - Database configuration fixtures
   - Connection pool fixtures
   - Session factory fixtures
   - Error injection fixtures
   - Metrics collection fixtures

2. Mock Factories
   - Async context manager mocks
   - Error simulation mocks
   - Metrics collection mocks
   - Health check mocks

3. Integration Test Support
   - Component integration tests
   - End-to-end scenarios
   - Performance test framework
   - Load test support

4. Test Utilities
   - Async test helpers
   - Resource cleanup tracking
   - Test data factories
   - Assertion helpers

#### Implementation Considerations

1. **Base Test Fixtures**
   ```python
   @pytest.fixture
   async def db_config():
       """Database configuration fixture."""
       return DatabaseSettings(
           DB_URL="postgresql+asyncpg://test:test@localhost:5432/test",
           POOL_SIZE=5,
           POOL_TIMEOUT=30
       )

   @pytest.fixture
   async def db_engine(db_config):
       """Database engine fixture with metrics."""
       metrics = DatabaseMetrics()
       engine = create_engine_with_metrics(
           db_config,
           metrics=metrics
       )
       try:
           yield engine
       finally:
           await engine.dispose()

   @pytest.fixture
   async def db_session(db_engine):
       """Database session fixture with error tracking."""
       async with create_tracked_session(db_engine) as session:
           yield session
           await session.rollback()
   ```

2. **Mock Factories**
   ```python
   class AsyncContextManagerMock:
       """Factory for async context manager mocks."""
       
       @classmethod
       def create(
           cls,
           return_value: Any = None,
           side_effect: Optional[Exception] = None,
           cleanup: Optional[Callable] = None
       ):
           """Create an async context manager mock.
           
           Args:
               return_value: Value to return from __aenter__
               side_effect: Exception to raise if any
               cleanup: Cleanup function to call in __aexit__
           """
           mock = AsyncMock()
           
           async def async_enter():
               if side_effect:
                   raise side_effect
               return return_value or mock
               
           async def async_exit(exc_type, exc, tb):
               if cleanup:
                   await cleanup()
               return False
           
           mock.__aenter__ = async_enter
           mock.__aexit__ = async_exit
           return mock

   class DatabaseErrorFactory:
       """Factory for database error simulation."""
       
       @classmethod
       def connection_error(
           cls,
           message: str = "Connection failed",
           retry_allowed: bool = True
       ) -> ConnectionError:
           return ConnectionError(
               message=message,
               correlation_id="test-correlation-id",
               retry_allowed=retry_allowed
           )
   ```

3. **Integration Test Base**
   ```python
   class DatabaseIntegrationTest:
       """Base class for database integration tests."""
       
       @pytest.fixture(autouse=True)
       async def setup_database(self):
           """Setup test database with all components."""
           self.metrics = DatabaseMetrics()
           self.config = DatabaseSettings()
           self.engine = create_test_engine(
               self.config,
               metrics=self.metrics
           )
           self.health = DatabaseHealth(
               self.metrics,
               thresholds={"error_rate": 0.01}
           )
           try:
               yield
           finally:
               await self.cleanup_database()
       
       async def cleanup_database(self):
           """Cleanup all database resources."""
           await self.engine.dispose()
   ```

4. **Test Utilities**
   ```python
   class AsyncTestUtils:
       """Utilities for async testing."""
       
       @staticmethod
       async def wait_for_condition(
           condition: Callable[[], bool],
           timeout: float = 1.0,
           interval: float = 0.1
       ) -> bool:
           """Wait for a condition to become true.
           
           Args:
               condition: Function that returns bool
               timeout: Maximum time to wait
               interval: Check interval
           """
           start_time = time.monotonic()
           while time.monotonic() - start_time < timeout:
               if condition():
                   return True
               await asyncio.sleep(interval)
           return False

   class ResourceTracker:
       """Track resource cleanup in tests."""
       
       def __init__(self):
           self.resources: Set[str] = set()
           self.cleanup_called: Set[str] = set()
       
       def register(self, resource_id: str) -> None:
           """Register a resource for tracking."""
           self.resources.add(resource_id)
       
       def cleanup(self, resource_id: str) -> None:
           """Mark resource as cleaned up."""
           self.cleanup_called.add(resource_id)
       
       def verify_all_cleaned(self) -> bool:
           """Verify all resources were cleaned up."""
           return self.resources == self.cleanup_called
   ```

#### Testing Requirements

1. **Unit Test Coverage**
   - Each component must have dedicated unit tests
   - Mock all external dependencies
   - Test both success and failure paths
   - Verify resource cleanup

2. **Integration Test Coverage**
   - Test component interactions
   - Verify error propagation
   - Test metric collection
   - Validate health checks

3. **Performance Test Coverage**
   - Connection pool behavior
   - Session management
   - Error handling overhead
   - Metric collection impact

4. **Load Test Coverage**
   - Concurrent operations
   - Resource limits
   - Error rates under load
   - Recovery behavior

#### Validation Checklist

Before considering Phase 6 complete:

1. [ ] All test fixtures implemented and documented
2. [ ] Mock factories cover all use cases
3. [ ] Integration tests verify component interactions
4. [ ] Performance tests validate efficiency
5. [ ] Load tests confirm stability
6. [ ] Resource cleanup verified
7. [ ] Error scenarios covered
8. [ ] Metrics collection validated

#### Common Pitfalls to Avoid

1. **Test Isolation**
   - DON'T share state between tests
   - DON'T rely on test execution order
   - DON'T leave resources uncleaned
   - DON'T ignore test side effects

2. **Mock Usage**
   - DON'T mock what you should integrate
   - DON'T create complex mock chains
   - DON'T ignore mock verification
   - DON'T use sync mocks for async code

3. **Resource Management**
   - DON'T skip cleanup verification
   - DON'T ignore cleanup failures
   - DON'T leak test resources
   - DON'T mix test and production resources

4. **Test Design**
   - DON'T write tests without assertions
   - DON'T ignore edge cases
   - DON'T skip error scenarios
   - DON'T write non-deterministic tests

====================================================

## Success Criteria
1. All tests pass with 100% coverage  
2. No resource leaks in long-running tests  
3. Error handling covers all edge cases  
4. Metrics provide actionable insights  
5. Documentation is complete and clear

## Implementation Notes
- Follow TDD approach for each phase  
- Write tests before implementation  
- Document all public interfaces  
- Include error handling in every layer  
- Maintain backward compatibility  
- Consider performance implications  
- Use type hints consistently  
- Follow SOLID principles

====================================================

## Additional Technical Specifications and Updates

### Technical Stack Versions

**Core Dependencies**  
- FastAPI: ^0.104.1  
- SQLAlchemy: ^2.0.23  
- asyncpg: ^0.29.0  
- Pydantic: ^2.5.2

**Observability Stack**  
- Logging: `structlog` ^23.2.0 for structured logging  
- Metrics: OpenTelemetry with Prometheus exporter  
  - opentelemetry-api: ^1.21.0  
  - opentelemetry-sdk: ^1.21.0  
  - opentelemetry-instrumentation-fastapi: ^0.42b0

### Performance Requirements

**API Response Times**  
- 95th percentile response time: < 200ms  
- 99th percentile response time: < 500ms  
- Maximum response time: < 1000ms

**Database Performance**  
- Query execution time: < 100ms for 95% of queries  
- Connection pool size: 20 connections (configurable)  
- Connection timeout: 10 seconds  
- Statement timeout: 30 seconds

**Monitoring Thresholds**  
- Error Rate Alert: > 1% of requests  
- Response Time Alert: > 500ms average over 5 minutes  
- Database Connection Alert: > 80% pool utilization

### API Specifications

**Health Check Endpoints**
```
GET /health
Response 200:
{
    "status": "healthy",
    "timestamp": "2023-12-18T12:00:00Z",
    "version": "1.0.0",
    "dependencies": {
        "database": "healthy",
        "cache": "healthy"
    }
}
```

**Metrics Endpoint**
```
GET /metrics
Content-Type: text/plain
Response 200: Prometheus-formatted metrics
```

**Logging Format**
```json
{
    "timestamp": "ISO8601",
    "level": "INFO|WARNING|ERROR",
    "event": "event.name",
    "trace_id": "uuid",
    "span_id": "uuid",
    "service": "service_name",
    "message": "Human readable message",
    "context": {
        "additional": "context",
        "fields": "here"
    }
}
```

### Acceptance Criteria

**Database Connection Management**
- On startup:  
  - Must verify database connection before accepting requests  
  - Must fail fast with clear error if DB_URL is invalid  
  - Must log connection pool statistics

- Runtime:  
  - Must implement connection retry with exponential backoff  
  - Must implement circuit breaker pattern for database failures  
  - Must gracefully handle connection pool exhaustion

- Shutdown:  
  - Must gracefully close all database connections  
  - Must wait for in-flight queries to complete (max 30s)  
  - Must log any queries that exceeded timeout

### Edge Cases

**Connection Handling**
- Invalid connection string: Immediate failure with detailed error  
- Connection timeout: Retry 3 times with exponential backoff  
- Pool exhaustion: Queue requests up to 100ms, then fail fast  
- Partial failure: Implement circuit breaker with 50% error threshold

**Query Execution**
- Long-running queries: Cancel after statement timeout  
- Deadlock detection: Retry transaction up to 3 times  
- Invalid data: Return 422 with detailed validation errors  
- Concurrent updates: Implement optimistic locking

### Data Flow and Security

**Data Flow**
1. Request → FastAPI Router  
2. Router → Dependency Injection  
3. Service Layer → Repository  
4. Repository → Database  
5. Response ← Transform/Serialize ← Database Result

**Security Requirements**
- Store database credentials in environment variables  
- Database password min length: 16 chars  
- TLS required for all database connections  
- Connection string must not be logged or exposed in errors  
- Implement row-level security where applicable  
- Use prepared statements for all queries

**Data Privacy**
- PII must be encrypted at rest using AES-256  
- Sensitive data must be redacted in logs  
- Implement audit logging for all data modifications  
- Implement data retention policies as defined per model

### Error Handling

**Database Errors**
- Connection errors:
  - Retry with exponential backoff  
  - Maximum 3 retries  
  - Log each retry attempt  
  - Return 503 if all retries fail

- Query errors:
  - Log full error context (except sensitive data)  
  - Return appropriate HTTP status:  
    - Constraint violation: 409  
    - Not found: 404  
    - Validation error: 422  
    - Server error: 500

====================================================

example database model: /models/user.py
