# MYRA AI Voice Assistant — Build Guide

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Node.js | 18+ | https://nodejs.org |
| React Native CLI | latest | `npm i -g react-native-cli` |
| Java JDK | 17 | https://adoptium.net |
| Android Studio | Hedgehog+ | https://developer.android.com/studio |
| Android SDK | API 34 | via Android Studio SDK Manager |
| Python | 3.11+ | https://python.org |
| Docker & Compose | latest | https://docker.com |

---

## 1. Clone & Install

```bash
git clone https://github.com/your-org/myra.git
cd myra
npm install
```

---

## 2. Environment Setup

### Android SDK
```bash
# Add to ~/.bashrc or ~/.zshrc
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
```

### Backend .env
```bash
cp backend/.env.example backend/.env
# Edit backend/.env — set OPENAI_API_KEY and JWT_SECRET_KEY
```

---

## 3. Start Backend (Docker — Recommended)

```bash
# Start all services (PostgreSQL, Redis, API)
docker-compose up -d

# Check health
curl http://localhost:8000/health/

# View logs
docker-compose logs -f api
```

### Backend (Manual — Development)
```bash
cd backend
pip install -r requirements.txt
uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## 4. Build Android (Debug)

```bash
# Start Metro bundler
npx react-native start

# In separate terminal — run on device/emulator
npx react-native run-android

# Or build debug APK
cd android && ./gradlew assembleDebug
# Output: android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 5. Build Android (Release APK)

### 5a. Generate Keystore (first time only)
```bash
cd android/app
keytool -genkeypair -v \
  -storetype PKCS12 \
  -keystore myra-release.keystore \
  -alias myra-key \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# Keep the keystore SAFE — you need it for all future updates
```

### 5b. Configure signing credentials
Create `android/gradle.properties` additions:
```properties
MYRA_UPLOAD_STORE_FILE=myra-release.keystore
MYRA_UPLOAD_KEY_ALIAS=myra-key
MYRA_UPLOAD_STORE_PASSWORD=<your_store_password>
MYRA_UPLOAD_KEY_PASSWORD=<your_key_password>
```

### 5c. Bundle JS assets
```bash
npx react-native bundle \
  --platform android \
  --dev false \
  --entry-file index.js \
  --bundle-output android/app/src/main/assets/index.android.bundle \
  --assets-dest android/app/src/main/res/
```

### 5d. Build release APK
```bash
cd android
./gradlew clean
./gradlew assembleRelease

# Output: android/app/build/outputs/apk/release/app-release.apk
```

### 5e. Build release AAB (for Play Store)
```bash
cd android && ./gradlew bundleRelease
# Output: android/app/build/outputs/bundle/release/app-release.aab
```

---

## 6. Install APK on Device

```bash
# Enable USB debugging on device, then:
adb install -r android/app/build/outputs/apk/release/app-release.apk

# Or direct install
adb install android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 7. Configure App Server URL

Before building, update the server URL in `backend/.env` and ensure it matches
what you put in `android/app/build.gradle` under `buildConfigField`.

For emulator testing: `http://10.0.2.2:8000`  
For physical device: `http://<your-machine-ip>:8000`  
For production: `https://api.myra.ai`

---

## 8. Required Android Permissions (Grant on device)

After installing:
1. Open MYRA
2. Grant all requested permissions
3. Go to Settings → Accessibility → MYRA → Enable
4. Go to Settings → Apps → MYRA → Draw over other apps → Enable
5. Go to Settings → Notifications → MYRA Notification Access → Enable

---

## 9. Run Tests

```bash
# Frontend tests
npm test

# Backend tests
cd backend
pytest tests/ -v --cov=backend --cov-report=term-missing
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `SDK location not found` | Set `ANDROID_HOME` env var |
| `Gradle build failed` | Run `cd android && ./gradlew clean` |
| `Metro bundler error` | Delete `node_modules`, run `npm install` |
| `WebSocket not connecting` | Check server URL in Settings |
| `Voice not working` | Grant microphone permission |
| `Can't make calls` | Grant phone permission |
