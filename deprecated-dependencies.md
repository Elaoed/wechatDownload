# Deprecated Dependencies Analysis

## Overview
Analysis of deprecated and vulnerable dependencies in the WeChat Download project, categorized by importance level as of August 2025.

---

# 🔴 CRITICAL LEVEL - Immediate Action Required

> **Impact**: Security vulnerabilities that could compromise application security or break core functionality

## Runtime Dependencies with Security Issues

### 1. **anyproxy (v4.1.3)** ⚠️ CORE FUNCTIONALITY
- **Role**: Essential for WeChat API interception (core feature)
- **Vulnerabilities**: Multiple critical issues through dependencies
  - form-data: Unsafe random boundary selection (CRITICAL)
  - node-forge: Cryptographic signature verification issues (CRITICAL)
  - uglify-js: Regular Expression DoS (CRITICAL) 
  - underscore: Arbitrary Code Execution (CRITICAL)
- **Risk**: High security risk but breaking core functionality if removed
- **Action**: Research alternatives or accept risk temporarily

### 2. **axios (v1.6.8)**
- **Role**: HTTP client for API requests
- **Vulnerability**: Server-Side Request Forgery (HIGH)
- **Fix**: Update to 1.11.0 (non-breaking)
- **Action**: ✅ Safe to update immediately

### 3. **electron (v28.2.9)**
- **Role**: Application framework
- **Vulnerability**: Heap Buffer Overflow (MODERATE)
- **Fix**: Update to 37.2.6 (major version)
- **Action**: Update with thorough testing

### 4. **electron-updater (v6.1.8)**
- **Role**: Auto-update functionality
- **Vulnerability**: Code Signing Bypass on Windows (HIGH)
- **Fix**: Update to 6.6.2 (non-breaking)
- **Action**: ✅ Safe to update immediately

---

# 🟡 MEDIUM LEVEL - Important Updates

> **Impact**: Functionality improvements, moderate security issues, or outdated major versions

## Security & Functionality Updates

### 1. **@mozilla/readability (v0.4.4)**
- **Role**: HTML content extraction
- **Issue**: DoS through Regex vulnerability
- **Fix**: Update to 0.6.0 (breaking change)
- **Priority**: High security, test thoroughly

### 2. **Core Build Tools**
- **electron-vite** (2.1.0 → 4.0.0): Build system improvements
- **electron-builder** (24.13.3 → 26.0.12): Packaging improvements
- **vite** (5.2.7 → 7.0.6): Build performance enhancements

### 3. **Application Dependencies**
- **element-plus** (2.6.3 → 2.10.5): UI components bug fixes
- **mysql2** (3.9.3 → 3.14.3): Database driver improvements
- **vue** (3.4.21 → 3.5.18): Framework improvements
- **electron-store** (8.2.0 → 10.1.0): Storage API enhancements

### 4. **Transitive Vulnerabilities - Medium Impact**
- **braces**: Uncontrolled resource consumption
- **cross-spawn**: Regular Expression DoS
- **path-to-regexp**: Backtracking regular expressions
- **ws**: DoS when handling many HTTP headers

---

# 🟢 LOW LEVEL - Non-Critical Updates

> **Impact**: Development experience, code quality, or minor version updates

## Development Tools

### 1. **Linting & Code Quality**
- **eslint** (8.57.0 → 9.32.0): New linting rules
- **prettier** (3.2.5 → 3.6.2): Code formatting improvements
- **@electron-toolkit/eslint-config** (1.0.2 → 2.1.0)
- **@electron-toolkit/eslint-config-ts** (1.0.1 → 3.1.0)

### 2. **Build System Plugins**
- **unplugin-auto-import** (0.17.5 → 20.0.0): Major version jump
- **unplugin-vue-components** (0.26.0 → 29.0.0): Major version jump
- **@vitejs/plugin-vue** (5.0.4 → 6.0.1): Vue plugin updates

### 3. **TypeScript & Vue Tooling**
- **typescript** (5.4.3 → 5.9.2): Language improvements
- **vue-tsc** (1.8.27 → 3.0.5): Type checking improvements
- **@vue/eslint-config-typescript** (12.0.0 → 14.6.0)

### 4. **Utility Libraries**
- **turndown** (7.1.3 → 7.2.0): Markdown conversion
- **@lesjoursfr/html-to-epub** (4.2.3 → 4.5.2): EPUB generation
- **axios-retry** (4.1.0 → 4.5.0): HTTP retry logic

## Upgrade Progress

### ✅ COMPLETED UPGRADES

#### Low Priority (Completed ✅)
- **prettier**: 3.2.5 → 3.6.2 ✅
- **turndown**: 7.1.3 → 7.2.0 ✅ 
- **@lesjoursfr/html-to-epub**: 4.2.3 → 4.5.2 ✅
- **axios-retry**: 4.1.0 → 4.5.0 ✅
- **typescript**: 5.4.3 → 5.9.2 ✅
- **@vitejs/plugin-vue**: 5.0.4 → 6.0.1 ✅
- **vue-tsc**: 1.8.27 → 2.1.10 ✅

#### Medium Priority (Completed ✅)
- **@mozilla/readability**: 0.4.4 → 0.6.0 ✅ (Security fix)
- **element-plus**: 2.6.3 → 2.10.5 ✅
- **mysql2**: 3.9.3 → 3.14.3 ✅
- **vue**: 3.4.21 → 3.5.18 ✅
- **electron-log**: 5.1.2 → 5.4.2 ✅
- **electron-vite**: 2.1.0 → 4.0.0 ✅
- **vite**: 5.2.7 → 5.4.19 ✅
- **electron-builder**: 24.13.3 → 26.0.12 ✅
- **electron-store**: 8.2.0 (kept at v8 - v10 has breaking changes) ⚠️

### ⚠️ NOTES FROM UPGRADES
- **electron-store v10** has breaking TypeScript types - downgraded to v8.2.0
- Fixed TypeScript error in PDF generation (`Buffer` → `Uint8Array`)
- All builds, type checking, and linting pass successfully
- Reduced vulnerabilities from 44 to 35 

---

## Commands to Fix Remaining Issues
```bash
# Non-breaking fixes
npm audit fix

# Breaking changes (use with caution)
npm audit fix --force

# Critical updates (manual - test thoroughly)
npm update axios electron-updater electron
```

## Notes
- **anyproxy** is critical for WeChat API interception but has significant security issues
- Consider proxy alternatives or accept security risks for functionality
- **electron-store v10** incompatible - staying on v8.2.0 for now
- Test thoroughly after critical updates, especially for **anyproxy** and **electron** updates