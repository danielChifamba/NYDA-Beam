# Security

## Reporting a vulnerability

Please **don't open a public issue** for a security problem. Use GitHub's private reporting instead: open the repository's **Security** tab, then **Report a vulnerability**. Include what you found, how to reproduce it and the version (`beam --version`). You'll get a reply as soon as I can manage it, and a credit in the changelog if you'd like one.

Supported versions: the latest 1.x release.

## What NYDA Beam is for

Moving files between **two machines you control**, in the same room, over a link you control: a cable, your own router, or your own phone hotspot. The design goal is that other devices on that network can't read your files, put files on your laptop, or take over the app.

**In scope**
- Another device on the same network trying to download, upload or control the app without permission.
- A malicious web page open in either laptop's browser trying to talk to the app.
- A hostile file name or folder structure sent from the other laptop.

**Out of scope**
- A laptop that is already compromised (malware, another local user, physical access).
- Flaws in the operating system, browser or Python itself.
- An attacker who can read or change network traffic while the app is in **plain (non-encrypted) mode**. This is documented below; use encrypted mode in that situation.

## How it protects you

| Threat | Control |
|---|---|
| Strangers on the network fetching files | Every file, zip, checksum and upload endpoint needs a session that starts with the 6-digit code. |
| Guessing the code | 5 wrong tries lock that address out for 30 seconds, doubling each time up to 5 minutes. After 30 wrong guesses in total, from any addresses, the code changes itself. Codes are compared in constant time. |
| A right guess from the wrong person | Optional **Allow prompt**: a device with the code still waits for you to press Allow, and the window shows what is asking. |
| A device you no longer want | **Disconnect** removes it at once. **Pause new connections** closes the door. **New code** revokes every session. |
| Stolen session cookie | Sessions are tied to the address they were created from, expire after 4 hours idle or 24 hours, and the cookie is `HttpOnly`, `SameSite=Strict` (and `Secure` in encrypted mode). |
| Someone controlling the app from the network | The control window is a **separate server bound to 127.0.0.1**. The network-facing server has no control routes. The control server also requires a per-run secret and a loopback `Host` header, which stops DNS-rebinding. |
| A web page forging requests | Cross-origin `POST`s are refused (`Origin` must match `Host`). Pages are served with a strict `Content-Security-Policy`, `X-Frame-Options: DENY` and `nosniff`. |
| Writing outside the receive folder | Uploaded names are sanitised (no `..`, drive letters, reserved Windows names, bad characters), the final path is checked to stay inside the folder, and existing files are never overwritten. |
| Half-sent files | Uploads go to a temporary file that is deleted if the transfer stops. |
| Eavesdropping | **Encrypted mode** serves everything over HTTPS (TLS 1.2 or newer) with a certificate made on the sender's laptop. |
| Cleartext leftovers | Sessions are cleared whenever encrypted mode is switched on or off. Nothing is cached (`Cache-Control: no-store`). |

The private key for encrypted mode is stored in `~/.nydabeam/beam-key.pem`, readable only by you on Linux and macOS. On Windows it relies on the permissions of your user profile folder.

## Known limitations

Please read these before relying on Beam for anything sensitive.

1. **Plain mode is not encrypted.** Anyone who can watch the traffic can read the files, the code (it is sent when you connect) and the session cookie. That is acceptable on a cable between two laptops or on your own hotspot. It is **not** acceptable on public or shared Wi-Fi. Turn on encrypted mode there.
2. **The certificate is self-made, so trust is on first use.** Browsers warn about it. Compare the fingerprint in the sender's Security panel with the browser's certificate details before you continue. Beam does not pin it for you yet.
3. **In plain mode the receiver can't prove it reached the real sender.** A rogue device could pretend to be the sender. Encrypted mode with a fingerprint check solves this.
4. **The code has six digits.** That is a deliberate trade-off for ease of use. The lockouts, automatic rotation and the Allow prompt exist to make guessing impractical, but an attacker with many network addresses is slowed rather than stopped. Use the Allow prompt on networks with people you don't know.
5. **One code for everyone.** Every device that enters the code gets the same access to the shared files.
6. **Sessions are tied to an address.** Devices behind the same NAT share an address and so share that trust boundary.
7. **Discovery is a broadcast.** Unless you pass `--no-discovery`, the laptop announces its name, port, file count and total size to the local network every couple of seconds. It never announces file names or the code.
8. **No malware scanning.** Received files are saved as they arrive. Treat them like any download.
9. **Little rate limiting beyond pairing.** A device that has the code could open many connections at once.

## Hardening tips

- On any network you don't fully trust: `beam --secure --approve`, then **Pause new connections** once your laptop is connected.
- Share only what you mean to. Remove files from the list when you're done.
- Press **New code** between sessions.
- Use `--no-discovery` if you don't want the laptop to announce itself.
- Close the app when you're finished. Nothing keeps running afterwards.
