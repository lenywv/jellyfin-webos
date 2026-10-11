# Workspace Guidelines: Jellyfin webOS Project

## Project Structure & Architecture
- This repository (`jellyfin-webos`) is the thin native webOS wrapper application for LG TVs.
- The wrapper contains:
  - `frontend/index.html`: Entry container with `#contentFrame` iframe that hosts Jellyfin Web (`/web/index.html`).
  - `frontend/css/webOS.css`: CSS styling injected into the webOS client / Jellyfin interface.
  - `frontend/js/webOS.js`: Native shell adapter, capabilities bridge, and webOS platform integration.
  - `services/`: Node.js background services for webOS (SSDP server discovery).
- Note: Core Jellyfin Web (`jellyfin-web`) and server logic are external to this repo. Changes to UI/styling here must be made via `frontend/css/webOS.css` or adapter logic in `frontend/js/webOS.js`.

## Debugging Workflow & Architecture Boundaries
1. **Understand Where Logic Actually Lives Before Modifying Code:**
   - `jellyfin-webos` is ONLY a wrapper shell. It does NOT bundle core UI views, media routing, image generation, or Jellyfin Web source.
   - Do NOT attempt to monkey-patch or write complex JS hacks (like overriding key events or monkey-patching `scrollTo`) when the core behavior is governed by Jellyfin Web or the server.
   - For UI glitches or styling behaviors, always inspect what CSS and settings Jellyfin Web already provides (e.g. `enableSmoothScroll` in localStorage, `.cardPadder`, `.cardBox`) before inventing new overrides.
2. **First Check Server / Web Layer for Content and Media Issues:**
   - If images are blurred, missing, or content fails to render, check the Jellyfin server logs, server libraries, and network responses before blaming the webOS client wrapper. Jellyfin uses blurhash placeholders that only resolve when the server successfully serves the primary image.
3. **Verify Runtime State Directly (Inspect over Guesswork):**
   - Use DevTools inspection (local browser or simulator) to check computed CSS and actual DOM nodes before assuming what rules apply.
   - Keep interventions surgical: prefer the 1-2 native flags or CSS properties over tens of lines of custom JS workarounds.

## Critical Testing & Verification Rules
1. **Never modify or overwrite the non-dev app on TV:**
   - Production / user app is `org.jellyfin.webos`. Never install, overwrite, or launch over it.
   - For dev testing on physical TV, always use the dev app ID: `org.jellyfin.webos.dev`.
   - Always restore `frontend/appinfo.json` and `services/services.json` to `org.jellyfin.webos` before committing.
2. **Prioritize Local Verification:**
   - Do NOT run intrusive tests on the TV when local verification is requested.
   - Verify UI changes locally using browser testing or the webOS TV Simulator.
   - Clean up any background inspect or test tasks after verification.

## TV UI & Styling Principles
- Keep custom CSS minimal and non-destructive.
- Avoid applying scaling (`transform: scale`) directly on flexbox list rows (`.listItem`) as it disrupts text truncation and alignment.
- When customizing card focus outlines, avoid setting arbitrary `border-radius` directly on `.cardScalable` without considering its native transparent spacing border; apply focus rings on `.cardPadder` instead.
