# Handoff: 004-middleware-stack

## Context for Next Feature

The middleware stack is now in place with correlation ID tracking and Prometheus metrics.

## Available for Use

### Correlation ID
```python
from stoat_ferret.api.middleware.correlation import get_correlation_id

# In any async context during a request
correlation_id = get_correlation_id()
```

The correlation ID is automatically:
- Generated as a UUID if not provided in request
- Extracted from `X-Correlation-ID` header if present
- Added to response `X-Correlation-ID` header
- Stored in contextvar for access during request processing

### Metrics

Available at `GET /metrics` in Prometheus exposition format.

Current metrics:
- `http_requests_total{method, path, status}` - Request counter
- `http_request_duration_seconds{method, path}` - Request duration histogram

## Integration Points

- Use `get_correlation_id()` in structured logging to correlate log entries
- Add custom metrics by importing from `prometheus_client`
- Middleware order in `app.py`: MetricsMiddleware (outer) -> CorrelationIdMiddleware (inner)

## Testing

The `client` fixture in `tests/test_api/conftest.py` automatically includes middleware.
Tests can check headers and metrics:

```python
def test_example(client):
    response = client.get("/some/endpoint")
    assert "X-Correlation-ID" in response.headers
```
