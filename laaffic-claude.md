# LaAffic API Complete Documentation

## Overview

The LaAffic API provides programmatic access to domain and web services management functionality. This documentation is designed for AI agents to understand and implement LaAffic API calls in web applications.

**Base URL**: `[https://www.laaffic.com/api/](https://api.laaffic.com/v3/)`

## Authentication & Authorization

### Required Headers

All API requests must include the following headers:

| Header | Description | Type | Required |
|--------|-------------|------|----------|
| `Content-Type` | Must be `application/json;charset=UTF-8` | String | Yes |
| `Sign` | Encrypted MD5 signature (see signature generation) | String | Yes |
| `Timestamp` | Current system timestamp in seconds | String | Yes |
| `Api-Key` | API authentication name from App Management | String | Yes |

### Example Headers

```json
{
  "Content-Type": "application/json;charset=UTF-8",
  "Sign": "05d7a50893e22a5c4bb3216ae3396c7c",
  "Timestamp": "1630468800",
  "Api-Key": "bDqJFiq9"
}
```

### Signature Generation

The signature is generated using the following process:

1. Concatenate: `API_KEY + API_SECRET + TIMESTAMP`
2. Generate MD5 hash (32-bit, case insensitive)

**Formula**: `MD5(API_KEY + API_SECRET + TIMESTAMP)`

**Example**:
- API Key: `bDqJFiq9`
- API Secret: `7bz1lzh9` 
- Timestamp: `1630468800`
- String to hash: `bDqJFiq97bz1lzh91630468800`
- Generated signature: `05d7a50893e22a5c4bb3216ae3396c7c`

### Getting API Credentials

1. Navigate to Account Management
2. Go to App Management 
3. Find your `appId` (this becomes your `Api-Key`)
4. Get your API Secret Key

## Response Format

All API responses follow a consistent format:

### Standard Response Structure

```json
{
  "status": "string",
  "reason": "string",
  "data": {} // Optional, contains response data
}
```

### Response Parameters

| Parameter | Description | Type |
|-----------|-------------|------|
| `status` | Response status code. "0" = success, others = failure | String |
| `reason` | Description of result or failure reason | String |
| `data` | Response data (varies by endpoint) | Object |

### Success Response Example

```json
{
  "status": "0",
  "reason": "success"
}
```

### Error Response Example

```json
{
  "status": "-16",
  "reason": "timestamp expires"
}
```

## Common Status Codes

| Status Code | Description |
|-------------|-------------|
| `0` | Success |
| `-16` | Timestamp expired |
| `-1` | Invalid signature |
| `-2` | Invalid API key |
| `-3` | Missing required parameters |
| `-4` | Invalid request format |
| `-5` | Rate limit exceeded |
| `-10` | Authentication failed |

## Implementation Guidelines for AI Agents

### 1. Authentication Implementation

```javascript
// Example implementation for generating signature
function generateSignature(apiKey, apiSecret, timestamp) {
  const stringToHash = apiKey + apiSecret + timestamp;
  return md5(stringToHash).toLowerCase();
}

// Example headers setup
function getApiHeaders(apiKey, apiSecret) {
  const timestamp = Math.floor(Date.now() / 1000).toString();
  const signature = generateSignature(apiKey, apiSecret, timestamp);
  
  return {
    'Content-Type': 'application/json;charset=UTF-8',
    'Sign': signature,
    'Timestamp': timestamp,
    'Api-Key': apiKey
  };
}
```

### 2. Error Handling

AI agents should implement proper error handling:

```javascript
function handleApiResponse(response) {
  if (response.status === "0") {
    // Success - process response.data
    return { success: true, data: response.data };
  } else {
    // Error - handle based on status code
    switch (response.status) {
      case "-16":
        // Timestamp expired - regenerate and retry
        return { success: false, error: "timestamp_expired", retry: true };
      case "-1":
        // Invalid signature - check credentials
        return { success: false, error: "invalid_signature", retry: false };
      default:
        return { success: false, error: response.reason, retry: false };
    }
  }
}
```

### 3. Rate Limiting Considerations

- Implement exponential backoff for rate limit errors
- Cache API responses where appropriate
- Batch requests when possible
- Monitor API usage to stay within limits

### 4. Security Best Practices

- Never expose API secrets in client-side code
- Store credentials securely (environment variables)
- Implement request timeouts
- Validate all input parameters
- Use HTTPS for all requests

## Request Methods

All API endpoints accept the following HTTP methods based on their functionality:

- `GET` - Retrieve data
- `POST` - Create new resources
- `PUT` - Update existing resources  
- `DELETE` - Remove resources

## Common Request Patterns

### Making API Calls

```javascript
async function makeApiCall(endpoint, method = 'GET', data = null) {
  const headers = getApiHeaders(API_KEY, API_SECRET);
  
  const config = {
    method: method,
    headers: headers,
    body: data ? JSON.stringify(data) : null
  };
  
  try {
    const response = await fetch(`https://www.laaffic.com/api/${endpoint}`, config);
    const result = await response.json();
    return handleApiResponse(result);
  } catch (error) {
    return { success: false, error: error.message, retry: true };
  }
}
```

## Data Validation

### Input Validation Rules

AI agents should validate the following before making API calls:

1. **Timestamp**: Must be current system time (within acceptable drift)
2. **API Key**: Must be valid alphanumeric string
3. **Signature**: Must match expected MD5 hash
4. **Content-Type**: Must be exactly `application/json;charset=UTF-8`

### Parameter Validation

- All required parameters must be present
- String parameters should be properly encoded
- Numeric parameters should be within valid ranges
- Dates should be in correct format (Unix timestamp)

## Testing and Debugging

### Test Endpoint Connectivity

```javascript
// Basic connectivity test
async function testConnection() {
  try {
    const result = await makeApiCall('health');
    return result.success;
  } catch (error) {
    console.error('Connection test failed:', error);
    return false;
  }
}
```

### Debug Mode Implementation

```javascript
const DEBUG_MODE = process.env.NODE_ENV === 'development';

function debugLog(message, data = null) {
  if (DEBUG_MODE) {
    console.log(`[LaAffic API Debug] ${message}`, data);
  }
}
```

## SDK Implementation Template

AI agents can use this template to create SDK wrappers:

```javascript
class LaAfficAPI {
  constructor(apiKey, apiSecret, baseUrl = 'https://www.laaffic.com/api/') {
    this.apiKey = apiKey;
    this.apiSecret = apiSecret;
    this.baseUrl = baseUrl;
  }
  
  generateSignature(timestamp) {
    return md5(this.apiKey + this.apiSecret + timestamp).toLowerCase();
  }
  
  getHeaders() {
    const timestamp = Math.floor(Date.now() / 1000).toString();
    return {
      'Content-Type': 'application/json;charset=UTF-8',
      'Sign': this.generateSignature(timestamp),
      'Timestamp': timestamp,
      'Api-Key': this.apiKey
    };
  }
  
  async request(endpoint, method = 'GET', data = null) {
    const response = await fetch(this.baseUrl + endpoint, {
      method,
      headers: this.getHeaders(),
      body: data ? JSON.stringify(data) : null
    });
    
    return await response.json();
  }
}
```

## Configuration Management

### Environment Variables

AI agents should use environment variables for configuration:

```bash
LAAFFIC_API_KEY=your_api_key_here
LAAFFIC_API_SECRET=your_api_secret_here
LAAFFIC_BASE_URL=https://www.laaffic.com/api/
LAAFFIC_TIMEOUT=30000
LAAFFIC_RETRY_ATTEMPTS=3
```

### Configuration Object

```javascript
const config = {
  apiKey: process.env.LAAFFIC_API_KEY,
  apiSecret: process.env.LAAFFIC_API_SECRET,
  baseUrl: process.env.LAAFFIC_BASE_URL || 'https://www.laaffic.com/api/',
  timeout: parseInt(process.env.LAAFFIC_TIMEOUT) || 30000,
  retryAttempts: parseInt(process.env.LAAFFIC_RETRY_ATTEMPTS) || 3
};
```

## Integration Examples

### Node.js Integration

```javascript
const LaAfficAPI = require('./laaffic-api');

const api = new LaAfficAPI(
  process.env.LAAFFIC_API_KEY,
  process.env.LAAFFIC_API_SECRET
);

// Example usage
async function example() {
  try {
    const result = await api.request('endpoint');
    if (result.status === "0") {
      console.log('Success:', result.data);
    } else {
      console.error('API Error:', result.reason);
    }
  } catch (error) {
    console.error('Request failed:', error);
  }
}
```

### Python Integration

```python
import hashlib
import time
import json
import requests

class LaAfficAPI:
    def __init__(self, api_key, api_secret, base_url='https://www.laaffic.com/api/'):
        self.api_key = api_key
        self.api_secret = api_secret
        self.base_url = base_url
    
    def generate_signature(self, timestamp):
        string_to_hash = f"{self.api_key}{self.api_secret}{timestamp}"
        return hashlib.md5(string_to_hash.encode()).hexdigest()
    
    def get_headers(self):
        timestamp = str(int(time.time()))
        return {
            'Content-Type': 'application/json;charset=UTF-8',
            'Sign': self.generate_signature(timestamp),
            'Timestamp': timestamp,
            'Api-Key': self.api_key
        }
    
    def request(self, endpoint, method='GET', data=None):
        url = self.base_url + endpoint
        headers = self.get_headers()
        
        response = requests.request(
            method=method,
            url=url,
            headers=headers,
            json=data if data else None
        )
        
        return response.json()
```

## Best Practices for AI Agents

1. **Credential Management**: Always store credentials securely
2. **Error Handling**: Implement comprehensive error handling
3. **Retry Logic**: Use exponential backoff for transient errors
4. **Logging**: Log all API interactions for debugging
5. **Caching**: Cache responses when appropriate
6. **Validation**: Validate all inputs before sending requests
7. **Monitoring**: Monitor API usage and performance
8. **Documentation**: Keep integration documentation updated

## Troubleshooting Guide

### Common Issues

1. **Signature Mismatch**
   - Verify API key and secret are correct
   - Check timestamp generation
   - Ensure proper string concatenation

2. **Timestamp Expired**
   - Synchronize system time
   - Account for network latency
   - Regenerate timestamp for retry

3. **Invalid API Key**
   - Verify credentials in App Management
   - Check for typos in API key
   - Ensure key is active

4. **Rate Limit Exceeded**
   - Implement exponential backoff
   - Monitor request frequency
   - Consider request batching

## Support and Resources

- API Documentation: https://www.laaffic.com/api/
- Account Management: Access through LaAffic dashboard
- App Management: Configure API credentials

---

*Note: This documentation is based on the available LaAffic API specification. AI agents should always refer to the latest official documentation for the most current information and any additional endpoints that may be available.*
