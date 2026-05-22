# MYRA Release Notes — v1.0.0

**Release Date:** 2024  
**Platform:** Android 8.0+ (API 26+)  
**Backend:** Python 3.11+ / FastAPI

---

## What's New in v1.0.0 — Initial Release

### 🎙️ Voice Assistant Core
- Full-duplex voice conversation with AI (OpenAI GPT-4o)
- Real-time speech-to-text via OpenAI Whisper
- Human-like text-to-speech with voice selection (Nova, Alloy, Echo, Fable, Onyx, Shimmer)
- Streaming AI responses for low-latency UX
- Wake-word detection ("Hey Myra")
- Voice interruption — tap/speak to stop response

### 🤖 AI Capabilities
- Contextual conversation memory (up to 20 messages)
- Personality modes: Professional, Friendly, Concise, Detailed, Humorous
- Structured command extraction from natural language
- Multi-turn conversation support
- Session persistence across app restarts

### 📱 Device Control
- **Phone calls** — "Call mom", "Call John Smith"
- **SMS messages** — "Text Sarah I'll be 10 minutes late"
- **App launcher** — "Open YouTube", "Open Spotify"
- **Device settings** — brightness, volume, WiFi, Bluetooth, ringer mode
- **Alarms & Timers** — "Set alarm for 7 AM", "Set a 5 minute timer"
- **Navigation** — "Navigate to Starbucks"
- **Web search** — "Search for best Italian restaurants"
- **Music playback** — "Play some jazz music"
- **Weather** — "What's the weather today?"
- **Calculator** — "What is 15% of 240?"

### 🔔 System Integration
- Notification reading via Notification Listener Service
- Floating overlay assistant button
- Accessibility Service for advanced device control
- Foreground service for background listening
- Boot receiver for auto-start
- Persistent notification for quick access

### 💬 Conversation UI
- Real-time transcript display with streaming
- Full conversation history with scrollback
- Text input fallback when voice unavailable
- Per-session conversation management
- Session deletion and history management

### ⚙️ Settings & Configuration
- Configurable server URL (self-host support)
- Voice speed, pitch, volume controls
- Language selection
- Silence threshold adjustment
- Persistent notification toggle
- Haptic feedback and sound effects
- Debug mode

### 🔐 Security
- JWT authentication with refresh token rotation
- Encrypted local token storage (EncryptedStorage)
- bcrypt password hashing
- Network security config (enforces HTTPS in production)
- ProGuard obfuscation in release builds

### 🏗️ Backend
- FastAPI async backend with WebSocket support
- PostgreSQL for persistent conversation storage
- Redis for caching and rate limiting
- Full REST API for auth, conversations, and commands
- Docker Compose deployment
- Nginx reverse proxy configuration
- Rate limiting middleware

---

## Architecture Highlights

```
MYRA Android App
├── React Native (TypeScript)
│   ├── Redux Toolkit + Redux Persist
│   ├── Socket.io WebSocket client
│   ├── React Navigation 6
│   └── Zustand (local UI state)
├── Native Android (Kotlin)
│   ├── DeviceControlModule (calls, SMS, apps, settings)
│   ├── OverlayModule (floating button)
│   ├── ContactsNativeModule (contact lookup)
│   ├── MyraAccessibilityService
│   ├── MyraNotificationListenerService
│   └── MyraForegroundService
└── MYRA Python Backend
    ├── FastAPI + uvicorn
    ├── SQLAlchemy async (PostgreSQL)
    ├── Redis caching
    ├── OpenAI GPT-4o + Whisper + TTS
    └── Docker deployment
```

---

## Known Limitations

- Wake word detection requires microphone permission and foreground service
- WiFi/Bluetooth toggle requires Android 10+ Settings Panel (direct toggle deprecated)
- Write Settings permission must be granted manually for brightness control
- Accessibility Service must be enabled manually in Android Settings
- Overlay permission must be granted manually

---

## Minimum Requirements

| Requirement | Value |
|-------------|-------|
| Android version | 8.0 (API 26) |
| RAM | 2 GB |
| Storage | 100 MB |
| Network | WiFi or 4G/5G |
| Microphone | Required for voice |
| Internet | Required for AI |

---

## Upcoming (v1.1.0)

- [ ] On-device wake word detection (PocketSphinx / Vosk)
- [ ] Offline STT fallback
- [ ] Smart home integration (Google Home, Alexa)
- [ ] Calendar integration
- [ ] Email reading and composition
- [ ] Multi-language support
- [ ] Custom wake word training
- [ ] iOS support
