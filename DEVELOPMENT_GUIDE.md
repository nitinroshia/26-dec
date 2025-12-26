# Pellacia Press - Phase 06 Distribution Backend Development Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Quick Start for Claude](#quick-start-for-claude)
3. [Architecture Overview](#architecture-overview)
4. [Implementation Phases](#implementation-phases)
5. [File Structure](#file-structure)
6. [Dependencies](#dependencies)
7. [Platform Integrations](#platform-integrations)
8. [Database Schema](#database-schema)
9. [CLI Interface Specification](#cli-interface-specification)
10. [Error Handling & Testing](#error-handling--testing)
11. [Integration Points](#integration-points)
12. [Implementation Checklist](#implementation-checklist)

---

## Project Overview

Pellacia Press is an AI-powered video automation platform that transforms raw articles into published video content across multiple social media platforms. Phase 06 focuses on the distribution layer - automated posting, scheduling, and analytics tracking for 6 social media platforms.

### Core Objectives
- **Automated Content Distribution**: Upload videos to YouTube, Instagram, TikTok, Twitter/X, LinkedIn, and Facebook
- **Content Scheduling**: Queue posts for optimal publication times across platforms
- **Analytics Tracking**: Collect and aggregate performance metrics with rate limit controls
- **Terminal-Based Operations**: CLI interface for reliable, scriptable operations
- **Phase-03 Integration**: REST API for web dashboard integration

### Key Requirements
- Support 5-6 posts per day per platform initially (scalable to higher volumes)
- Terminal-based Python application with async processing
- Comprehensive error handling and retry logic
- Controlled analytics fetching to avoid rate limit violations
- Secure credential management
- PostgreSQL database for job queue and analytics storage

### Business Constraints
- **Free VPN Usage**: Must work with free VPNs (Netherlands, Japan, Romania locations) for TikTok access
- **Browser Automation Fallback**: Selenium/Playwright as primary fallback when APIs fail
- **Breaking News Priority**: System must interrupt normal operations for urgent content
- **Conservative Posting**: 30-45 minute minimum intervals to avoid platform bans
- **Instant Alerts**: Telegram notifications for immediate failure awareness

---

## Quick Start for Claude

### Immediate Action Items
1. **Set up project structure** using the File Structure section below
2. **Install dependencies** from the Dependencies section
3. **Create database schema** from Database Schema section
4. **Implement core CLI framework** (`main.py`, `src/cli/commands.py`)
5. **Start with TikTok VPN integration** (highest complexity)

### Key Implementation Decisions Already Made
- **Python 3.10+** with asyncio
- **PostgreSQL** for data persistence
- **Free VPNs with redundancy** for TikTok (don't use paid proxies)
- **Browser automation as primary fallback** (not just last resort)
- **Telegram alerts** for instant notifications
- **Breaking news interruption** of normal posting queue

### Development Approach
- **Start with infrastructure components** (VPN, browser automation, alerts)
- **Build platform integrations** one by one (TikTok first due to complexity)
- **Add resilience features** (retry logic, fallbacks) as you build
- **Test early and often** - each platform integration needs immediate testing

### Success Criteria
- **TikTok uploads work** via VPN + browser automation fallback
- **All 6 platforms can be posted to** simultaneously
- **Breaking news interrupts** normal queue operations
- **Telegram alerts fire** immediately on failures
- **30-minute spacing** enforced between posts

## Architecture Overview

### System Components
1. **CLI Interface**: Terminal commands for upload, scheduling, and analytics
2. **Platform Modules**: Platform-specific upload and analytics logic
3. **Job Scheduler**: Queue management for timed posts
4. **Analytics Collector**: Rate-controlled metric fetching from platforms
5. **REST API**: Endpoints for phase-03 dashboard integration
6. **Database Layer**: PostgreSQL for persistence and analytics storage

### Data Flow
```
Phase 05 Output → Distribution CLI/API → Platform APIs → Post Confirmation
                                               ↓
Analytics Collector → Platform APIs → Database → REST API → Phase 03 Dashboard
```

### Technology Stack
- **Language**: Python 3.10+
- **Async Framework**: asyncio + aiohttp
- **Database**: PostgreSQL 15+ with asyncpg
- **Job Scheduling**: APScheduler
- **CLI Framework**: Click
- **API Framework**: FastAPI (for REST endpoints)
- **Platform SDKs**: Platform-specific libraries and REST clients

## Implementation Phases

### 🚀 Phase 1: Infrastructure Foundation (Week 1-2) - CRITICAL FIRST
**Goal**: Build the resilient infrastructure layer that handles VPNs, browser automation, and monitoring

**Deliverables**:
- ✅ Free VPN manager with 6 connection redundancy (NL/JP/RO)
- ✅ Browser automation framework (Playwright) with session persistence
- ✅ Telegram alert system for instant notifications
- ✅ Health monitoring dashboard
- ✅ Basic CLI framework and configuration

**Key Files to Create**:
- `src/infrastructure/free_vpn_manager.py` - VPN connection logic
- `src/fallback/browser_uploader.py` - Browser automation
- `src/fallback/session_manager.py` - Browser session persistence
- `src/monitoring/alert_system.py` - Telegram alerts
- `src/monitoring/health_dashboard.py` - Terminal monitoring UI
- `main.py` - Basic CLI entry point
- `src/core/config.py` - Configuration management

**Testing Commands**:
```bash
# Test VPN connectivity
pellacia-dist test vpn

# Test browser automation setup
pellacia-dist test browser --platform tiktok

# Test alerts
pellacia-dist test alerts
```

### 📤 Phase 2: Core Upload Engine (Week 2-4) - START HERE
**Goal**: Build the upload orchestration system with cascading fallbacks

**Deliverables**:
- ✅ Resilient uploader with 5-layer fallback strategy
- ✅ Platform authentication for all 6 platforms
- ✅ TikTok VPN integration with browser fallback
- ✅ Multi-platform concurrent uploads
- ✅ Comprehensive error handling and retry logic

**Key Files to Create**:
- `src/core/resilient_uploader.py` - Main upload orchestration
- `src/core/retry.py` - Error handling logic
- `src/platforms/tiktok.py` - TikTok with VPN support
- `src/platforms/youtube.py` - YouTube upload
- `src/platforms/instagram.py` - Instagram upload
- `src/platforms/twitter.py` - Twitter upload
- `src/platforms/linkedin.py` - LinkedIn upload
- `src/platforms/facebook.py` - Facebook upload

**Testing Commands**:
```bash
# Test single platform upload
pellacia-dist upload --platform tiktok --file test.mp4 --test

# Test multi-platform upload
pellacia-dist upload --platforms youtube,tiktok --file test.mp4 --test

# Test VPN fallback for TikTok
pellacia-dist upload --platform tiktok --file test.mp4 --force-vpn-fallback
```

### ⏰ Phase 3: Scheduling & Priority System (Week 4-5)
**Goal**: Add intelligent queuing with breaking news support

**Deliverables**:
- ✅ Priority queue with breaking news interruption
- ✅ Posting throttler (30-45 min intervals)
- ✅ APScheduler integration for timed posts
- ✅ Queue persistence and recovery

**Key Files to Create**:
- `src/scheduling/priority_queue.py` - Priority management
- `src/scheduling/posting_throttler.py` - Rate limiting
- `src/scheduling/scheduler.py` - APScheduler integration
- Database migrations for queue tables

**Testing Commands**:
```bash
# Test breaking news priority
pellacia-dist upload --file breaking.mp4 --platforms all --priority breaking

# Test normal scheduling
pellacia-dist schedule --platforms all --file news.mp4 --at "2024-12-27 14:00"

# View queue status
pellacia-dist queue status
```

### 📊 Phase 4: Analytics System (Week 5-6)
**Goal**: Add controlled analytics collection with rate limit management

**Deliverables**:
- ✅ Rate-limited analytics fetching for all platforms
- ✅ Unified analytics storage and aggregation
- ✅ Analytics CLI commands
- ✅ Background analytics collection jobs

**Key Files to Create**:
- `src/analytics/collector.py` - Platform analytics fetching
- `src/analytics/aggregator.py` - Data processing and storage
- `src/analytics/rate_limiter.py` - API rate limit management
- Database tables for analytics data

**Testing Commands**:
```bash
# Fetch analytics for specific post
pellacia-dist analytics fetch --post-id <id> --platform youtube

# Run daily analytics collection
pellacia-dist analytics collect-daily
```

### 🌐 Phase 5: Phase-03 Integration (Week 6-7)
**Goal**: Connect to web dashboard with REST API

**Deliverables**:
- ✅ FastAPI REST endpoints
- ✅ Pydantic data models
- ✅ API documentation
- ✅ Webhook support for real-time updates

**Key Files to Create**:
- `src/api/endpoints.py` - REST API routes
- `src/api/models.py` - Request/response models
- `api.py` - FastAPI application
- API documentation and integration tests

### 🛠️ Phase 6: Manual Intervention System (Week 7-8)
**Goal**: Handle cases where all automation fails

**Deliverables**:
- ✅ Manual intervention queue
- ✅ Export tools for manual uploading
- ✅ Resolution tracking
- ✅ Clear instructions generation

**Key Files to Create**:
- `src/fallback/manual_queue.py` - Manual intervention management
- CLI commands for manual operations

**Testing Commands**:
```bash
# View manual intervention queue
pellacia-dist manual list

# Export manual upload instructions
pellacia-dist manual export --id <id>

# Mark as manually resolved
pellacia-dist manual resolve --id <id> --post-url <url>
```

## File Structure

```
phase-06-distribution/
├── config/
│   ├── platforms.json          # Platform credentials and settings
│   └── database.json           # Database connection config
├── src/
│   ├── platforms/              # Platform-specific modules
│   │   ├── youtube.py
│   │   ├── instagram.py
│   │   ├── tiktok.py
│   │   ├── twitter.py
│   │   ├── linkedin.py
│   │   └── facebook.py
│   ├── scheduling/
│   │   ├── queue.py            # Job queue management
│   │   └── scheduler.py        # APScheduler integration
│   ├── analytics/
│   │   ├── collector.py        # Fetch metrics from platforms
│   │   ├── aggregator.py       # Process and unify metrics
│   │   └── storage.py          # Database operations
│   ├── core/
│   │   ├── uploader.py         # Main upload orchestration
│   │   ├── retry.py            # Error handling & retry logic
│   │   ├── validator.py        # Input validation
│   │   └── config.py           # Configuration management
│   ├── api/
│   │   ├── endpoints.py        # FastAPI routes
│   │   └── models.py           # Pydantic models
│   └── cli/
│       └── commands.py         # Click CLI commands
├── tests/
│   ├── test_platforms/         # Platform-specific tests
│   ├── test_scheduling.py
│   ├── test_analytics.py
│   └── test_cli.py
├── migrations/
│   └── 001_initial_schema.sql  # Database schema
├── docs/
│   ├── DEVELOPMENT_GUIDE.md    # This document
│   ├── API_REFERENCE.md        # Platform API details
│   └── TESTING_GUIDE.md
├── requirements.txt            # Python dependencies
├── pyproject.toml              # Project configuration
├── main.py                     # CLI entry point
├── api.py                      # FastAPI application
└── README.md                   # Project documentation
```

## Dependencies

### Core Dependencies
```
# Core async and web
aiohttp==3.9.1
fastapi==0.104.1
uvicorn==0.24.0
pydantic==2.5.0

# Database
asyncpg==0.29.0
sqlalchemy==2.0.23
alembic==1.13.0

# CLI and scheduling
click==8.1.7
apscheduler==3.10.4

# Platform SDKs
google-api-python-client==2.110.0
google-auth-oauthlib==1.2.0
facebook-sdk==4.0.0
tweepy==4.14.0
linkedin-api==2.1.0
requests==2.31.0

# Utilities
python-dotenv==1.0.0
pydantic-settings==2.1.0
structlog==23.2.0
tenacity==8.2.3
```

### Development Dependencies
```
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
black==23.11.0
isort==5.12.0
mypy==1.7.1
ruff==0.1.7
```

## Platform Integrations

### YouTube
**API**: YouTube Data API v3
**Authentication**: OAuth 2.0
**Key Endpoints**:
- Upload: `POST /upload/youtube/v3/videos`
- Analytics: YouTube Analytics API
**Rate Limits**: 10,000 requests/day, 1 upload/minute
**Metrics**: views, watch_time, subscribers_gained, impressions

### Instagram
**API**: Instagram Graph API (via Meta)
**Authentication**: OAuth 2.0 with Facebook App
**Key Endpoints**:
- Upload: `POST /{ig-user-id}/media`
- Analytics: `GET /{ig-user-id}/insights`
**Rate Limits**: 200/hour per user
**Metrics**: impressions, reach, engagement, follower_demographics

### TikTok
**API**: TikTok for Developers API
**Authentication**: OAuth 2.0
**Key Endpoints**:
- Upload: `POST /video/upload/`
- Analytics: `GET /video/analytics/`
**Rate Limits**: 500/hour
**Metrics**: views, likes, comments, shares, audience_demographics

### Twitter/X
**API**: Twitter API v2
**Authentication**: OAuth 2.0
**Key Endpoints**:
- Upload: `POST /2/tweets` + media upload
- Analytics: `GET /2/tweets/{id}` (with public_metrics)
**Rate Limits**: 300/hour
**Metrics**: impressions, engagements, retweets, likes

### LinkedIn
**API**: LinkedIn Marketing API
**Authentication**: OAuth 2.0
**Key Endpoints**:
- Upload: `POST /rest/videos`
- Analytics: `GET /rest/organizationalEntityShareStatistics`
**Rate Limits**: 100/hour
**Metrics**: impressions, clicks, engagement, professional_demographics

### Facebook
**API**: Facebook Graph API
**Authentication**: OAuth 2.0
**Key Endpoints**:
- Upload: `POST /{page-id}/videos`
- Analytics: `GET /{page-id}/insights`
**Rate Limits**: 200/hour
**Metrics**: reach, engagement, impressions, video_views

## Database Schema

### Core Tables

```sql
-- Scheduled posts and upload jobs
CREATE TABLE scheduled_posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    platforms TEXT[] NOT NULL,
    video_path TEXT NOT NULL,
    title TEXT,
    description TEXT,
    hashtags TEXT[],
    scheduled_at TIMESTAMP WITH TIME ZONE,
    status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    error_message TEXT,
    post_ids JSONB  -- Platform-specific post IDs after upload
);

-- Analytics data storage
CREATE TABLE post_analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id TEXT NOT NULL,
    platform TEXT NOT NULL,
    metrics JSONB NOT NULL,
    fetched_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (post_id, platform, fetched_at::date)
);

-- Platform credentials (encrypted)
CREATE TABLE platform_credentials (
    platform TEXT PRIMARY KEY,
    credentials JSONB NOT NULL,  -- Encrypted OAuth tokens
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Upload history and audit log
CREATE TABLE upload_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    scheduled_post_id UUID REFERENCES scheduled_posts(id),
    platform TEXT NOT NULL,
    status TEXT NOT NULL,
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    post_id TEXT,
    error_message TEXT,
    retry_count INTEGER DEFAULT 0
);

-- Indexes for performance
CREATE INDEX idx_scheduled_posts_status ON scheduled_posts(status);
CREATE INDEX idx_scheduled_posts_scheduled_at ON scheduled_posts(scheduled_at);
CREATE INDEX idx_post_analytics_post_id ON post_analytics(post_id);
CREATE INDEX idx_post_analytics_platform ON post_analytics(platform);
CREATE INDEX idx_upload_history_scheduled_post ON upload_history(scheduled_post_id);
```

## CLI Interface Specification

### Core Commands

**Upload Commands**:
```bash
# Upload to single platform immediately
pellacia-dist upload --platform youtube --file video.mp4 --title "Video Title"

# Upload to multiple platforms
pellacia-dist upload --platforms youtube,instagram,tiktok --file video.mp4

# Upload with metadata
pellacia-dist upload --platforms all --file video.mp4 \
  --title "My Video" \
  --description "Description here" \
  --hashtags "#video #content"
```

**Scheduling Commands**:
```bash
# Schedule post for later
pellacia-dist schedule --platforms youtube,instagram \
  --file video.mp4 \
  --at "2024-12-27 14:00:00" \
  --timezone "America/New_York"

# View scheduled posts
pellacia-dist schedule list

# Cancel scheduled post
pellacia-dist schedule cancel --id <uuid>
```

**Analytics Commands**:
```bash
# Fetch analytics for specific post
pellacia-dist analytics fetch --post-id <id> --platform youtube

# Fetch analytics for date range
pellacia-dist analytics fetch --platform youtube --days 7

# View analytics summary
pellacia-dist analytics summary --platform youtube --start-date 2024-01-01
```

**Queue Management**:
```bash
# View queue status
pellacia-dist queue status

# Clear failed jobs
pellacia-dist queue clear --status failed

# Retry failed uploads
pellacia-dist queue retry --id <uuid>
```

## Error Handling & Retry Logic

### Retry Strategy
- **Exponential Backoff**: 1s, 2s, 4s, 8s, 16s delays between retries
- **Maximum Retries**: 3 attempts per upload
- **Platform-Specific Handling**: Different retry logic for different error types
- **Rate Limit Handling**: Automatic pause when limits approached

### Error Types
- **Network Errors**: Retry with backoff
- **Authentication Errors**: Refresh tokens or request re-auth
- **Rate Limit Errors**: Wait until reset time
- **Validation Errors**: Fail immediately (user input issues)
- **Platform Errors**: Retry or escalate based on error code

### Monitoring & Alerting
- Structured logging with context
- Error aggregation and reporting
- Health check endpoints
- Alert system for critical failures

## Testing Strategy

### Unit Tests
- Platform-specific upload logic
- Authentication flows
- Data validation
- Error handling scenarios

### Integration Tests
- End-to-end upload workflows
- Database operations
- API endpoint testing
- CLI command testing

### Test Data
- Mock platform APIs for testing
- Test video files
- Pre-configured test credentials

### CI/CD Pipeline
- Automated testing on commits
- Code quality checks (linting, type checking)
- Security scanning
- Deployment automation

## Integration Points

### Phase 05 (Production)
**Input**: Video files from `phase-05-production/output/`
**Format**: MP4 files with metadata JSON
**Trigger**: File system monitoring or API calls

### Phase 03 (Editorial)
**Output**: REST API endpoints for dashboard
**Features**:
- View scheduled posts
- Monitor upload status
- Display analytics data
- Manual upload triggers

### Database Integration
**Shared Database**: PostgreSQL instance with other phases
**Schema Isolation**: Dedicated tables with clear naming
**Migration Safety**: Backward-compatible schema changes

## Security Considerations

### Credential Management
- Encrypted storage of OAuth tokens
- Environment-specific configurations
- Token refresh automation
- Access logging and auditing

### API Security
- Rate limiting on REST endpoints
- Input validation and sanitization
- Secure error messages (no credential leaks)
- HTTPS-only communication

### Platform Security
- OAuth best practices
- Scope minimization
- Regular credential rotation
- Compliance with platform policies

## Deployment & Operations

### Environment Setup
- Environment variable configuration
- Database migration automation
- Health check endpoints

### Monitoring
- Application metrics (Prometheus)
- Error tracking (Sentry)
- Log aggregation (ELK stack)
- Performance monitoring

### Backup & Recovery
- Database backups
- Credential backup procedures
- Disaster recovery procedures
- Rollback strategies

## Success Metrics

### Technical Metrics
- Upload success rate (>99%)
- Average upload time (<30 seconds per platform)
- Analytics collection coverage (>95% of posts)
- System uptime (>99.9%)

### Business Metrics
- Posts published per day (target: 5-6 per platform)
- Time to publish after video production (<5 minutes)
- Analytics data freshness (<24 hours after available)
- User satisfaction with dashboard integration

---

---

## Implementation Checklist

### ✅ Pre-Implementation Setup
- [ ] Create project directory structure (see File Structure section)
- [ ] Set up Python 3.10+ virtual environment
- [ ] Install all dependencies from requirements.txt
- [ ] Set up PostgreSQL database and create schema
- [ ] Configure platform credentials in config/platforms.json
- [ ] Set up Telegram bot for alerts (get bot token and chat ID)

### ✅ Phase 1: Infrastructure Foundation
- [ ] Implement `src/infrastructure/free_vpn_manager.py` with 6 VPN connections
- [ ] Create `src/fallback/browser_uploader.py` with Playwright integration
- [ ] Build `src/fallback/session_manager.py` for browser session persistence
- [ ] Implement `src/monitoring/alert_system.py` with Telegram alerts
- [ ] Create `src/monitoring/health_dashboard.py` terminal UI
- [ ] Set up basic CLI framework in `main.py`
- [ ] Test VPN connectivity: `pellacia-dist test vpn`
- [ ] Test browser automation: `pellacia-dist test browser --platform tiktok`
- [ ] Verify Telegram alerts work

### ✅ Phase 2: Core Upload Engine
- [ ] Build `src/core/resilient_uploader.py` with 5-layer fallback strategy
- [ ] Implement TikTok upload with VPN + browser fallback
- [ ] Create platform modules for all 6 platforms
- [ ] Add comprehensive error handling and retry logic
- [ ] Test single platform uploads
- [ ] Test multi-platform concurrent uploads
- [ ] Verify VPN fallback works for TikTok

### ✅ Phase 3: Scheduling & Priority System
- [ ] Implement `src/scheduling/priority_queue.py` with breaking news support
- [ ] Create `src/scheduling/posting_throttler.py` for 30-45 min intervals
- [ ] Set up APScheduler integration
- [ ] Test breaking news interruption
- [ ] Test normal scheduling queue
- [ ] Verify throttling prevents overposting

### ✅ Phase 4: Analytics System
- [ ] Build `src/analytics/collector.py` for platform metrics
- [ ] Create `src/analytics/rate_limiter.py` for API limits
- [ ] Implement analytics storage and aggregation
- [ ] Add analytics CLI commands
- [ ] Test controlled analytics fetching
- [ ] Verify rate limit compliance

### ✅ Phase 5: Phase-03 Integration
- [ ] Create FastAPI application with endpoints
- [ ] Implement Pydantic models for requests/responses
- [ ] Add webhook support for real-time updates
- [ ] Generate API documentation
- [ ] Test integration with Phase-03 dashboard

### ✅ Phase 6: Manual Intervention System
- [ ] Build manual intervention queue
- [ ] Create export tools for manual uploads
- [ ] Add resolution tracking
- [ ] Generate clear manual instructions
- [ ] Test full failure recovery workflow

### ✅ Production Readiness
- [ ] Comprehensive test suite (unit + integration)
- [ ] Error monitoring and logging
- [ ] Performance optimization
- [ ] Security audit (credential handling)
- [ ] Documentation completion
- [ ] Deployment scripts

### 📋 Key Testing Scenarios
- [ ] **VPN Failure**: Disconnect VPN mid-upload → verify browser fallback
- [ ] **API Rate Limit**: Hit limits → verify graceful handling
- [ ] **Breaking News**: Test interruption of normal queue
- [ ] **Platform Down**: Simulate platform outage → verify fallback activation
- [ ] **Concurrent Uploads**: Test 6-platform simultaneous posting
- [ ] **Manual Intervention**: Force all fallbacks to fail → verify manual queue

### 🚨 Critical Success Metrics
- [ ] TikTok uploads work reliably via VPN + browser automation
- [ ] All 6 platforms can be posted to simultaneously
- [ ] Breaking news posts interrupt normal operations
- [ ] Telegram alerts fire within 30 seconds of failures
- [ ] 30-minute minimum spacing enforced between posts
- [ ] Zero data loss during system restarts
- [ ] All posts eventually published (manual fallback works)

---

## Claude Implementation Notes

### 🎯 Development Priorities
1. **Start with infrastructure** (VPN, browser, alerts) - these are used by everything else
2. **TikTok first** - most complex due to VPN requirements
3. **Test early, test often** - each component needs immediate validation
4. **Build resilience in** - expect failures and design around them

### 🛠️ Key Technical Decisions
- **Free VPNs are unreliable** - design for frequent failures with redundancy
- **Browser automation is primary fallback** - not a "last resort"
- **Telegram alerts are critical** - instant mobile notifications for failures
- **Conservative posting prevents bans** - most valuable asset is platform accounts

### 📞 When to Ask for Clarification
- Platform API behavior questions
- Specific error handling scenarios
- Database schema optimization
- Performance bottlenecks

### 🎉 Success Definition
System can reliably post to all 6 platforms, handle failures gracefully, prioritize breaking news, and keep user informed via instant alerts - all while staying within free VPN constraints.

This development guide provides the complete specification for implementing the Phase 06 Distribution Backend. Use this document to guide the development process, ensuring all requirements and technical details are properly addressed.
