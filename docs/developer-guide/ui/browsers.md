# Browser Compatibility

PhotoPrism ships a single-page app that targets evergreen browsers. The loader script at `assets/static/js/browser-check.js` (rendered through `assets/templates/index.gohtml`) blocks unsupported clients before Vue boots, so keep this page aligned with the logic in that script.

## Supported Platforms

- Chrome, Edge, and Firefox: latest stable versions on Windows, macOS, and Linux
- Safari 13+ on macOS and iOS 13+ (the splash screen warns older iOS devices)
- Chromium-based mobile browsers that ship with modern ES2019 features

Internet Explorer is **not** supported. Legacy Android WebView builds without ES modules or Fetch support will hit the browser-check warning.

When introducing APIs that may not exist on the minimum baseline (for example `AbortController` on Safari 13), add a capability check or a lightweight polyfill under `assets/static/js/browser-check.js`.

## Testing

- Run the Vitest unit suite (`make vitest-watch`) on every UI change.
- Use the “Devices” tab in Chrome DevTools or Safari’s Responsive Design Mode to spot layout regressions on phones and tablets.
- [BrowserStack](https://www.browserstack.com/) remains free for open-source projects and is the easiest way to test on edge versions of Safari, iOS, and legacy Android without owning physical devices.
- Capture baseline screenshots for new layouts via the Playwright workflows documented in `AGENTS.md` so we can diff rendering changes over time.
