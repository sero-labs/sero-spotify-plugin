# @sero-ai/plugin-spotify

Spotify plugin for Sero — browse playlists, play tracks, and let the Sero Agent build mixes for you.

## Features

- **Spotify Web Playback SDK** — in-app playback (requires Widevine DRM)
- **Playlist browsing** — sync your Spotify library, browse tracks
- **Agent tools** — create playlists, search tracks, get recommendations via the `spotify` tool
- **OAuth PKCE** — secure, client-only Spotify auth flow

## Installation

```bash
# From the Sero plugins directory
git clone <repo-url> sero-spotify-plugin
cd sero-spotify-plugin
npm install
npm run build
```

Then register the plugin in Sero's settings or restart Sero to auto-discover it.

## Prerequisites — Widevine DRM

The Spotify Web Playback SDK requires Widevine DRM for audio decryption.
Sero uses the [castlabs Electron fork](https://github.com/castlabs/electron-releases)
which bundles Widevine CDM support. On macOS, the Electron binary must also be
**VMP-signed** with production credentials or Spotify's license server will
reject playback requests.

**One-time setup:**

```bash
# 1. Install the signing tool
pipx install castlabs-evs

# 2. Create a free castlabs EVS account
evs-account signup

# 3. Sign the Electron binary (from the Sero desktop app directory)
cd apps/desktop && bash scripts/sign-vmp.sh
```

Re-run `scripts/sign-vmp.sh` after any `npm install` that re-downloads the
Electron binary (the signature is on the binary, not in source).

## Setup

1. Create a Spotify app at https://developer.spotify.com/dashboard.
2. Copy your **Client ID**.
3. Add this redirect URI in Spotify app settings:
   - `http://127.0.0.1:5185/spotify-auth-callback.html` (dev)
4. Build the plugin:

```bash
npm install
npm run build
```

5. Sign the Electron binary for DRM (see Prerequisites above).
6. Start Sero and open **Spotify** in the sidebar.
7. Paste your Client ID, connect, and authorize.

## Development

```bash
npm run dev        # Start Vite dev server on port 5185
npm run build      # Production build → dist/ui/
npm run typecheck  # Type-check the UI code
```

## Callback Troubleshooting

If Spotify redirects to `127.0.0.1:5185` and shows `ERR_CONNECTION_REFUSED`,
the dev server is not running.

1. Start the dev server: `npm run dev`
2. Confirm port `5185` is listening: `lsof -i :5185`

## Agent Tool

Tool name: `spotify`

Actions:

- `connection_status`
- `list_playlists`
- `search_tracks`
- `suggest_songs`
- `create_playlist`
- `add_tracks`
- `start_playlist`
- `pause_playback`
- `next_track`
- `previous_track`

State is global-scoped in `~/.sero-ui/apps/spotify/state.json` (with workspace
fallback in Pi CLI mode).

## Project Structure

```
sero-spotify-plugin/
├── extension/          # Pi extension (agent tools)
│   ├── index.ts        # Extension entry point
│   └── lib/
│       ├── action-handlers.ts  # Tool action implementations
│       ├── spotify.ts          # Spotify API helpers (server-side)
│       └── state.ts            # State file read/write
├── shared/
│   └── types.ts        # Shared TypeScript types
├── ui/                 # Federated React UI
│   ├── SpotifyApp.tsx  # Main app component (MF exposed)
│   ├── components/     # UI components
│   ├── lib/            # Hooks and API helpers
│   ├── styles/         # CSS styles
│   ├── index.html      # Vite entry HTML
│   └── spotify-auth-callback.html  # OAuth callback page
├── package.json
├── vite.config.ts
├── tsconfig.extension.json
└── README.md
```
