# Voice Analytics Platform - Server Setup & Run Guide

This guide contains all instructions to run both the backend and frontend servers for the ChefVision Voice Analytics Platform.

## 📁 Project Structure
```
chefvision-analytics-platform/
├── transcription-platform/          # Django Backend
│   ├── venv/                        # Virtual environment
│   ├── manage.py                    # Django management script
│   ├── requirements.txt             # Python dependencies
│   └── ...
├── expo-voice-analytics-mobile-app/ # React Native Frontend
│   ├── package.json                 # Node.js dependencies
│   ├── App.tsx                      # Main app component
│   └── src/                         # Source code
└── RUN_SERVERS.md                   # This file
```

## 🚀 Quick Start Commands

### Option 1: Complete System (Recommended)
```bash
# From the root directory (chefvision-analytics-platform/)
# Terminal 1 - Django Backend
cd transcription-platform && source venv/bin/activate && python manage.py runserver

# Terminal 2 - Celery Worker (Background Processing)
cd transcription-platform && source venv/bin/activate && celery -A backend worker -l info

# Terminal 3 - Frontend
cd expo-voice-analytics-mobile-app && npm start
```

### Option 2: Background Execution
```bash
# Start Redis (if not running)
redis-server &

# Start backend in background
cd transcription-platform && source venv/bin/activate && python manage.py runserver &

# Start Celery worker in background
cd transcription-platform && source venv/bin/activate && celery -A backend worker -l info &

# Start frontend in background
cd expo-voice-analytics-mobile-app && npm start &
```

## 🔧 Detailed Setup Instructions

### 🐍 Backend Setup (Django)

#### Prerequisites
- Python 3.13+ installed
- Virtual environment support
- Redis server installed and running

#### First Time Setup
```bash
cd transcription-platform

# Create virtual environment (if doesn't exist)
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run migrations (if needed)
python manage.py migrate

# Create superuser (optional)
python manage.py createsuperuser
```

#### Daily Run Commands
```bash
# Terminal 1 - Django Backend
cd transcription-platform
source venv/bin/activate
python manage.py runserver

# Terminal 2 - Celery Worker (Required for AI analysis & background tasks)
cd transcription-platform
source venv/bin/activate
celery -A backend worker -l info
```

#### Backend URLs
- **API Root:** http://localhost:8000/api/
- **Admin Panel:** http://localhost:8000/admin/
- **Dashboard:** http://localhost:8000/dashboard/

### 🔧 Redis & Celery Setup (Background Processing)

#### Prerequisites
- Redis server installed
- Python virtual environment activated

#### First Time Redis Setup
```bash
# Install Redis (macOS with Homebrew)
brew install redis

# Install Redis (Ubuntu/Debian)
sudo apt-get install redis-server

# Install Redis (CentOS/RHEL)
sudo yum install redis
```

#### Daily Run Commands - Redis & Celery
```bash
# Terminal 1 - Start Redis server
redis-server

# Terminal 2 - Start Celery worker
cd transcription-platform
source venv/bin/activate
celery -A backend worker -l info
```

#### What Celery Handles
- **🤖 AI Analysis**: Automatic OpenAI/Azure analysis when transcription completes
- **📊 Insights Generation**: Dashboard metrics and team/user analytics
- **📧 Notifications**: Processing and sending notification emails
- **🏷️ Template Processing**: Analysis template application and management
- **🧹 Cleanup Tasks**: Old notifications and expired assignments cleanup

#### Celery Monitoring
```bash
# View active tasks
celery -A backend inspect active

# View registered tasks
celery -A backend inspect registered

# View worker stats
celery -A backend inspect stats
```

### 📱 Frontend Setup (React Native/Expo)

#### Prerequisites
- Node.js 18+ installed
- npm or yarn package manager

#### First Time Setup
```bash
cd expo-voice-analytics-mobile-app

# Install dependencies
npm install

# Update packages for compatibility (if needed)
npm install expo@~54.0.10 expo-file-system@~19.0.15 react-native-svg@15.12.1
```

#### Daily Run Commands
```bash
cd expo-voice-analytics-mobile-app
npm start
```

#### Frontend URLs
- **Metro Bundler:** http://localhost:8081
- **Web App:** http://localhost:8081 (in browser)
- **Mobile:** Use Expo Go app and scan QR code

## 🔍 Verification Steps

### ✅ Backend Health Check
```bash
# Test basic connectivity
curl -i http://localhost:8000

# Test API endpoint (should return 401 - authentication required)
curl -i http://localhost:8000/api/

# Expected response: HTTP 401 with authentication error (this is correct!)
```

### ✅ Frontend Health Check
```bash
# Test Metro bundler
curl -i http://localhost:8081

# Expected response: HTTP 200 with HTML page
```

### ✅ Redis Health Check
```bash
# Test Redis connection
redis-cli ping

# Expected response: PONG

# Check Redis info
redis-cli info server
```

### ✅ Celery Health Check
```bash
# Check worker status
celery -A backend inspect ping

# Expected response: pong from workers

# Check registered tasks
celery -A backend inspect registered

# Expected tasks:
# - apps.analysis.tasks.process_analysis_job
# - apps.analysis.tasks.process_completed_transcription
# - apps.insights.tasks.generate_team_insights
# - apps.operations.tasks.cleanup_old_notifications
```

## 🛠️ Troubleshooting

### Backend Issues

#### Virtual Environment Not Found
```bash
cd transcription-platform
rm -rf venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### Missing Dependencies
```bash
cd transcription-platform
source venv/bin/activate
pip install -r requirements.txt
```

#### Port 8000 Already In Use
```bash
# Kill existing process
lsof -ti:8000 | xargs kill -9

# Or use different port
python manage.py runserver 8001
```

### Frontend Issues

#### Package Compatibility Warnings
```bash
cd expo-voice-analytics-mobile-app
npm install expo@~54.0.10 expo-file-system@~19.0.15 react-native-svg@15.12.1 react-native-reanimated@~4.1.1
```

#### Node Modules Issues
```bash
cd expo-voice-analytics-mobile-app
rm -rf node_modules package-lock.json
npm install
```

#### Port 8081 Already In Use
```bash
# Kill existing Metro process
lsof -ti:8081 | xargs kill -9

# Restart
npm start
```

### Redis & Celery Issues

#### Redis Not Running
```bash
# Check if Redis is running
redis-cli ping

# Expected response: PONG
# If not responding, start Redis:
redis-server
```

#### Celery Worker Not Starting
```bash
# Check Redis connection
redis-cli ping

# Kill existing Celery processes
pkill -f "celery worker"

# Restart Celery worker
cd transcription-platform
source venv/bin/activate
celery -A backend worker -l info
```

#### Redis Connection Issues
```bash
# Check Redis status
brew services list | grep redis

# Restart Redis (macOS)
brew services restart redis

# Restart Redis (Linux)
sudo systemctl restart redis
```

#### Celery Tasks Not Processing
```bash
# Check worker status
celery -A backend inspect active

# Check task queue
celery -A backend inspect reserved

# Purge task queue (if stuck)
celery -A backend purge
```

## 🔒 Security Features Implemented

### Critical Fixes Applied
- ✅ HTTPS validation enforced in production
- ✅ JWT token validation with proper expiration checks
- ✅ Enhanced WebSocket authentication with token refresh
- ✅ Secure cross-platform token storage (encrypted for web)
- ✅ Comprehensive error boundaries preventing crashes
- ✅ Memory leak prevention with proper cleanup
- ✅ Enhanced accessibility support
- ✅ Development debugging code properly conditional

### Configuration Files Updated
- `src/constants/index.ts` - API configuration with HTTPS validation
- `src/services/apiClient.ts` - JWT validation and secure requests
- `src/services/tokenManager.ts` - Secure token storage
- `src/services/websocket.ts` - Enhanced WebSocket security
- `src/components/ErrorBoundary.tsx` - Crash prevention
- `src/services/secureStorage.ts` - Cross-platform secure storage

## 📝 Development Commands

### Backend Development
```bash
# Run with debug info
cd transcription-platform
source venv/bin/activate
python manage.py runserver --verbosity=2

# Check for issues
python manage.py check

# Run migrations
python manage.py makemigrations
python manage.py migrate
```

### Frontend Development
```bash
# Run with verbose logging
cd expo-voice-analytics-mobile-app
npm start -- --verbose

# Type checking
npm run type-check

# Clear cache and restart
npx expo start --clear
```

## 🌐 Network Access

### Local Development
- Backend: http://localhost:8000
- Frontend: http://localhost:8081

### Network Access (if needed)
```bash
# Backend - allow network access
python manage.py runserver 0.0.0.0:8000

# Frontend - will show network URL automatically
npm start
```

## 📱 Mobile Testing

### Using Expo Go App
1. Install Expo Go from App Store/Play Store
2. Start frontend with `npm start`
3. Scan QR code with Expo Go
4. App will load on your device

### Using Simulator
```bash
# iOS Simulator (Mac only)
npm start
# Press 'i' for iOS simulator

# Android Emulator
npm start
# Press 'a' for Android emulator
```

## 🔄 Regular Maintenance

### Weekly Updates
```bash
# Backend dependencies
cd transcription-platform
source venv/bin/activate
pip list --outdated

# Frontend dependencies
cd expo-voice-analytics-mobile-app
npm outdated
```

### Clean Reinstall
```bash
# Backend
cd transcription-platform
rm -rf venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Frontend
cd expo-voice-analytics-mobile-app
rm -rf node_modules package-lock.json
npm install
```

## 🚨 Emergency Procedures

### Complete Reset
```bash
# Kill all servers and workers
lsof -ti:8000,8081,6379 | xargs kill -9
pkill -f "celery worker"
pkill -f "redis-server"

# Reset backend
cd transcription-platform
rm -rf venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Reset frontend
cd expo-voice-analytics-mobile-app
rm -rf node_modules package-lock.json
npm install

# Start all services
redis-server &
cd transcription-platform && source venv/bin/activate && python manage.py runserver &
cd transcription-platform && source venv/bin/activate && celery -A backend worker -l info &
cd expo-voice-analytics-mobile-app && npm start &
```

## 📞 Support Information

### Created By
Claude Code Assistant - All critical security and performance issues resolved

### Last Updated
September 24, 2025

### Key Improvements Made
- Fixed all critical security vulnerabilities
- Enhanced error handling and crash prevention
- Improved performance and memory management
- Added comprehensive accessibility support
- Implemented proper cleanup for all resources
- Updated package compatibility issues

---

## 🏁 **Complete System Architecture**

The ChefVision Voice Analytics Platform requires **3 main services** to run:

1. **🌐 Django Backend** (`port 8000`) - API server, WebSocket transcription, Azure Speech integration
2. **⚡ Celery Worker** - Background processing for AI analysis, insights, and notifications
3. **🎯 Redis Server** (`port 6379`) - Message broker and task queue for Celery
4. **📱 React Native Frontend** (`port 8081`) - Mobile/web application interface

### 🎯 **Service Dependencies:**
- **Frontend** ↔ **Backend** (API calls, WebSocket for live transcription)
- **Backend** → **Celery Worker** (Background tasks via Redis)
- **Celery Worker** → **Redis** (Task queue and results storage)
- **Backend** → **Azure Speech Service** (Live transcription processing)

### ⚠️ **Important Notes:**
- **Celery is required** for AI analysis to work after transcription completion
- **Redis must be running** before starting Celery worker
- **All 3 backend services** should be running for full functionality
- Live transcription works without Celery, but post-processing won't work

---

**Note:** Keep this file in the root directory for easy reference. All commands assume you're starting from the `chefvision-analytics-platform/` directory.