# Threat model

## What this project does and where untrusted input enters
- A Flutter client (Windows, macOS, Linux, Android, iOS) for the Obsidian VPN. It connects to an Obsidian server and can install that server on the user's VPS over SSH. The Go core lives in `core/` (a submodule of the public ObsidianVPN repo).
- `server/keyserver.py` is installed on the VPS and serves keys over the network; its input is untrusted.
- Untrusted input to the app: connection URIs/links and QR codes, server responses, subscription/config data, and remote version/update metadata.

## Components that matter most / least
- Most: `server/keyserver.py`, `app/lib/` (VPS setup over SSH, URI/config parsing, storage of keys and credentials, update logic), native platform code for the tunnel (`app/android`, `app/ios`, `app/macos`, `app/linux`, `app/windows`).
- Less: UI code, assets, `screenshots/`, `app-docs/`, `scripts/` (build helpers).
- The Go core itself is scanned separately as the ObsidianVPN project.

## How to exercise it
- `cd app && flutter test` runs the unit tests; `flutter analyze` runs static analysis. Full application builds happen only in CI.
- `python3 server/keyserver.py` can be run directly.

## How you rate severity
- Remote unauthenticated access to keys or credentials via keyserver, command injection into SSH setup commands, privilege escalation through the root helper (`pkexec`), insecure storage or leakage of secrets: high or critical.
- Malicious link/QR/config causing unintended connection settings or code execution: high.
- Pure resource-exhaustion DoS: medium.

## Anything to leave alone
- Findings that need a rooted/jailbroken device or local root access.
