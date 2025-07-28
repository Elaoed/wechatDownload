# WeChat Download Project Commands

## Common Commands
- **List files**: `ls -la`
- **Search text**: `grep -r "pattern" .` or use `rg "pattern"` (ripgrep)
- **Find files**: `find . -name "*.ts"` or `find . -name "*.vue"`
- **View file**: `cat filename`
- **File permissions**: `chmod 755 filename`

## Project-specific Commands
- **Build**: `npm run build`
- **Type check**: `npm run typecheck`
- **Lint**: `npm run lint`
- **Format code**: `npm run format`
- **Start dev**: `npm run dev`
- **Build for different platforms**:
  - Windows: `npm run build:win`
  - macOS: `npm run build:mac`
  - Linux: `npm run build:linux`

## Development Workflow
1. Install dependencies: `npm install`
2. Run development server: `npm run dev`
3. Type check: `npm run typecheck`
4. Lint and format: `npm run lint && npm run format`
5. Build for production: `npm run build`

## Key Project Files
- **Main process**: `src/main/index.ts`
- **Worker thread**: `src/main/worker.ts`
- **Service logic**: `src/main/service.ts`
- **Frontend**: `src/renderer/src/views/Home.vue`
- **Settings**: `src/renderer/src/views/Setting.vue`

## Debug Commands
- **View logs**: Check electron-log output in app data directory
- **Open dev tools**: F12 when app is running
- **Check proxy status**: Monitor network requests in dev tools
- **Check proxy running**: `lsof -i :8001` (check if port 8001 is in use)
- **Check system proxy**: `scutil --proxy` (macOS) or `netsh winhttp show proxy` (Windows)
- **View worker errors**: Check console for worker thread errors

## Troubleshooting Batch Download Issues
1. **Check CA certificate**: Settings → Install Certificate
2. **Verify proxy setup**: Check if port 8001 is accessible
3. **WeChat app**: Must open NEW articles in WeChat (not browser)
4. **Network**: Ensure no conflicting proxies or VPNs
5. **Logs**: Check detailed logs in electron-log files for specific errors

## Common Error Solutions
- **IMKCFRunLoopWakeUpReliable**: Harmless macOS system warning, ignore
- **Nothing downloads**: Check CA certificate installation and proxy status
- **Timeout errors**: Ensure proper WeChat article opening procedure
- **Parameter extraction fails**: Check network connectivity and WeChat session
