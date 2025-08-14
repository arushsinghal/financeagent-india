# Changelog

## 0.2.0
- Package renamed to `financeagent-india`.
- Removed all non-India utilities and currency parsing logic.
- Simplified public API; unsupported markets now raise `ValueError`.
- Trimmed constants and internal helpers to minimal surface.

## 0.2.1
- Removed unused imports.
- Added explicit install_requires (requests, beautifulsoup4).
- Pre-publish verification adjustments.

## 0.1.0
- Initial pruned release: India (NSE) only retained from original multi-market project.
