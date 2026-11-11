# Security: client profile, Flutter
- WebViews with JavaScript enabled never load untrusted content;
  deep links and push-notification payloads are validated before acting on
  them; secrets go in platform secure storage, never `SharedPreferences`.
