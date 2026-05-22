# MYRA API Documentation

**Base URL:** `https://api.myra.ai`  
**WebSocket:** `wss://api.myra.ai/ws/connect`  
**Auth:** Bearer JWT token in `Authorization` header

---

## Authentication

### POST `/api/v1/auth/register`
Register a new user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securepassword",
  "name": "Jane Smith"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "user": { "id": "uuid", "email": "...", "name": "...", "isPremium": false },
    "token": "eyJhbGci...",
    "refreshToken": "uuid-uuid",
    "expiresIn": 3600
  }
}
```

---

### POST `/api/v1/auth/login`
Login with email and password.

**Request:**
```json
{ "email": "user@example.com", "password": "securepassword" }
```

**Response 200:** Same as register

---

### POST `/api/v1/auth/refresh`
Refresh an access token.

**Request:**
```json
{ "refreshToken": "uuid-uuid" }
```

---

### POST `/api/v1/auth/logout`
Revoke refresh token and end session.

**Headers:** `Authorization: Bearer <token>`

---

### GET `/api/v1/auth/profile`
Get current user profile.

**Headers:** `Authorization: Bearer <token>`

---

### PATCH `/api/v1/auth/profile`
Update user preferences.

**Request:**
```json
{
  "name": "Jane",
  "language": "en-US",
  "voice_id": "nova",
  "theme": "dark"
}
```

---

## Conversation

### GET `/api/v1/conversation/sessions`
List conversation sessions.

**Query params:** `limit=20`, `offset=0`

---

### GET `/api/v1/conversation/sessions/{session_id}/messages`
Get messages in a session.

---

### DELETE `/api/v1/conversation/sessions/{session_id}`
Delete a session and all its messages.

---

### POST `/api/v1/conversation/query`
Non-streaming text query (for simple integrations).

**Request:**
```json
{
  "text": "What is the weather today?",
  "personality": "friendly",
  "stream": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "response": "I'll check the weather for you...",
    "commands": [{ "type": "weather", "payload": {} }],
    "tokensUsed": 120
  }
}
```

---

## Commands

### POST `/api/v1/commands/execute`
Execute a device command server-side.

**Request:**
```json
{
  "type": "calculate",
  "payload": { "expression": "42 * 7" },
  "rawText": "calculate 42 times 7"
}
```

### GET `/api/v1/commands/types`
List all supported command types.

---

## Health

### GET `/health/`
Basic health check — no auth required.

### GET `/health/detailed`
Database + Redis + AI health check.

---

## WebSocket Protocol

### Connection
Connect to `wss://api.myra.ai/ws/connect`

**First message must be auth:**
```json
{
  "type": "auth",
  "payload": { "token": "eyJhbGci..." },
  "timestamp": "2024-01-01T00:00:00Z",
  "messageId": "uuid"
}
```

**Server response:**
```json
{
  "type": "auth_success",
  "payload": { "userId": "uuid", "sessionId": "uuid" }
}
```

---

### Client → Server Events

| Type | Payload | Description |
|------|---------|-------------|
| `auth` | `{token}` | Authenticate (first message) |
| `session_start` | `{personality, sessionId?}` | Start new conversation session |
| `text_message` | `{text, sessionId?}` | Send text query |
| `audio_stream` | `{audio: base64, sessionId?}` | Send audio chunk for STT |
| `interrupt` | `{sessionId}` | Interrupt current AI response |
| `ping` | `{timestamp}` | Heartbeat |

---

### Server → Client Events

| Type | Payload | Description |
|------|---------|-------------|
| `auth_success` | `{userId, sessionId}` | Auth confirmed |
| `session_start` | `{sessionId}` | DB session created |
| `partial_transcript` | `{text, confidence}` | Real-time STT (partial) |
| `transcript` | `{text, confidence}` | Final STT result |
| `ai_response_stream` | `{text, done}` | Streaming AI token |
| `ai_response_end` | `{fullText, audioUrl?, tokensUsed, commands}` | Response complete |
| `command` | `{commandId, command}` | Command to execute on device |
| `command_result` | `{commandId, result}` | Result of device command |
| `pong` | `{timestamp}` | Heartbeat response |
| `error` | `{message, code?}` | Error event |

---

## Command Types

| Type | Payload | Description |
|------|---------|-------------|
| `call` | `{number, name?}` | Make phone call |
| `message` | `{number, name?, message}` | Send SMS |
| `open_app` | `{packageName, appName?}` | Open Android app |
| `close_app` | `{packageName}` | Force-close app |
| `set_alarm` | `{time: "HH:MM", label?}` | Set alarm clock |
| `timer` | `{duration: seconds, label?}` | Set countdown timer |
| `reminder` | `{text, time?}` | Create reminder |
| `play_music` | `{query?, service?}` | Play music |
| `search` | `{query, engine?}` | Web search |
| `navigate` | `{destination}` | Google Maps navigation |
| `device_control` | `{action, value?}` | Device settings |
| `weather` | `{location?}` | Weather lookup |
| `calculate` | `{expression}` | Math calculation |

### Device Control Actions
| Action | Value | Description |
|--------|-------|-------------|
| `brightness` | 0–100 | Screen brightness |
| `volume` | 0–100 | Media volume |
| `wifi` | bool | Toggle WiFi |
| `bluetooth` | bool | Toggle Bluetooth |
| `ringer` | `normal\|silent\|vibrate` | Ringer mode |

---

## Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad request / validation error |
| 401 | Unauthorized — invalid or expired token |
| 403 | Forbidden |
| 404 | Resource not found |
| 409 | Conflict (e.g. email already registered) |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

**WebSocket close codes:**
| Code | Meaning |
|------|---------|
| 4001 | Auth timeout |
| 4002 | Auth message required |
| 4003 | No token provided |
| 4004 | Invalid token |
| 1001 | Idle timeout |
