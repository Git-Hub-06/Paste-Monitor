# Paste Monitor

A Chrome extension that intercepts clipboard paste events and automatically masks sensitive data before it reaches the page — with an instant in-page alert and a local paste history log.

Built as a lightweight browser-native DLP (Data Loss Prevention) tool for security-conscious users and practitioners working across multiple environments.

---

## Why This Exists

In IT environments, analysts copy credentials, API keys, server IPs, and connection strings between tools. One accidental paste into a ticket, a web form, or a chat window is all it takes for sensitive data to be exposed.

Paste Monitor catches that before it happens.

---

## How It Works

1. A content script listens for paste events at the **capture phase** (runs before any page script)
2. The pasted text is scanned against a pattern registry
3. If a match is found:
   - The original paste is **blocked** (`preventDefault`)
   - A **masked version** is inserted in its place
   - An **in-page alert banner** appears identifying what was detected
4. Every paste (masked or clean) is logged locally to `chrome.storage.local` and visible in the popup

**No data leaves the browser. There are no external requests. Zero data egress.**

---

## Detection Patterns

| Category | Examples Detected | Masked As |
|---|---|---|
| OpenAI API Key | `sk-abc123...` | `sk-****` |
| AWS Access Key | `AKIAIOSFODNN7...` | `AWS-****` |
| GitHub PAT | `ghp_xxxxxxxxxxxx` | `ghp_****` |
| Google API Key | `AIzaSyXXXXXXXXX` | `AIza-****` |
| Bearer Token | `Bearer eyJhbGci...` | `Bearer ****` |
| Email Address | `user@domain.com` | `****@****.***` |
| Username | `username=admin`, `user: root` | `username=****` |
| Password | `password=secret`, `pwd=abc123` | `password=****` |
| Credential String | `admin:P@ss@db.internal` | `****:****@****` |
| IPv4 Address | `192.168.1.100`, `10.0.0.1` | `[IP-HIDDEN]` |
| IPv6 Address | `fe80::1`, `2001:db8::ff00` | `[IPv6-HIDDEN]` |
| Internal FQDN | `dc01.corp.local`, `app.internal` | `[HOST-HIDDEN]` |
| Bare Hostname | `WKSTN-01`, `SRV-DC01` | `[HOST-HIDDEN]` |

---

## Popup Features

- **Stats bar** — total pastes, masked count, clean count
- **Per-entry detail** — masked text, source URL, timestamp, detection type tags
- **Clear history** button
- All data stored locally in `chrome.storage.local`

---

## Installation (Unpacked)

> Chrome Web Store submission coming. For now, load as an unpacked extension.

1. Clone or download this repository
2. Open Chrome and go to `chrome://extensions`
3. Enable **Developer Mode** (top right toggle)
4. Click **Load unpacked**
5. Select the folder containing `manifest.json`

The extension icon will appear in your toolbar. Pin it to keep the popup accessible.

---

## File Structure

```
paste-monitor-pro/
├── manifest.json      # MV3 manifest — permissions, content script config
├── content.js         # Paste interceptor — pattern matching, masking, alert banner
├── background.js      # Service worker — stores paste events to local history
├── popup.html         # Extension popup UI
└── popup.js           # Popup logic — renders history, stats, clear button
```

---

## Permissions Used

| Permission | Reason |
|---|---|
| `storage` | Stores paste history locally via `chrome.storage.local` |
| `host_permissions: <all_urls>` | Allows content script to run on all pages |

No `clipboardRead`, no network requests, no remote code.

---

## Extending the Pattern Library

Both `content.js` and `popup.js` share an identical `SECRET_PATTERNS` array at the top of each file. To add a new pattern:

```js
{
    regex: /your-pattern-here/g,
    mask: "REPLACEMENT_TEXT",
    label: "Display Label in Popup"
}
```

Add it to both files to keep masking consistent between interception and history display.

---

## Changelog

### v2.0
- Added 8 new detection patterns: Email, Username, Password, Credential String, IPv4, IPv6, Internal FQDN, Bare Hostname
- UI rethemed to white/professional light design with dark navy header
- Stats bar redesigned as column layout with large bold numbers
- Detection type tags shown per history entry
- In-page alert restyled to white card with red left-border stripe

### v1.1
- Fixed: `insertText()` was undefined — masked paste silently failed
- Fixed: `sendMessage()` missing — paste history was always empty
- Fixed: `innerHTML` replaced with `textContent` — eliminated XSS vector
- Fixed: unified regex patterns between `content.js` and `popup.js`
- Fixed: event listener changed to `capture: true`
- Added: GitHub PAT, Google API Key, Bearer Token patterns
- Added: timestamps, detection labels, masked flag per history entry
- Added: Clear History button, stats bar, popup entry detail

### v1.0
- Initial release with basic OpenAI, AWS, GitHub key detection
- Known issues: `insertText()` undefined, history never populated, XSS via `innerHTML`

---

## License

MIT — free to use, fork, and extend.

---

## Author

**Githendran Prakash**  
Lead Threat Responder — ThirdLine Advisory  
[LinkedIn](https://linkedin.com/in/) · [GitHub](https://github.com/)
