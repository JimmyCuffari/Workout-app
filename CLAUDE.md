# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running Locally

There is no build step. Open `index.html` directly in a browser, or serve it with any static HTTP server:

```
python -m http.server 8000
```

There are no tests, no linter, and no package manager.

## Architecture

This is a single-file PWA. All application logic lives in `index.html` (~855 lines). `sw.js` is the service worker (37 lines).

**Storage & Sync:**
- Primary store: `localStorage` under key `'wt-data-v1'`
- Cloud backend: JSONBin (`https://api.jsonbin.io/v3/b/`) — credentials `BIN_ID` / `API_KEY` are hardcoded near line 361
- Sync is local-first; on reconnect the app merges remote + local and writes back

**State shape (top-level JS variables ~line 373):**
```
sessions[]        // {id, profile, date, exercises[], note}
customExercises[] // user-added exercise names
customProfiles[]  // profiles beyond the two defaults (Jimmy, Alan)
activeProfile     // currently selected profile id
view              // 'log' | 'history' | 'progress' | 'settings' | 'adduser' | 'confirm'
draft             // the in-progress session object
darkMode          // boolean, persisted to localStorage('wt-dark')
```

**Data layer (~lines 379–426):**
- `saveLocal()` / `loadLocal()` — read/write localStorage
- `fetchBin()` / `putBin(data)` — JSONBin GET/PUT

**Rendering (~lines 486–612):**  
`render()` is the single orchestrator; it delegates to `renderLog()`, `renderHistory()`, `renderProgress()`, `renderSettings()`, `renderAddUser()`, `renderConfirm()` based on `view`.

**Session mutations (~lines 763–820):**  
`saveSession()`, `addSet(ei)`, `removeSet(ei, si)`, `updSet(ei, si, f, v)`, `addCustomEx()`, `removeEx(ei)`, `saveNote()`

**Exercise name normalization:**  
`NAME_MAP` and `migrateNames()` (near line 396) canonicalize historical spelling variants on every load — update this map when renaming exercises.

**Service Worker (`sw.js`):**  
Cache-first strategy with cache name `'wt-v1'`. JSONBin requests are always bypassed to the network (line 22).
