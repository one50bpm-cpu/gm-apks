# GM apks

A single-page sideload hub for Android-TV APKs, hosted free on GitHub Pages.

- **Live page:** https://one50bpm-cpu.github.io/gm-apks/
- **On a TV (Downloader app code):** `7934617` → redirects to the live page
- **Repo:** https://github.com/one50bpm-cpu/gm-apks

---

## 1. How it works (the 30-second version)

It's a **static website** — just files in this repo, served by GitHub Pages. There is no server, no database, no build step.

- `index.html` — the whole page: layout, styling, and the small JavaScript that draws the app cards.
- `manifest.json` — the **list of apps** and their versions/links. This is the file you normally edit.
- `*.apk` — the app files that are hosted directly in the repo (the "Download Local" button points at these).
- `explosion.webp` — the title graphic behind the 🤜🏻🤛🏻 fists.
- `robots.txt` — tells search engines not to index the page.

When someone opens the page, the JavaScript in `index.html` does this:

1. Tries to **fetch `manifest.json`** (the live app list).
2. If that works, it draws one card per app from the manifest.
3. If the fetch fails (e.g. the file is opened offline), it falls back to a **baked-in copy** of the list that lives inside `index.html` (the `FALLBACK` block).

> ⚠️ **Important:** because of that fallback, the app list exists in **two places** — `manifest.json` **and** the `FALLBACK` array inside `index.html`. When you change the list, update **both** so they stay in sync. (The live page reads `manifest.json`; the fallback is just insurance.)

---

## 2. What each card shows

Every app is one card. Depending on the data it has, a card shows:

**Version lines**
- `Local version: vX` — the version of the APK hosted in this repo (only shown if there is a local file).
- `Official version: vY` — the latest version upstream. If `Y` is newer than `X`, it turns **green** with a **NEWER** tag.

**Buttons (up to three)**
- **Download / Local** — downloads the APK hosted in this repo. *Only appears if the app has a local file.*
- **Official / Website** — opens the app's official site/page. *Only appears if `official` is set.*
- **Try Direct / Download** — a stable "always-latest" direct link that the host resolves to the newest APK each time. *Only appears if `direct` is set.*

---

## 3. The `manifest.json` format

```json
{
  "updated": "2026-10-10",
  "apps": [
    {
      "id": "smarttube",                 // unique short slug
      "name": "SmartTube",               // display name on the card
      "version": "32.56",                // version of the LOCAL (repo-hosted) apk
      "file": "SmartTube-v32.56.apk",    // local apk filename in this repo ("" = no local copy)
      "size": 25046686,                  // bytes (shown as MB). Use 0 if no local file
      "official": "https://github.com/yuliskov/SmartTube/releases/latest",  // "Official Website" button ("" = hide)
      "direct": "https://github.com/yuliskov/SmartTube/releases/download/latest/smarttube_stable.apk", // "Try Direct Download" button (optional — omit to hide)
      "latest": {                        // what "Official version" shows
        "version": "32.56",              // latest upstream version
        "url": "https://..."             // where that latest lives
      }
    }
  ]
}
```

Field notes:
- `file: ""` + `size: 0` → **link-only** app (no "Download Local" button, no "Local version" line). Used for SportzXLive and Proton VPN.
- `official: ""` → hides the "Official Website" button (used for Projectivy, which are hosted-from-server only).
- `direct` is optional — leave it out to hide the "Try Direct Download" button.
- `latest.version` drives the green **NEWER** flag: if it's higher than `version`, the card flags an update. Versions are compared number-by-number (`3.0.5 > 3.0.4`).

---

## 4. How to make changes

> After **any** change: `git add` → `git commit` → `git push`. GitHub Pages rebuilds automatically in ~1–2 minutes. Then **hard-refresh** the page (Ctrl/Cmd+Shift+R) to beat browser cache.

### 4a. Change an app's "Official version" (flag an update)
Edit `manifest.json` → that app's `latest.version` (and `latest.url` if the download moved). Mirror it in the `FALLBACK` block in `index.html`. Example — Netfly got a 3.0.5:
```json
"latest": { "version": "3.0.5", "url": "https://netflyapp.com/file/netfly_tv.apk" }
```
The card will now show `Official version: v3.0.5` in green with **NEWER**.

### 4b. Upload a newer local APK
1. Copy the new `.apk` into the repo folder (keep a clear name, e.g. `Reezn-v2.1.0.apk`).
2. In `manifest.json` **and** the `FALLBACK` block, update that app's `file`, `version`, and `size` (bytes).
3. Commit & push (the APK is committed into the repo).

Get a file's size in bytes:
```bash
stat -c%s Reezn-v2.1.0.apk
```

### 4c. Add a brand-new app
Append an object to `apps` in `manifest.json` **and** add a matching line to the `FALLBACK` block in `index.html`. Two patterns:

**With a hosted APK:**
```json
{ "id": "myapp", "name": "My App", "version": "1.0", "file": "MyApp-v1.0.apk", "size": 12345678,
  "official": "https://example.com", "latest": { "version": "1.0", "url": "https://example.com" } }
```
(…and drop `MyApp-v1.0.apk` into the repo.)

**Link-only (no hosted APK):**
```json
{ "id": "myapp", "name": "My App", "version": "1.0", "file": "", "size": 0,
  "official": "https://example.com/download",
  "direct": "https://example.com/latest.apk",
  "latest": { "version": "1.0", "url": "https://example.com/download" } }
```

### 4d. Remove an app
Delete its object from `manifest.json` and its line from `FALLBACK`. Optionally `git rm` the hosted `.apk` to reclaim space.

### 4e. Change the title text
Edit the `<h1>` in `index.html` (currently `GM apks · 7934617`) and the `<title>` tag.

### 4f. Change the explosion / fists graphic
- The explosion is `explosion.webp`, layered behind the fists.
- To swap it, replace `explosion.webp` and bump the cache tag in `index.html` (`src="explosion.webp?v=3"` → `?v=4`) so browsers re-fetch it.
- Size/position/scale of the whole logo is controlled by `.emoji-logo` in the `<style>` block (`font-size` scales it; `transform: translateY(...)` moves it up/down).
- The fists are the emoji `🤜🏻🤛🏻` inside `<span class="arms">`; their black outline is the `text-shadow` on `.arms`.

---

## 5. Hosting & privacy notes

- **Free GitHub Pages has no login/password option** — anyone with the link can open the page (needed so TVs can download anonymously via Downloader).
- The page is **de-indexed** (`robots.txt` + `noindex` meta) so it won't show in search engines, and it doesn't link back to the repo. The repo itself is still public (required for free Pages) and reachable if someone types its URL.
- APKs committed here live in the repo's git history.

## 6. Device note (Sony Bravia / this setup)

The TVs here are **32-bit `armeabi-v7a`**. For SmartTube use the **universal `smarttube_stable.apk`** (the "Visit APK Website" link already points there) — the `arm64-v8a` build will **not** install on 32-bit. Kodi here is the `armeabi-v7a` Omega build for the same reason.

## 7. Verified APK origins

These were confirmed by inspecting each APK's signing cert and embedded update URLs:

| App | Package | Origin |
|---|---|---|
| CinemaHQ | `com.app.mlounge` | self-updates from `cinemahq.app` → `chq-api.ddns.me` |
| Netfly | `org.drive_hunter` | `netflyapp.com` |
| Reezn | `com.rmedia.rplayer` | `reezntvapp.com` |
| SmartTube | `org.smarttube.stable` | GitHub `yuliskov/SmartTube` (cert: `O=SmartTube`) |
| Projectivy Launcher | `com.spocky.projengmenu` | Play Store / `projectivylauncher.com` (cert: Despesse Mickael) |
| Projectivy Icon Pack | `dev.projectivy.blueprint.app` | GitHub `SicMundus86/ProjectivyIconPack` |
| Kodi | `org.xbmc.kodi` | `kodi.tv` |
| Proton VPN | `ch.protonvpn.android` | GitHub `ProtonVPN/android-app` |
