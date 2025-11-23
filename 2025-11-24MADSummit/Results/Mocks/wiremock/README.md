# WireMock Mappings for Bicycle Distribution API

This directory contains WireMock mapping files for the Bicycle Distribution API v1.0.0.

## Overview

WireMock is a flexible library for stubbing and mocking web services. These mapping files define how WireMock should respond to various API requests.

## Files Included

### Success Response Mappings

1. **get-bicycles.json** - GET /v1/bicycles
    - Returns a list of bicycles with sample data
    - Supports filtering (though the stub returns static data)
    - Returns 3 sample bicycles in different statuses

2. **post-bicycles.json** - POST /v1/bicycles
    - Creates a new bicycle
    - Returns 201 Created with Location header
    - Validates that model and serialNumber are present

3. **get-bicycle-by-id.json** - GET /v1/bicycles/{bicycleId}
    - Returns a single bicycle by ID
    - Uses response templating to echo back the bicycle ID from the URL
    - Matches any valid UUID format

4. **put-bicycle-by-id.json** - PUT /v1/bicycles/{bicycleId}
    - Updates a bicycle
    - Returns 200 OK with updated bicycle data
    - Uses response templating to include request body data in response

5. **get-bicycle-status.json** - GET /v1/bicycles/{bicycleId}/status
    - Returns the status of a bicycle
    - Uses dynamic timestamp generation

6. **put-bicycle-status.json** - PUT /v1/bicycles/{bicycleId}/status
    - Updates the status of a bicycle
    - Returns 200 OK with updated status info
    - Echoes back the status and location from request body

### Error Response Mappings

7. **error-404-not-found.json** - 404 Not Found
    - Triggered when requesting bicycle ID: 00000000-0000-0000-0000-000000000000
    - Returns standard error response format

8. **error-400-bad-request.json** - 400 Bad Request
    - Triggered when POST /bicycles is called without required 'model' field
    - Returns validation error message

9. **error-401-unauthorized.json** - 401 Unauthorized
    - Triggered when Authorization header is missing
    - Returns authentication required error
    - Has lower priority (10) to allow authenticated requests to pass through

## Setup Instructions

### Option 1: Using WireMock Standalone

**Run from the `wiremock` directory** (where this README is located):

1. Download WireMock standalone JAR:
   ```bash
   curl -o wiremock-standalone.jar https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.3.1/wiremock-standalone-3.3.1.jar
   ```

2. Start WireMock:
   ```bash
   java -jar wiremock-standalone.jar --port 8080 --global-response-templating
   ```

   Note: WireMock automatically uses the `mappings/` subdirectory in the current directory.

### Option 2: Using Docker

**Run from the `wiremock` directory** (where this README is located):

```bash
docker run -it --rm \
  -p 8080:8080 \
  -v $PWD:/home/wiremock \
  wiremock/wiremock:3.3.1
```

### Option 3: Using WireMock in Spring Boot Test

Add dependency to your `pom.xml`:
```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock-standalone</artifactId>
    <version>3.3.1</version>
    <scope>test</scope>
</dependency>
```

Use in your test:
```java
@SpringBootTest
@AutoConfigureWireMock(port = 0)
class BicycleApiTest {
    // Tests here
}
```

## Testing the API

Once WireMock is running, you can test the endpoints:

### Get all bicycles
```bash
curl -H "x-api-version: 1.0.0" http://localhost:8080/v1/bicycles
```

### Create a bicycle
```bash
curl -X POST http://localhost:8080/v1/bicycles \
  -H "Content-Type: application/json" \
  -H "x-api-version: 1.0.0" \
  -d '{
    "model": "City Bike Pro 2025",
    "serialNumber": "BIKE-2025-999999",
    "description": "Test bicycle",
    "initialLocation": "Garage-Central"
  }'
```

### Get bicycle by ID
```bash
curl -H "x-api-version: 1.0.0" \
  http://localhost:8080/v1/bicycles/a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d
```

### Update bicycle
```bash
curl -X PUT http://localhost:8080/v1/bicycles/a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d \
  -H "Content-Type: application/json" \
  -H "x-api-version: 1.0.0" \
  -d '{
    "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
    "model": "City Bike Pro 2025",
    "serialNumber": "BIKE-2025-001234",
    "status": "IN_HOTEL",
    "currentLocation": "Grand Hotel Munich"
  }'
```

### Get bicycle status
```bash
curl -H "x-api-version: 1.0.0" \
  http://localhost:8080/v1/bicycles/a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d/status
```

### Update bicycle status
```bash
curl -X PUT http://localhost:8080/v1/bicycles/a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d/status \
  -H "Content-Type: application/json" \
  -H "x-api-version: 1.0.0" \
  -d '{
    "status": "IN_GARAGE",
    "location": "Garage-Central",
    "notes": "Bicycle returned for maintenance"
  }'
```

### Test error responses

**404 Not Found:**
```bash
curl -H "x-api-version: 1.0.0" \
  http://localhost:8080/v1/bicycles/00000000-0000-0000-0000-000000000000
```

**400 Bad Request (missing model):**
```bash
curl -X POST http://localhost:8080/v1/bicycles \
  -H "Content-Type: application/json" \
  -H "x-api-version: 1.0.0" \
  -d '{"serialNumber": "BIKE-2025-999999"}'
```

**401 Unauthorized (no auth header):**
```bash
curl -H "x-api-version: 1.0.0" http://localhost:8080/v1/bicycles
# Note: This won't trigger 401 in the current setup as the stub doesn't check for auth
# You can remove the Authorization check from the stub or add it to your requests
```

## Features Used

### Response Templating
Several mappings use WireMock's response templating feature:
- `{{request.pathSegments.[2]}}` - Extracts the bicycle ID from URL path
- `{{jsonPath request.body '$.field'}}` - Extracts fields from request body
- `{{now format='yyyy-MM-dd'T'HH:mm:ss'Z'}}` - Generates current timestamp

### Priority
Error mappings use `priority: 10` to ensure they are matched before the general success mappings (default priority is 5).

### URL Pattern Matching
- `urlPath` - Exact path matching
- `urlPathPattern` - Regex pattern matching (used for UUID matching)

### Body Matching
- `matchesJsonPath` - Validates JSON structure
- Request body patterns ensure required fields are present

## Customization

To customize the responses:

1. Edit the JSON files to change response data
2. Add new mappings by creating additional JSON files in the mappings directory
3. Adjust priorities to control matching order
4. Add query parameter matching using `queryParameters` in the request section

## API Version Header

All mappings require the `x-api-version: 1.0.0` header as specified in the OpenAPI spec. Requests without this header will not match the stubs.

## Notes

- These stubs return static data and don't maintain state
- For stateful behavior, consider using WireMock's scenarios feature
- The response templating requires WireMock to be started with `--global-response-templating` flag or the transformer enabled
- Authentication is simplified in these stubs - real implementation should validate tokens

## References

- [WireMock Documentation](https://wiremock.org/docs/)
- [Response Templating](https://wiremock.org/docs/response-templating/)
- [Request Matching](https://wiremock.org/docs/request-matching/)