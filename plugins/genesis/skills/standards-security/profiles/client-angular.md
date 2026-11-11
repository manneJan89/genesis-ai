# Security: client profile, Angular
- Never call `bypassSecurityTrustHtml`, `bypassSecurityTrustUrl`,
  `bypassSecurityTrustResourceUrl` or `bypassSecurityTrustScript` on anything
  derived from user, server or URL data. Each call is a hole in Angular's
  built-in protection and needs a stated reason.
- `[innerHTML]` only through Angular's own sanitization (no bypass), and
  never write HTML through `ElementRef.nativeElement.innerHTML`,
  `Renderer2.setProperty(..., 'innerHTML', ...)` or `document.write`.
