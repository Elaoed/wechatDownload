# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
微信公众号文章下载工具 - WeChat article download tool built with Electron + TypeScript + Vue3.

**Core Architecture**:
- **Main Process** (`src/main/`): Electron main process with proxy server, worker thread management, and system integration
- **Renderer Process** (`src/renderer/`): Vue 3 frontend with TypeScript
- **Worker Threads**: Background processing for article downloads and EPUB generation
- **Proxy System**: AnyProxy-based HTTP proxy for intercepting WeChat API calls

## Essential Commands
- **Development**: `npm run dev` 
- **Type checking**: `npm run typecheck` (runs both node and web checks)
- **Linting**: `npm run lint`
- **Code formatting**: `npm run format`
- **Build**: `npm run build` (includes typecheck)
- **Platform builds**: `npm run build:win`, `npm run build:mac`, `npm run build:linux`

## Core Architecture Components

### Main Process (`src/main/index.ts`)
- Electron app initialization and window management
- AnyProxy server setup for intercepting WeChat requests
- IPC handlers for renderer communication
- CA certificate management and installation
- Comprehensive proxy cleanup system for all platforms
- Auto-updater integration

### Worker Threads
- **Main Worker** (`src/main/worker.ts`): Handles article downloading, HTML/Markdown conversion, MySQL integration
- **EPUB Worker** (`src/main/epubWorker.ts`): Generates EPUB files from downloaded HTML articles
- **Service Layer** (`src/main/service.ts`): Core business logic classes (GzhInfo, ArticleInfo, etc.)

### Proxy & Parameter Extraction
The app uses AnyProxy to intercept WeChat API calls and extract:
- `__biz`: WeChat account ID  
- `uin`: WeChat user ID
- `key`: Authentication parameter

Three download modes:
1. **Single article**: Direct URL download (no proxy needed)
2. **Batch download**: Proxy intercepts parameters, downloads entire account history
3. **Monitor download**: Proxy collects URLs from multiple opened articles

### Frontend (`src/renderer/src/`)
- **Home.vue**: Main interface for downloads and monitoring
- **Setting.vue**: Configuration interface for all download options
- **EpubCreator.vue**: EPUB generation interface

## Key Configuration
- **Proxy port**: 8001 (hardcoded in createProxy function)
- **Logging**: Uses electron-log, outputs to app data directory
- **Storage**: electron-store for settings persistence
- **CA certificates**: Auto-generated in `~/.anyproxy/certificates/`

## Development Notes
- Uses electron-vite for build system
- Supports MySQL integration for article storage
- Comprehensive proxy cleanup on app exit (critical for system stability)
- Platform-specific CA certificate handling (Windows auto-install)
- Worker thread architecture prevents UI blocking during downloads

## Debugging
- **Proxy issues**: Check port 8001 availability with `lsof -i :8001`
- **CA certificate**: Settings → Install Certificate (Windows), manual install (macOS/Linux)
- **Logs**: Settings → Open Log Directory
- **Worker errors**: Check electron-log files for detailed worker thread errors

## Critical Implementation Details
- Proxy cleanup is essential - implements multiple cleanup strategies for robustness
- WeChat article URLs must be opened fresh (not from browser cache)
- MySQL schema defined in `/doc/mysql.sql`
- Filter rules support title/author keyword filtering with include/exclude patterns

## Development Guidelines
- **Download Mode Isolation**: When modifying code for any download mode, ALWAYS ensure other modes remain unaffected. The three modes (Single Article, Batch Download, Monitor Download) share common infrastructure but have distinct workflows.
- **Test All Modes**: After making changes, verify that all three download modes still function correctly
- **Proxy State Management**: Be careful when modifying proxy-related code as it affects both Batch and Monitor modes
- **Worker Thread Safety**: Changes to worker.ts should consider all DlEventEnum values (ONE, BATCH_WEB, BATCH_DB, BATCH_SELECT)
- **UI State Consistency**: Frontend changes should maintain proper button states and user feedback for all modes

## Monitor Mode Specific Notes
- **Critical Proxy Pattern**: Always set `PROXY_SERVER = null` after `PROXY_SERVER.close()` to enable proper reuse
- **Download Interval**: Located in `downloadOption.dlInterval` (default: 100ms), affects sequential download speed in `batchDownloadFromWebSelect()`
- **Two-Step Process**: Start monitoring (collects URLs) → Stop monitoring (triggers download of collected articles)
- **Proxy Lifecycle**: `monitorLimitArticle()` → `createProxy()` → collect URLs → `stopMonitorLimitArticle()` → `cleanupProxyImmediate()` → download articles

---

# COMPREHENSIVE PROJECT ANALYSIS

## 🏗️ Architecture Overview

### Core Architecture Pattern
- **Main Process**: Electron backend with system integration
- **Renderer Process**: Vue 3 frontend with Element Plus UI
- **Worker Threads**: Background processing for downloads and EPUB generation
- **Proxy System**: AnyProxy-based HTTP interception for WeChat API calls

### Directory Structure
```
src/
├── main/                    # Electron Main Process
│   ├── index.ts            # App initialization, proxy management
│   ├── service.ts          # Business logic classes & utilities
│   ├── worker.ts           # Download worker thread
│   ├── epubWorker.ts       # EPUB generation worker
│   ├── utils.ts            # Utility functions
│   └── logger.ts           # Logging configuration
├── renderer/src/            # Vue 3 Frontend
│   ├── views/
│   │   ├── Home.vue        # Main download interface
│   │   ├── Setting.vue     # Configuration panel
│   │   └── EpubCreator.vue # EPUB generation interface
│   └── App.vue             # Root component
└── preload/                 # Electron Preload Scripts
    └── index.ts            # IPC bridge
```

## 🔧 Key Dependencies

### Production Dependencies
- **electron**: Desktop app framework
- **vue**: Frontend framework (v3.4.15)
- **element-plus**: UI component library
- **anyproxy**: HTTP proxy server for intercepting WeChat API
- **cheerio**: Server-side HTML parsing
- **turndown**: HTML to Markdown conversion
- **@lesjoursfr/html-to-epub**: EPUB generation
- **mysql2**: Database connectivity
- **axios**: HTTP client with retry mechanism
- **electron-store**: Settings persistence
- **electron-log**: Logging system
- **electron-updater**: Auto-update functionality

### Development Dependencies
- **electron-vite**: Build system
- **typescript**: Type checking
- **vue-tsc**: Vue TypeScript compiler
- **eslint + prettier**: Code quality tools

## 🔌 API Endpoints & Data Flow

### WeChat API Endpoints Intercepted
1. **Article List**: `https://mp.weixin.qq.com/mp/profile_ext?action=getmsg`
2. **Comments**: `https://mp.weixin.qq.com/mp/appmsg_comment?action=getcomment`
3. **Comment Replies**: `https://mp.weixin.qq.com/mp/appmsg_comment?action=getcommentreply`
4. **QQ Music Info**: `https://mp.weixin.qq.com/mp/qqmusic?action=get_song_info`
5. **Proxy Triggers**:
   - Batch mode: `https://mp.weixin.qq.com/mp/getbizbanner`
   - Monitor mode: `https://mp.weixin.qq.com/mp/geticon`

### Parameter Extraction
The proxy system extracts these critical parameters:
- `__biz`: WeChat account identifier
- `uin`: WeChat user ID
- `key`: Authentication parameter
- `pass_ticket`: Session ticket
- Additional headers: Cookie, User-Agent, Host

### Data Flow Architecture
```
User Input → Frontend (Vue) → IPC → Main Process → Worker Thread → WeChat API
                                                 ↓
MySQL Database ← File System ← Content Processing ← Response Data
```

## 🚀 Core Features

### Download Modes

#### 1. **Single Article Mode** (`DlEventEnum.ONE`)
**Properties:**
- **UI Button**: "下载" (Download)
- **Proxy Required**: ❌ No
- **Trigger Method**: Direct URL input
- **Worker Event**: `DlEventEnum.ONE`

**Main Function:**
```typescript
// Frontend: Home.vue:67 - dlOne()
// Backend: index.ts:412 - downloadOne(url)
// Worker: DlEventEnum.ONE → axiosDlOne(articleInfo)
```

**Key Features:**
- Direct download from WeChat article URL
- No parameter extraction needed
- Immediate processing without proxy setup
- Single article processing

#### 2. **Batch Download Mode** (`DlEventEnum.BATCH_WEB` / `DlEventEnum.BATCH_DB`)
**Properties:**
- **UI Button**: "批量下载" (Batch Download)
- **Proxy Required**: ✅ Yes (web source) / ❌ No (DB source)
- **Trigger URL**: `https://mp.weixin.qq.com/mp/getbizbanner`
- **Worker Events**: `DlEventEnum.BATCH_WEB` or `DlEventEnum.BATCH_DB`

**Main Function:**
```typescript
// Frontend: Home.vue:83 - dlBatch()
// Backend: index.ts:423 - monitorArticle()
// Worker: batchDownloadFromWeb() or batchDownloadFromDb()
```

**Key Features:**
- **Two Sub-modes**: Web API source vs Database source
- **Parameter Extraction**: `__biz`, `uin`, `key`, `pass_ticket`, headers
- **Auto-cleanup**: 30-second timeout with proxy cleanup
- **User Confirmation**: Dialog before starting download
- **Full Account History**: Downloads entire WeChat account's articles

#### 3. **Monitor Download Mode** (`DlEventEnum.BATCH_SELECT`)
**Properties:**
- **UI Button**: "监控下载" (Monitor Download) - Toggle button
- **Proxy Required**: ✅ Yes
- **Trigger URL**: `https://mp.weixin.qq.com/mp/geticon`
- **Worker Event**: `DlEventEnum.BATCH_SELECT`

**Main Function:**
```typescript
// Frontend: Home.vue:90 - dlRestrictionsBatch()
// Backend: index.ts:594 - monitorLimitArticle() / stopMonitorLimitArticle()
// Worker: batchDownloadFromWebSelect(articleArr)
```

**Key Features:**
- **Two-step Process**: Start monitoring → Collect URLs → Stop & download
- **URL Collection**: Builds `articleArr[]` from opened articles
- **Manual Control**: User decides when to start/stop monitoring
- **Parameter Extraction**: From `Referer` header (`mid`, `sn`, `chksm`, `idx`)
- **Selective Download**: User chooses specific articles to download

### Output Formats
- **HTML**: Styled with custom CSS for optimal reading
- **Markdown**: Clean conversion with preserved formatting
- **PDF**: Browser-generated via Electron's printToPDF
- **EPUB**: Compiled e-book format with metadata
- **MySQL**: Structured database storage

### Advanced Features
- **Comment Extraction**: Full comment threads with replies
- **Media Handling**: Images, audio, and QQ Music integration
- **Filter System**: Title/author-based include/exclude patterns
- **Retry Logic**: Axios-retry with exponential backoff
- **Anti-Crawling Protection**: 60-second cooldown on detection

## 🛠️ Business Logic Classes

### Core Service Classes
```typescript
// Main data structures
class GzhInfo          // WeChat account metadata
class ArticleInfo      // Article content & metadata
class DownloadOption   // User configuration settings
class FilterRuleInfo   // Content filtering rules
class PdfInfo          // PDF generation data
class ArticleMeta      // Article metadata (author, publish time, etc.)

// Worker communication
class NodeWorkerResponse // Thread messaging protocol
enum NwrEnum            // Response codes
enum DlEventEnum        // Download event types
```

### Key Service Methods
- **HTML Processing**: Image data-src conversion, style extraction
- **Content Conversion**: HTML→Markdown with custom Turndown rules
- **Comment System**: Nested reply structure with geolocation
- **Filter Engine**: Regex-based title/author filtering
- **Database Operations**: MySQL CRUD with duplicate key handling

## 🔒 System Integration

### Proxy Management
- **Port**: Fixed at 8001
- **Certificate**: Auto-generated CA certificate in `~/.anyproxy/certificates/`
- **Platform Support**: 
  - Windows: Auto-install via certutil
  - macOS: Manual installation required
  - Linux: Manual installation required

### Critical Cleanup System
Implements comprehensive proxy cleanup on app exit:
```typescript
// Multiple cleanup strategies for robustness
1. AnyProxy.utils.systemProxyMgr.disableGlobalProxy()
2. Platform-specific system commands
3. Registry cleanup (Windows)
4. Network service cleanup (macOS)
```

### Auto-Update System
- **Framework**: electron-updater
- **Strategy**: Manual download confirmation
- **Config**: Configurable update server
- **Process**: Background download → User confirmation → Restart & install

## 🗄️ Database Schema

### Article Storage Table
```sql
CREATE TABLE wx_article (
  title VARCHAR(255),
  content LONGTEXT,
  author VARCHAR(100),
  content_url VARCHAR(500) PRIMARY KEY,
  create_time DATETIME,
  copyright_stat INT,
  comm JSON,           -- Comments data
  comm_reply JSON,     -- Reply data
  digest TEXT,         -- Article summary
  cover VARCHAR(500),  -- Cover image URL
  js_name VARCHAR(100), -- Account name
  md_content LONGTEXT  -- Markdown content
);
```

## ⚙️ Configuration System

### Settings Storage
- **Framework**: electron-store
- **Location**: OS-specific app data directory
- **Format**: JSON with type-safe access

### Key Configuration Options
- **Download Sources**: Web API vs Database
- **Threading**: Single vs Multi-threaded
- **Intervals**: Download delay (default: 500ms)
- **Batch Limits**: Articles per batch (default: 10)
- **Output Formats**: HTML, Markdown, PDF toggles
- **File Organization**: Account-based directory structure
- **MySQL Integration**: Full database configuration

## 🔍 Worker Thread Architecture

### Main Worker (worker.ts)
- **Purpose**: Article downloading, HTML/Markdown conversion, MySQL integration
- **Responsibilities**:
  - HTTP requests to WeChat APIs
  - Content parsing and cleaning
  - Format conversion (HTML→Markdown)
  - Database operations
  - File system operations

### EPUB Worker (epubWorker.ts)
- **Purpose**: EPUB file generation from downloaded articles
- **Process**: HTML collection → Metadata extraction → EPUB compilation

### Communication Protocol
```typescript
enum NwrEnum {
  START,         // Initialize worker
  SUCCESS,       // Log success message
  FAIL,          // Log failure message
  ONE_FINISH,    // Single download complete
  BATCH_FINISH,  // Batch download complete
  CLOSE,         // Terminate worker
  PDF,           // Generate PDF
  PDF_FINISHED   // PDF generation complete
}
```

## 🚨 Security Considerations

### Anti-Crawling Protection
- **Rate Limiting**: Configurable download intervals
- **Failure Handling**: 60-second cooldown on detection
- **Retry Logic**: Maximum 1 retry with extended delays
- **User-Agent Rotation**: Maintains browser headers

### Certificate Management
- **CA Generation**: Automatic root certificate creation
- **Installation**: Platform-specific auto-install (Windows only)
- **Security**: Self-signed certificates for HTTPS interception

### Data Privacy
- **Local Storage**: All data stored locally by default
- **Optional MySQL**: User-controlled database integration
- **No Telemetry**: No external data transmission except WeChat APIs

## 🐛 Common Issues & Solutions

### Proxy Problems
- **Symptom**: "Can't connect to internet"
- **Cause**: Proxy settings not cleared on abnormal exit
- **Solution**: Multiple cleanup strategies implemented in v1.7+

### Certificate Issues
- **Symptom**: HTTPS interception fails
- **Cause**: CA certificate not installed or trusted
- **Solution**: Settings → Install Certificate → Restart app

### Rate Limiting
- **Symptom**: "Access denied" or empty responses
- **Cause**: WeChat anti-crawling protection triggered
- **Solution**: Automatic 60-second cooldown + retry logic

## 📁 File Organization

### Default Paths
- **Downloads**: `{userData}/savePath/`
- **Temporary**: `{temp}/wechatDownload/`
- **Logs**: `{appData}/wechatDownload/logs/`
- **Settings**: `{userData}/config.json`
- **Certificates**: `~/.anyproxy/certificates/`

### File Naming Convention
- **Articles**: `{title}.html`, `{title}.md`
- **Organization**: Optional account-based directories
- **Sanitization**: Windows-compatible filename cleaning

## 🚀 Performance Optimizations

### Multi-threading
- **Worker Isolation**: Heavy processing in separate threads
- **UI Responsiveness**: Non-blocking main thread
- **Memory Management**: Worker termination after completion

### Batch Processing
- **Configurable Limits**: Prevent memory overflow
- **Progressive Loading**: Fetch → Process → Save cycle
- **Retry Mechanism**: Automatic failure recovery

### Caching Strategy
- **Temporary Files**: Efficient intermediate storage
- **Database Optimization**: Duplicate key handling
- **Image Handling**: Local storage with fallback URLs
