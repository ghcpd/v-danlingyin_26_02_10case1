# Shared Backend Utilities

This project contains shared utility functions used by backend services.

## Installation

```bash
npm install shared-utils
```

## API Documentation

### Overview

This library exports utilities for implementing rate limiting on backend services. All rate limiting functions are defined in [rateLimiter.ts](rateLimiter.ts).

### Configuration

#### `RateLimitConfig`

An interface that defines the configuration parameters for rate limiting.

**Properties:**
- `maxRequests` (number): The maximum number of requests allowed within a single time window.
- `windowMs` (number): The duration of the rate limit window in milliseconds.

**Example:**
```typescript
const config: RateLimitConfig = {
  maxRequests: 10,
  windowMs: 60000  // 1 minute window
};
```

### Functions

#### `checkRateLimit(key: string, config: RateLimitConfig): boolean`

Check whether a request identified by the given key is allowed under the current rate limit configuration.

**Parameters:**
- `key` (string): A unique identifier for the request (e.g., user ID, IP address, or session ID).
- `config` (RateLimitConfig): Rate limiting configuration object.

**Returns:**
- `true` if the request is allowed (has not exceeded the rate limit).
- `false` if the request should be rejected (has exceeded the rate limit).

**Behavior:**
- Tracks request counts per key within rolling time windows.
- Creates a new window if none exists or if the current window has expired.
- Increments the count for existing windows and returns the result.

**Example:**
```typescript
import { checkRateLimit, RateLimitConfig } from './rateLimiter';

const config: RateLimitConfig = { maxRequests: 10, windowMs: 60000 };

if (checkRateLimit('user123', config)) {
  console.log('Request allowed');
  // Process the request
} else {
  console.log('Rate limit exceeded');
  // Reject the request
}
```

#### `resetRateLimit(key: string): void`

Reset the rate limit data for a specific key. This clears all recorded request history for the given identifier.

**Parameters:**
- `key` (string): The unique identifier to reset (must match a key previously used in `checkRateLimit`).

**Effect:**
- Removes the request record associated with the key from the internal store.
- The next call to `checkRateLimit` with the same key will be treated as a fresh window.

**Example:**
```typescript
import { resetRateLimit } from './rateLimiter';

// Later, clear rate limit history for a user
resetRateLimit('user123');
```

## Usage Pattern

```typescript
import { checkRateLimit, resetRateLimit, RateLimitConfig } from './rateLimiter';

// Configuration: 100 requests per 15 minutes
const config: RateLimitConfig = {
  maxRequests: 100,
  windowMs: 15 * 60 * 1000  // 15 minutes
};

// Check rate limit for an incoming request
const userId = req.user.id;
if (!checkRateLimit(userId, config)) {
  res.status(429).json({ error: 'Too many requests' });
  return;
}

// Process the request...
// Later, if needed, reset the rate limit for a user
resetRateLimit(userId);
```