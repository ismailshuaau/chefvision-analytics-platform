# ChefVision Voice Analytics Platform - Project State & Context

> **Last Updated:** October 2, 2025
> **Purpose:** Single source of truth for project state, architecture, and development context
> **Use Case:** Resume development across Claude Code sessions without losing context

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Current Status](#current-status)
4. [Recent Changes](#recent-changes)
5. [Active Issues & Solutions](#active-issues--solutions)
6. [Development Setup](#development-setup)
7. [Key Files & Their Purposes](#key-files--their-purposes)
8. [Common Tasks](#common-tasks)
9. [Known Bugs & Workarounds](#known-bugs--workarounds)
10. [Next Steps & Roadmap](#next-steps--roadmap)

---

## 📚 Project Overview

### What is this project?
**ChefVision Voice Analytics Platform** - An enterprise voice analytics solution that provides:
- Real-time voice transcription using Azure Speech Services
- AI-powered analysis of meeting content (summaries, action items, sentiment)
- Template-driven analysis system with role-based access
- Team collaboration and organizational analytics
- Mobile (iOS/Android) and Web support

### Tech Stack

**Frontend (Mobile/Web App):**
- React Native with Expo (~54.0.10)
- TypeScript
- Redux Toolkit for state management
- React Navigation for routing
- React Native Paper for UI components
- WebSocket for real-time transcription

**Backend (API Server):**
- Django 5.1.1 with Django REST Framework
- Python 3.13
- PostgreSQL (production) / SQLite (development)
- Celery for background tasks
- Redis for task queue and caching
- Azure Speech Services for transcription
- OpenAI/Azure OpenAI for AI analysis

**Infrastructure:**
- Expo Go for mobile testing
- Metro Bundler for development
- Git for version control (separate repos for frontend/backend)

---

## 🏗️ Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                   CHEFVISION PLATFORM ARCHITECTURE              │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐
│  Mobile/Web App  │ (React Native + Expo)
│  Port: 8081      │
└────────┬─────────┘
         │
         │ HTTP/REST API + WebSocket
         │
         ▼
┌──────────────────┐     ┌──────────────┐
│  Django Backend  │────▶│    Redis     │
│  Port: 8000      │     │  Port: 6379  │
└────────┬─────────┘     └──────────────┘
         │                       ▲
         │                       │
         ▼                       │
┌──────────────────┐            │
│  Celery Worker   │────────────┘
│  (Background)    │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────┐
│  External Services:                  │
│  - Azure Speech (Transcription)      │
│  - OpenAI/Azure (AI Analysis)        │
└──────────────────────────────────────┘
```

### Directory Structure

```
chefvision-analytics-platform/
├── expo-voice-analytics-mobile-app/    # Frontend (React Native)
│   ├── src/
│   │   ├── components/                 # Reusable UI components
│   │   ├── screens/                    # Screen components
│   │   ├── services/                   # API clients, WebSocket, etc.
│   │   ├── store/                      # Redux state management
│   │   ├── navigation/                 # Navigation config
│   │   ├── utils/                      # Utility functions
│   │   ├── constants/                  # App constants
│   │   └── types/                      # TypeScript types
│   ├── assets/                         # Images, fonts, etc.
│   ├── App.tsx                         # Root component
│   └── package.json                    # Dependencies
│
├── transcription-platform/             # Backend (Django)
│   ├── apps/
│   │   ├── analysis/                   # AI analysis logic
│   │   ├── api/                        # REST API endpoints
│   │   ├── transcriptions/             # Transcription handling
│   │   ├── operations/                 # Templates, roles, permissions
│   │   └── insights/                   # Analytics & insights
│   ├── backend/                        # Django settings
│   ├── venv/                           # Python virtual environment
│   ├── manage.py                       # Django CLI
│   └── requirements.txt                # Python dependencies
│
├── API_DOCUMENTATION.md                # API reference
├── DESIGN_GUIDELINES.md                # UI/UX guidelines
├── RUN_SERVERS.md                      # Server startup guide
└── PROJECT_STATE.md                    # THIS FILE (source of truth)
```

---

## 📊 Current Status

### Version Information
- **Frontend Version:** 1.0.0
- **Backend Version:** 1.0.0
- **Last Deploy:** October 2, 2025
- **Environment:** Development

### Feature Status

| Feature | Status | Notes |
|---------|--------|-------|
| User Authentication | ✅ Complete | JWT-based with automatic refresh |
| Real-time Transcription | ✅ Complete | Azure Speech Services |
| AI Analysis | ✅ Complete | OpenAI GPT-4o integration |
| Template System | ✅ Complete | Role-based template management |
| Bulk Analysis | ✅ Complete | Multi-session analysis |
| Team Analytics | ✅ Complete | Insights generation |
| Mobile App | ✅ Complete | iOS & Android via Expo |
| Web App | ✅ Complete | Same codebase as mobile |
| Background Tasks | ✅ Complete | Celery worker integration |
| Token Auto-Refresh | ✅ Complete | **Just implemented (Oct 2, 2025)** |

### Active Services

When running the platform, these services should be active:

1. **Redis Server** - Port 6379 (Message broker)
2. **Django Backend** - Port 8000 (API server)
3. **Celery Worker** - Background processing
4. **Metro Bundler** - Port 8081 (Frontend dev server)

**Quick Start Command:**
```bash
# See RUN_SERVERS.md for detailed instructions
```

---

## 🔄 Recent Changes

### October 2, 2025 - Automatic Token Refresh Implementation

**Problem Solved:**
Users experienced "Failed to load analysis templates" errors after 30 minutes of inactivity due to expired access tokens.

**Solution Implemented:**
Comprehensive automatic token refresh system with three layers:
1. **Proactive Refresh** - Validates and refreshes tokens before each API call
2. **Background Refresh** - Monitors token expiry every 10 minutes
3. **Reactive Refresh** - Handles 401 errors with retry logic

**Files Modified:**
- `expo-voice-analytics-mobile-app/src/services/apiClient.ts` - Enhanced with proactive validation
- `expo-voice-analytics-mobile-app/src/services/tokenRefreshService.ts` - **NEW** background service
- `expo-voice-analytics-mobile-app/src/components/AuthInitializer.tsx` - Integrated refresh service
- `expo-voice-analytics-mobile-app/src/screens/main/DashboardScreen.tsx` - Improved error handling
- `expo-voice-analytics-mobile-app/src/screens/analysis/BulkAnalysisScreen.tsx` - Fixed memory leaks

**Configuration:**
```typescript
// Token Lifetimes (backend/settings.py)
ACCESS_TOKEN_LIFETIME: 30 minutes
REFRESH_TOKEN_LIFETIME: 7 days

// Refresh Thresholds (apiClient.ts)
Proactive: < 5 minutes remaining
Background: < 10 minutes remaining
```

**Git Commits:**
- Frontend: `e19f788` - "feat: implement automatic token refresh"
- Backend: `e2be70e` - "fix: remove duplicate serializers"

### Code Cleanup (Same Session)

**Removed:**
- Duplicate API service (`src/services/api.ts`)
- Duplicate ErrorBoundary component
- Duplicate serializers in backend
- 100+ console.log statements
- Backup files (AdvancedAnalyticsScreen.backup.tsx)

**Fixed:**
- Memory leaks in polling components
- Race conditions in token refresh
- TypeScript `any` types replaced with proper interfaces
- Missing exception handling in Django views

---

## 🔧 Active Issues & Solutions

### Issue 1: Session Expiration ✅ SOLVED
**Status:** RESOLVED (Oct 2, 2025)
**Solution:** Automatic token refresh system implemented
**Details:** See "Recent Changes" section above

### Issue 2: None Currently Active
All critical issues have been resolved.

---

## 💻 Development Setup

### Prerequisites
- Node.js 18+
- Python 3.13+
- Redis server
- Expo CLI
- Git

### First Time Setup

**Backend:**
```bash
cd transcription-platform
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser  # Optional
```

**Frontend:**
```bash
cd expo-voice-analytics-mobile-app
npm install
```

### Daily Startup

**Terminal 1 - Redis:**
```bash
redis-server
```

**Terminal 2 - Django Backend:**
```bash
cd transcription-platform
source venv/bin/activate
python manage.py runserver
```

**Terminal 3 - Celery Worker:**
```bash
cd transcription-platform
source venv/bin/activate
celery -A backend worker -l info
```

**Terminal 4 - Frontend:**
```bash
cd expo-voice-analytics-mobile-app
npm start
```

### Access URLs
- **Backend API:** http://localhost:8000/api/
- **Admin Panel:** http://localhost:8000/admin/
- **Frontend Web:** http://localhost:8081
- **Mobile:** Scan QR code with Expo Go app

---

## 📁 Key Files & Their Purposes

### Critical Configuration Files

| File | Purpose | When to Edit |
|------|---------|--------------|
| `expo-voice-analytics-mobile-app/src/constants/index.ts` | API URLs, app config | Changing backend URL |
| `transcription-platform/backend/settings.py` | Django settings, JWT config | Adding features, changing tokens |
| `expo-voice-analytics-mobile-app/package.json` | Frontend dependencies | Adding npm packages |
| `transcription-platform/requirements.txt` | Backend dependencies | Adding Python packages |
| `expo-voice-analytics-mobile-app/app.json` | Expo configuration | App name, version, permissions |

### Key Service Files

| File | Purpose | Key Functions |
|------|---------|--------------|
| `src/services/apiClient.ts` | HTTP client with auth | `get()`, `post()`, token refresh |
| `src/services/tokenRefreshService.ts` | Background token refresh | `start()`, `stop()` |
| `src/services/tokenManager.ts` | Token storage | `getAccessToken()`, `setTokens()` |
| `src/services/websocket.ts` | WebSocket connection | Real-time transcription |
| `src/services/transcriptionService.ts` | Transcription API | Session management |
| `src/services/analysisService.ts` | Analysis API | AI analysis, polling |

### Important Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `AuthInitializer` | `src/components/AuthInitializer.tsx` | Auth state initialization |
| `ErrorBoundary` | `src/components/common/ErrorBoundary.tsx` | Error handling |
| `DashboardScreen` | `src/screens/main/DashboardScreen.tsx` | Main app screen |
| `LiveTranscriptionScreen` | `src/screens/transcription/LiveTranscriptionScreen.tsx` | Voice recording |

---

## 🔨 Common Tasks

### Adding a New Feature

1. **Backend:**
   ```bash
   cd transcription-platform
   # Create new app if needed
   python manage.py startapp feature_name
   # Add to INSTALLED_APPS in settings.py
   # Create models, views, serializers
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Frontend:**
   ```bash
   cd expo-voice-analytics-mobile-app
   # Add service method in src/services/
   # Create screen component in src/screens/
   # Add navigation route in src/navigation/
   ```

### Debugging Authentication Issues

**Check token status:**
```javascript
// In browser console or React Native debugger
import { isTokenExpired, getTokenTimeToExpiry } from './src/utils';
const token = await tokenManager.getAccessToken();
console.log('Expired:', isTokenExpired(token));
console.log('Time left:', getTokenTimeToExpiry(token), 'seconds');
```

**Force token refresh:**
```javascript
import tokenRefreshService from './src/services/tokenRefreshService';
await tokenRefreshService.forceRefresh();
```

### Running Database Migrations

```bash
cd transcription-platform
source venv/bin/activate
python manage.py makemigrations
python manage.py migrate
```

### Clearing Redis Cache

```bash
redis-cli FLUSHALL
```

### Resetting Database (Development Only)

```bash
cd transcription-platform
rm db.sqlite3
python manage.py migrate
python manage.py createsuperuser
```

---

## 🐛 Known Bugs & Workarounds

### Bug: None Currently Active
All known bugs have been fixed as of October 2, 2025.

### Previous Bugs (Fixed)

1. **Session Expiration After 30 Minutes** ✅
   - **Fixed:** Oct 2, 2025
   - **Solution:** Automatic token refresh system

2. **Memory Leaks in Polling Components** ✅
   - **Fixed:** Oct 2, 2025
   - **Solution:** Added cleanup functions in useEffect

3. **Race Conditions in Token Refresh** ✅
   - **Fixed:** Oct 2, 2025
   - **Solution:** Promise-based debouncing

---

## 🗺️ Next Steps & Roadmap

### Immediate Priorities (This Week)

- [ ] Test automatic token refresh in production
- [ ] Monitor error rates for session-related issues
- [ ] Update deployment documentation

### Short Term (This Month)

- [ ] Add user analytics dashboard
- [ ] Implement template sharing between teams
- [ ] Add export functionality for analysis results
- [ ] Improve mobile app performance

### Long Term (Next Quarter)

- [ ] Multi-language support for transcription
- [ ] Advanced analytics with charts
- [ ] Offline mode for mobile app
- [ ] Integration with calendar apps

---

## 🔐 Security & Compliance

### Authentication Flow

```
Login → JWT Access Token (30 min) + Refresh Token (7 days)
  ↓
API Request → Validate Token → Refresh if < 5 min remaining
  ↓
Token Expired → Auto-refresh → Retry Request
  ↓
Refresh Failed → Redirect to Login
```

### Token Configuration

**Current Settings (Optimal for Business Apps):**
- Access Token: 30 minutes
- Refresh Token: 7 days
- Proactive Refresh: < 5 minutes
- Background Check: Every 10 minutes

**Security Features:**
- HTTPS required in production
- Token rotation on refresh
- Secure storage (SecureStore on mobile, encrypted localStorage on web)
- Automatic token cleanup on logout

---

## 📞 Support & Resources

### Documentation Files
- `API_DOCUMENTATION.md` - Complete API reference
- `DESIGN_GUIDELINES.md` - UI/UX design standards
- `RUN_SERVERS.md` - Server setup and troubleshooting
- `PROJECT_STATE.md` - This file (project context)

### Git Repositories
- **Frontend:** `github.com:Chef-Vision/expo-voice-analytics-mobile-app.git`
- **Backend:** `github.com:Chef-Vision/transcription-platform.git`

### External Services
- **Azure Speech Services** - Real-time transcription
- **OpenAI API** - AI-powered analysis
- **Expo** - Mobile app infrastructure

---

## 🧠 Context for Claude Code

### When Starting a New Session

**Quick Context Load:**
1. Read this file (PROJECT_STATE.md)
2. Check `git log --oneline -5` in both repos for recent changes
3. Review `RUN_SERVERS.md` for current server status
4. Check `package.json` and `requirements.txt` for dependencies

**Common Questions to Ask:**
- "What was I working on last?" → Check "Recent Changes" section
- "How do I run the servers?" → Check "Development Setup" section
- "What files are most important?" → Check "Key Files" section
- "Are there any known issues?" → Check "Active Issues" section

**Important Notes:**
- Always run all 4 services (Redis, Django, Celery, Metro) for full functionality
- Token refresh is automatic - no manual intervention needed
- Frontend and backend are in separate git repos
- This file is the single source of truth - update it when making major changes

---

## 📝 Change Log

### October 2, 2025
- ✅ Implemented automatic token refresh system
- ✅ Fixed memory leaks in polling components
- ✅ Removed duplicate code and files
- ✅ Added exception handling in Django views
- ✅ Created this PROJECT_STATE.md file
- 📝 Updated both git repositories

### September 30, 2025
- Added modern template system
- Improved template builder UI

### September 26, 2025
- Enhanced analytics dashboard
- Added team collaboration features

---

## 🎯 Quick Reference Commands

```bash
# Start all services (from project root)
# Terminal 1: redis-server
# Terminal 2: cd transcription-platform && source venv/bin/activate && python manage.py runserver
# Terminal 3: cd transcription-platform && source venv/bin/activate && celery -A backend worker -l info
# Terminal 4: cd expo-voice-analytics-mobile-app && npm start

# Git status both repos
cd expo-voice-analytics-mobile-app && git status && cd ../transcription-platform && git status

# Pull latest changes
cd expo-voice-analytics-mobile-app && git pull && cd ../transcription-platform && git pull

# Install dependencies
cd expo-voice-analytics-mobile-app && npm install && cd ../transcription-platform && source venv/bin/activate && pip install -r requirements.txt

# Run migrations
cd transcription-platform && source venv/bin/activate && python manage.py migrate

# Check Celery tasks
cd transcription-platform && source venv/bin/activate && celery -A backend inspect registered
```

---

**End of PROJECT_STATE.md**
*This file should be updated whenever significant changes are made to the project.*
*Last updated by: Claude Code (October 2, 2025)*
