# API Reference

## Availability

There is currently no HTTP API endpoint. AWS invokes the Python handler directly. An API Gateway, Lambda Function URL, or event source mapping is not defined in this repository.

## Handler

- **Module:** `function.lambda_function`
- **Function:** `lambda_handler(event, context)`
- **Invocation type:** AWS Lambda event invocation

## Request Contract

The handler expects a JSON-like mapping with a required `input` property:

```json
{
  "input": "Hello"
}
```

`context` is accepted for Lambda compatibility and is not currently used.

## Successful Response

For exactly `input: "Hello"`, the handler returns the string:

```text
World
```

## Error Behavior

- Missing `input` causes a `KeyError`.
- Any value other than `Hello` reaches a bare `raise` and fails the invocation. This should be replaced with a deliberate exception and a documented error schema before exposing the function to external clients.
- The startup message is written to standard output and therefore appears in CloudWatch Logs when running on Lambda.

## Local Example

```python
from function.lambda_function import lambda_handler

assert lambda_handler({"input": "Hello"}, None) == "World"
```

## Future HTTP Mapping

If API Gateway is added, a recommended mapping is:

| Method | Path | Request | Success |
|---|---|---|---|
| `POST` | `/hello` | `{ "input": "Hello" }` | HTTP 200 with `{ "message": "World" }` |

That mapping is a proposal, not a currently deployed endpoint. Authentication, throttling, CORS, request validation, and a stable error format must be defined before publication.
