# Blackburn Launcher — Battlefield 3 Singleplayer Without Battlelog
[![Releases](https://img.shields.io/badge/Download-Releases-blue?logo=github)](https://github.com/Eagle125/blackburn-launcher/releases)

![Battlefield 3](https://upload.wikimedia.org/wikipedia/en/4/4e/Battlefield_3_cover_art.jpg)

Badges
- [![battlefield](https://img.shields.io/badge/topic-battlefield-2b7bb9)](https://github.com/topics/battlefield)
- [![battlefield-3](https://img.shields.io/badge/topic-battlefield--3-2b7bb9)](https://github.com/topics/battlefield-3)
- [![battlelog](https://img.shields.io/badge/topic-battlelog-2b7bb9)](https://github.com/topics/battlelog)
- [![linux-gaming](https://img.shields.io/badge/topic-linux--gaming-2b7bb9)](https://github.com/topics/linux-gaming)
- [![proton](https://img.shields.io/badge/topic-proton-2b7bb9)](https://github.com/topics/proton)
- [![proxy-server](https://img.shields.io/badge/topic-proxy--server-2b7bb9)](https://github.com/topics/proxy-server)
- [![singleplayer](https://img.shields.io/badge/topic-singleplayer-2b7bb9)](https://github.com/topics/singleplayer)
- [![steam](https://img.shields.io/badge/topic-steam-2b7bb9)](https://github.com/topics/steam)
- [![steam-deck](https://img.shields.io/badge/topic-steam--deck-2b7bb9)](https://github.com/topics/steam-deck)
- [![steam-play](https://img.shields.io/badge/topic-steam--play-2b7bb9)](https://github.com/topics/steam-play)

Purpose
- Let Sargeant Blackburn play Battlefield 3 singleplayer without Battlelog.
- Offer a launcher that routes authentication and campaign services locally.
- Provide options for native Linux, Proton, and Steam Deck.

Quick alignment
- Target: Linux and SteamOS users who want to run BF3 singleplayer without Battlelog.
- Approach: Replace Battlelog dependencies with local proxies and a launcher that starts the game with minimal network hooks.

Download and run
- Visit the Releases to get the launcher binary and helpers:
  https://github.com/Eagle125/blackburn-launcher/releases
- The release page contains one or more artifacts. Download the launcher file for your platform and execute it.

What this repo delivers
- A small proxy server that answers Battlelog calls the singleplayer needs.
- A launcher that sets environment variables and starts Battlefield 3.
- Scripts to run with Proton or native Wine.
- Steam launch options templates.
- Step-by-step guides for Steam Deck and desktop Linux.
- Troubleshooting and logs to diagnose launch failures.

How it works — plain flow
1. Launcher starts a local proxy.
2. The proxy answers Battlelog endpoints the game attempts to reach.
3. The launcher sets keys and tokens the game checks at runtime.
4. The launcher starts the game with adjusted network routes.
5. Singleplayer proceeds without connecting to official Battlelog servers.

Contents
- bin/ — compiled launcher binaries and helper scripts.
- proxy/ — proxy server code and configs.
- scripts/ — automation for Steam/Proton, Steam Deck, and systemd units.
- docs/ — deep guides and logs
- src/ — source for proxy and launcher
- examples/ — config examples for Proton prefixes and launch options

Architecture detail
- Proxy server
  - Written in Go for portability.
  - Listens on localhost, configurable port.
  - Implements endpoints used by BF3 for campaign checks.
  - Returns static JSON, ETags, and tokens the engine expects.
- Launcher
  - Small binary that starts proxy and the game.
  - Sets environment variables for Proton, Wine, and the game.
  - Option to run the game in a sandboxed prefix.
- Scripts
  - Steam compatibility wrapper.
  - systemd user unit to run the proxy at boot.
  - Steam Deck toggle script for controller mapping.
- Logs
  - The launcher writes logs to ~/.local/share/blackburn-launcher/logs.
  - Proxy writes access logs and request dumps for debugging.

Supported platforms
- Desktop Linux (x86_64)
- SteamOS / Steam Deck (proton + linux)
- Experimental: WINE on BSD or macOS (user contributed)

Prerequisites
- A valid copy of Battlefield 3 on Steam.
- Wine or Proton (for Windows build of the game).
- Basic terminal skills.
- The launcher binary from Releases.

Install — Desktop Linux (step by step)
1. Download the release from:
   https://github.com/Eagle125/blackburn-launcher/releases
2. Extract the archive if needed:
   tar xvf blackburn-launcher-*.tar.gz
3. Make the launcher executable:
   chmod +x blackburn-launcher
4. Move the binary to a bin folder:
   sudo mv blackburn-launcher /usr/local/bin/
5. Copy the proxy config:
   mkdir -p ~/.config/blackburn
   cp config/proxy-default.json ~/.config/blackburn/proxy.json
6. Edit the proxy config if you need custom ports or tokens.
7. Start once to generate a log:
   blackburn-launcher --register --steam-path "/home/you/.steam/steam"

Install — Steam (non-Deck)
1. Right-click Battlefield 3 in Steam.
2. Select Properties > General > Launch Options.
3. Add the launcher call:
   /usr/local/bin/blackburn-launcher --steam-game "Battlefield 3" --steam-path "%command%"
4. For Proton, set the Proton version in Properties > Compatibility.
5. Add any Proton overrides in ~/.steam/root/compatibilitytools.d if needed.

Install — Proton prefix manual
1. Locate game prefix:
   ~/.steam/steam/steamapps/compatdata/65500/pfx
2. Copy blackburn proxy files into the prefix:
   cp -r proxy ~/.steam/steam/steamapps/common/blackburn-proxy
3. Set the launcher to run inside prefix with WINE:
   WINEPREFIX=~/.steam/steam/steamapps/compatdata/65500/pfx /usr/local/bin/blackburn-launcher --wine

Steam Deck quick steps
- Use Desktop Mode.
- Open Konsole.
- Download release file from:
  https://github.com/Eagle125/blackburn-launcher/releases
- Make executable and move to /usr/local/bin.
- Add a custom Steam shortcut:
  - Click Add a Game > Add a Non-Steam Game > Browse.
  - Select /usr/local/bin/blackburn-launcher and set the target to the BF3 game.
- Switch back to Gaming Mode.
- Run BF3 from the new shortcut.

Steam Deck tips
- Use Plasma or KDE if you need GUI tools.
- Set controller mapping in Steam for Xbox/PlayStation layout.
- Allow the launcher to adjust SDL_GAMECONTROLLERCONFIG if needed.
- If you use Proton, test first in Desktop Mode.

Configuration file (example)
- Path: ~/.config/blackburn/proxy.json
{
  "listen": "127.0.0.1:8085",
  "endpoints": {
    "/api/1/cheatCheck": { "status": 200, "body": { "ok": true } },
    "/api/1/session": { "status": 200, "body": { "auth": "local", "user": "sarge" } }
  },
  "log": {
    "file": "~/.local/share/blackburn-launcher/logs/proxy.log",
    "level": "info"
  }
}

Launcher options
- --help: print help
- --proxy-config PATH: use custom proxy config
- --steam-path PATH: game path or Steam %command% variable
- --wine: run launcher inside a WINE/Proton prefix
- --register: write a small marker file in the Steam library to allow auto-start
- --no-network: block outgoing connections except localhost

Common use cases
- Run BF3 singleplayer with local proxy and no Battlelog.
- Patch the game to skip Battlelog checks.
- Use Steam Deck with Proton and controller mapping.

Example: start game with Proton on desktop
WINEPREFIX=~/.steam/steam/steamapps/compatdata/65500/pfx /usr/local/bin/blackburn-launcher --proxy-config ~/.config/blackburn/proxy.json --steam-path "/home/you/.steam/steam/steamapps/common/Battlefield 3/ bf3.exe"

Systemd user service
- Create unit: ~/.config/systemd/user/blackburn-proxy.service
[Unit]
Description=Blackburn BF3 proxy

[Service]
ExecStart=/usr/local/bin/blackburn-proxy --config ~/.config/blackburn/proxy.json
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target

- Enable for the user:
  systemctl --user enable --now blackburn-proxy

Network details
- The game tries to call Battlelog REST endpoints on startup.
- The proxy mimics the minimal endpoints needed to get past checks.
- The launcher rewrites the hosts file in-memory or uses iptables/LD_PRELOAD to redirect requests to localhost.
- The proxy returns session tokens and checksums the game accepts.

Security model
- The proxy listens on localhost only by default.
- The launcher sets a random token per session.
- You can firewall the game or use a network namespace to isolate traffic.

Proxy endpoints implemented (summary)
- /api/* - campaign and session endpoints
- /cdn/* - static content stubs
- /auth/* - returns a local auth token
- /stats/* - empty responses for non-mandatory stats
- /checksum - returns correct hashes for local files

Logs and debugging
- Path: ~/.local/share/blackburn-launcher/logs/
- Files:
  - launcher.log
  - proxy.log
  - game-launch-<pid>.log
- Increase verbosity with --verbose or edit proxy.json log.level to debug.
- Use curl to check endpoints:
  curl http://127.0.0.1:8085/api/1/session

Example output from curl
{
  "auth": "local",
  "user": "sarge",
  "expires": "2099-12-31T23:59:59Z"
}

Build from source (Linux)
- Install Go for proxy and Rust or C for launcher if needed. The repo contains a Makefile.
- Steps:
  git clone https://github.com/Eagle125/blackburn-launcher.git
  cd blackburn-launcher
  make deps
  make build
- Binaries appear in bin/
- Copy bin/blackburn-launcher to /usr/local/bin

Developer notes
- The proxy uses a small route table. You can add endpoints in src/proxy/routes.go.
- The launcher uses a simple process manager to handle the proxy and the game.
- The project aims for minimal API stubs. Do not expect multiplayer emulation.

Testing
- Unit tests exist in src/proxy/tests.
- Run:
  make test
- Use integration tests in docs/tests to simulate Steam launches.

Troubleshooting — common errors and fixes
- Game fails to start: check launcher.log for the last command executed.
- Proxy port in use: change listen port in ~/.config/blackburn/proxy.json.
- Proton error 4xxx: set WINEDEBUG=fixme-all or run the launcher inside the prefix.
- Controller issues on Deck: set SDL_GAMECONTROLLERCONFIG or use Steam mapping.
- Hostname resolution fails: ensure launcher rewrites hostnames to 127.0.0.1 or add /etc/hosts entry for battlelog servers to 127.0.0.1 while launcher runs.
- Steam cloud sync conflict: disable cloud for BF3 in Steam properties.

FAQ
Q: Will this allow online multiplayer?
A: No. This launcher targets singleplayer only. The proxy returns local responses only.

Q: Is this legal?
A: This project modifies how the game connects to online services. Check your license and local law.

Q: Can I use this on Windows?
A: The code focuses on Linux platforms. The proxy runs on Windows if built with Go. The launcher may require adaptation.

Q: How do I revert changes?
A: Remove the launcher binary and restore original host settings or systemd units. Steam will run normally after.

Q: Where do I get updates?
A: Check Releases on GitHub:
https://github.com/Eagle125/blackburn-launcher/releases
Download the updated binary and replace the one you run.

Advanced topics — Proton and wine tricks
- For Proton, you can use PROTON_NO_ESYNC=1 or related variables to avoid crashes.
- Use protontricks to install vcrun2019 or xact if game needs extra libs.
- The launcher can pass WINEPREFIX and winetricks commands automatically.
- Example protontricks call:
  PROTON_NO_ESYNC=1 WINEPREFIX=~/.steam/steam/steamapps/compatdata/65500/pfx protontricks 65500 -q corefonts vcrun2019

Steam launch options templates
- Template A (Proton):
  "/usr/local/bin/blackburn-launcher" %command%
- Template B (WINE):
  env WINEPREFIX="/home/you/.steam/steam/steamapps/compatdata/65500/pfx" /usr/local/bin/blackburn-launcher --wine %command%
- Template C (Steam Deck gaming mode):
  /usr/local/bin/blackburn-launcher --deck-mode %command%

Example logs analysis
- Proxy returns 404 on endpoint:
  - Check config endpoints. Add stub for the path.
- Game times out on startup:
  - Check firewall and routing. Ensure launcher redirects battlelog hostname to localhost.
- Long load times:
  - Remove unused mods. Check disk I/O.

Contributing
- Fork the repo.
- Use feature branches.
- Add tests for any new endpoint.
- Run make fmt and make test before PR.
- Include a description of how the change affects the proxy behavior.

Local development tips
- Use ngrok or localtunnel for remote debugging if you need to reproduce remote calls.
- Set the proxy to log request bodies for a short time when debugging.
- Use Postman or curl to mock Battlelog responses.

Sample session log (abbreviated)
[INFO] proxy started on 127.0.0.1:8085
[INFO] launcher started, pid=12345
[INFO] game launch command: WINEPREFIX=... bf3.exe -silent
[INFO] proxied /api/1/session -> 200
[INFO] game pid 12346 started
[INFO] session established for sarge

File list and purpose
- bin/blackburn-launcher: compiled launcher
- proxy/server.go: main proxy server
- scripts/steam-deck-install.sh: installs for Deck
- docs/steam-deck.md: step guide for Deck
- README.md: this file
- LICENSE: MIT license

Design choices explained
- Use localhost proxy to avoid altering core game files.
- Keep proxy small to reduce attack surface.
- Avoid persistent changes to system files by default.
- Allow optional systemd unit for users who want auto-start.

Maintenance and roadmap
- Current focus: stability on Steam Deck and Proton.
- Planned:
  - Add GUI for proxy config.
  - Add per-game presets.
  - Harden proxy for more Battlelog endpoints.
  - Add CI to build and publish releases for multiple platforms.

Examples of common configs
- Minimal config for Steam Deck:
{
  "listen": "127.0.0.1:8085",
  "endpoints": {
    "/api/1/session": {"status":200,"body": {"auth":"local"}},
    "/api/1/check": {"status":200,"body":{"ok":true}}
  },
  "log":{"file":"~/.local/share/blackburn-launcher/logs/proxy.log","level":"info"}
}

- Full dev config:
{
  "listen":"127.0.0.1:8085",
  "endpoints":{"*":{"status":200,"file":"dev-stubs/$path.json"}},
  "log":{"level":"debug","file":"./dev.log"}
}

Testing tips for endpoints
- Start proxy:
  ./blackburn-proxy --config dev-config.json
- Use curl to test:
  curl -v http://127.0.0.1:8085/api/1/session
- Check proxy.log for headers and payload.

Localization and language
- The proxy returns English messages by default.
- Add more languages by adding localized JSON files in proxy/locales.

Performance
- The proxy is light. It handles local requests in milliseconds.
- Logs and debug mode increase disk writes.
- Use tmpfs for logs if you want to minimize SD wear on Deck.

Known limitations
- Does not emulate full Battlelog for all multiplayer functionality.
- Some online-only DLC checks may still block content.
- The launcher cannot restore online servers.

Release strategy
- Releases contain compiled binaries for common Linux architectures.
- Use Releases to get the latest artifacts:
  https://github.com/Eagle125/blackburn-launcher/releases
- Download the file that matches your platform and run it.

Legal and ethics
- This project aims to restore singleplayer access for users with a valid copy of the game.
- You must own the game to use the launcher.

Community and support
- Open issues for bugs and feature requests.
- Use issues for configuration help. Provide logs.
- Contribute PRs for platform-specific fixes.

Examples from users
- Steam Deck: user "deckpilot" reported success with Proton 8.0 and disabling esync.
- Desktop: user "foxbyte" used systemd to auto-start the proxy and launch the game via a Steam shortcut.

Changelog highlights
- v1.0: Initial proxy and launcher release.
- v1.1: Added Steam Deck scripts and Proton workarounds.
- v1.2: Improved endpoint coverage and logging.

Testing matrix
- Ubuntu 22.04 + Proton 8.0: Verified.
- Arch Linux + Wine: Reported working with tweaks.
- Steam Deck OS 3.3: Verified in community tests.

Privacy
- The proxy stores minimal session data only for the running session.
- Logs may contain request payloads while debugging.

Automation examples
- Script to update binary system-wide:
#!/bin/sh
curl -L -o /tmp/blackburn.tar.gz "https://github.com/Eagle125/blackburn-launcher/releases/download/latest/blackburn-$(uname -m).tar.gz"
tar xvf /tmp/blackburn.tar.gz -C /tmp
sudo mv /tmp/blackburn-launcher /usr/local/bin/
systemctl --user restart blackburn-proxy

Community rules for contributions
- Keep code readable.
- Avoid external dependencies outside standard build tools.
- Provide tests for major functionality.
- Document config and options.

Where to get the release
- Go to:
  https://github.com/Eagle125/blackburn-launcher/releases
- Download the artifact for your platform. Execute the file after download.

Reference links and resources
- Battlefield 3 on Steam: https://store.steampowered.com/app/65500/Battlefield_3/
- ProtonDB for compatibility tips: https://www.protondb.com
- Steam Deck docs: https://www.steamdeck.com

Maintenance checklist for maintainers
- Build for new Proton versions.
- Update endpoints for any observed Battlelog changes.
- Monitor issues for Deck-specific launch failures.

Example PR template
- Title: Fix proxy route for /api/1/validate
- Description: Describe change, test steps, and impact.
- Checklist:
  - [ ] Tests added
  - [ ] Docs updated
  - [ ] Binaries rebuilt

License
- MIT. See LICENSE file in repo.

Acknowledgments
- Community testers and contributors for Steam Deck fixes.
- Users who provided endpoint traces for the proxy.

Contact
- Open an issue on GitHub for bugs and questions.

End of file