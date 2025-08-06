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
