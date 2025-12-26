# Pellacia Press - Phase 06 Distribution Backend Development Guide

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

### Phase 1: Basic Upload Infrastructure
**Goal**: Enable immediate video uploading to all 6 social media platforms
**Deliverables**:
- Platform authentication setup (OAuth flows) for all platforms
- Complete platform upload modules for YouTube, Instagram, TikTok, Twitter/X, LinkedIn, Facebook
- Basic CLI commands for single and multi-platform uploads
- Error handling with retry logic
- Rate limit monitoring and management

**Key Files**:
- `src/platforms/youtube.py` - YouTube upload logic
- `src/platforms/instagram.py` - Instagram upload logic
- `src/platforms/tiktok.py` - TikTok upload logic
- `src/platforms/twitter.py` - Twitter/X upload logic
- `src/platforms/linkedin.py` - LinkedIn upload logic
- `src/platforms/facebook.py` - Facebook upload logic
- `src/core/uploader.py` - Main upload orchestration
- `src/core/retry.py` - Error handling and retry logic
- `main.py` - CLI entry point

### Phase 2: Multi-Platform Upload & Scheduling
**Goal**: Support simultaneous posting and future scheduling across all platforms
**Deliverables**:
- Batch upload functionality to all 6 platforms concurrently
- Platform selection and validation
- Upload status tracking and reporting
- Job queue system with persistence for scheduled posts
- Time zone support and conflict resolution
- CLI commands for scheduling operations

**Key Features**:
- Command: `upload --platforms all --file video.mp4` (supports all 6 platforms)
- Support for 5-6 posts/day per platform
- Upload confirmation and status reporting
- Queue management (view, cancel, reschedule scheduled posts)

**Key Files**:
- `src/scheduling/queue.py` - Job queue management
- `src/scheduling/scheduler.py` - APScheduler integration
- Database tables for scheduled posts

### Phase 3: Analytics System
**Goal**: Implement comprehensive analytics tracking for all platforms with rate limit controls
**Deliverables**:
- Platform-specific analytics collection for all 6 platforms
- Rate limit controls and monitoring
- Unified analytics storage schema
- CLI commands for analytics operations
- Automated daily analytics collection

**Key Considerations**:
- 24-72 hour data delay after posting
- Daily batch collection (not real-time)
- Platform-specific rate limits and batch sizes
- JSON storage for flexible metrics schema
- Support for all platform-specific metrics (views, engagement, demographics, etc.)

### Phase 4: Phase-03 Integration & API
**Goal**: Connect distribution backend to web dashboard with full API support
**Deliverables**:
- REST API endpoints for all operations
- React dashboard component integration
- Analytics visualization and reporting
- Real-time status updates
- Complete API documentation

**Key Files**:
- `src/api/endpoints.py` - FastAPI routes
- `src/api/models.py` - Pydantic data models
- Frontend components in phase-03

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

This development guide provides the complete specification for implementing the Phase 06 Distribution Backend. Use this document to guide the development process, ensuring all requirements and technical details are properly addressed.