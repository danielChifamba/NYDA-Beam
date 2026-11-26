# NYDA Beam user guide

How to send files from one laptop to another with no internet and no flash drive, and how to connect the two laptops.

The same guide is built into the app: press the **Help** button. In a terminal, run `beam guide`.

**Contents**

1. [The idea in one minute](#1-the-idea-in-one-minute)
2. [Getting NYDA Beam](#2-getting-nyda-beam)
3. [Sending files, step by step](#3-sending-files-step-by-step)
4. [Connecting the two laptops](#4-connecting-the-two-laptops)
5. [Receiving, and sending files back](#5-receiving-and-sending-files-back)
6. [Keeping it safe](#6-keeping-it-safe)
7. [Command line](#7-command-line)
8. [Troubleshooting](#8-troubleshooting)
9. [Questions people ask](#9-questions-people-ask)

---

## 1. The idea in one minute

- One laptop, the **sender**, runs NYDA Beam and shows an **address** and a **6-digit code**.
- The other laptop, the **receiver**, opens that address in a web browser and types the code.
- Files travel straight between the two laptops. Nothing goes to the internet.

The receiver needs **no app, no account and no internet**. Only the two laptops need to be on the same network, and that network can be as simple as one cable.

Both laptops can run NYDA Beam too. Then each one sees the other automatically and you can send in either direction.

## 2. Getting NYDA Beam

Pick one way.

**A. The Windows app (`NYDA-Beam.exe`)** is the easiest. Copy the file to the laptop and double-click it. No Python needed. To make the file yourself, run `build_exe.bat` once on a laptop that has Python and internet. It creates `dist\NYDA-Beam.exe` and a ready-to-share folder `release\NYDA-Beam`. For terminal commands, run the optional `build_cli.bat` to also get `NYDA-Beam-CLI.exe`.

**B. From the source folder** needs Python 3.9 or newer.

```text
python beam.py
```

On Windows you can double-click `start-beam.bat` instead. For a native window rather than a browser tab, run `pip install pywebview` once.

**C. Encrypted mode (optional)** needs the `cryptography` package (`pip install cryptography`) or OpenSSL on the sender's laptop. `build_exe.bat` installs it before building, so the `.exe` can use encrypted mode.

> **Windows firewall, first time only.** Windows asks whether to allow the app on the network. Tick **Private networks** and press **Allow access**. If you pressed Cancel, open *Windows Security, Firewall and network protection, Allow an app through firewall*, find NYDA Beam (or Python) and tick **Private**. Make sure the network is marked *Private*, not *Public*.

## 3. Sending files, step by step

1. **Connect the laptops** to the same network (see section 4). Do this first so Beam can find an address.
2. **Start NYDA Beam** on the laptop that has the files.
3. Press **Add files** or **Add folder** and choose what to send. In the app window you can also drag files and folders onto it. In a browser tab, or any time, you can paste a path under *Paste a path instead*.
4. Look at **On the other laptop**. It shows one or more addresses such as `192.168.1.5:8765`, and the **pairing code**.
5. On the **other laptop**, open a web browser and type the address. Press Enter.
6. Type the **6-digit code**. It connects by itself when you enter the sixth digit.
7. Press the download button next to a file, or **Download all** for one zip with everything.

**Shortcut:** press **Copy link** in the sender's window. That copies an address with the code already inside. Send it to the other laptop by any means, and opening it connects straight away. Share it only with the person you mean.

The sender's window shows each transfer with live speed and progress.

### The address, explained

If you see more than one address, that is normal: a laptop often has several network adapters. NYDA Beam sorts them **best first** and labels each one. Use the first, or the one that matches how the other device is connected.

| Label | Looks like | Means |
|---|---|---|
| **Phone hotspot** | `192.168.43.x` | The laptop joined a phone's hotspot. A phone or laptop on that hotspot uses this one |
| **This laptop's hotspot** | `192.168.137.x` | The laptop is sharing its own hotspot (Windows "Mobile hotspot") |
| **Wi-Fi**, **Ethernet**, **Network** | `192.168.x.x` or `10.x.x.x` | Wi-Fi or router |
| **Direct cable** | `169.254.x.x` | A cable straight between the two laptops |
| **Virtual adapter**, **VPN** | `192.168.56.x`, `172.x.x.x` | VirtualBox, WSL, Docker or a VPN. **Only this laptop can reach these.** They are tucked under *other addresses* |

When a device connects, the address it used is tagged **Worked for a device**.

If encrypted mode is on, the address starts with `https://` and you must type that part.

## 4. Connecting the two laptops

Both laptops need to be on the same network. **The network does not need internet.** Choose whichever you have.

| Option | Speed | Best for |
|---|---|---|
| **Ethernet cable** between the laptops | about 110 MB/s on gigabit | Very big files. Fastest and steadiest. |
| **Same Wi-Fi or router** | 20 to 100+ MB/s | Quick, no cable. The router can be offline. |
| **Phone hotspot** | 20 to 100 MB/s | When there is no router around. |
| **USB-to-Ethernet adapter** | like a cable | A laptop with no network port. |

### Option 1: Ethernet cable (fastest)

1. Turn **Wi-Fi off** on both laptops so traffic can't take another route.
2. Plug **one normal network cable** into both laptops. You don't need a special crossover cable.
3. **Wait up to a minute.** Windows gives both laptops an address on its own.
4. Start NYDA Beam on the sender. An address tagged **Direct cable** appears. Type it into the other laptop's browser.

Until the link is up, the sender shows *No network found yet*. It updates by itself, and **Refresh network** forces a new look.

**If no cable address appears after two minutes**, set the addresses by hand.

*Windows.* Do these on **both** laptops, in a **Command Prompt run as Administrator** (right-click it, *Run as administrator*; without that you get "requires elevation").

1. Find your adapter name. It is the text after `Ethernet adapter`, and may be `Ethernet 2` rather than `Ethernet`:

   ```text
   ipconfig
   ```

2. Give the laptops fixed addresses. On the **sending** laptop:

   ```text
   netsh interface ip set address name="Ethernet" static 192.168.50.1 255.255.255.0
   ```

   On the **receiving** laptop:

   ```text
   netsh interface ip set address name="Ethernet" static 192.168.50.2 255.255.255.0
   ```

3. **Mark the cable network Private** (both laptops). A bare cable is an "Unidentified network", which Windows treats as Public, and its firewall then blocks NYDA Beam. This is the most common reason a hand-set cable link "does nothing":

   ```text
   powershell -Command "Set-NetConnectionProfile -InterfaceAlias 'Ethernet' -NetworkCategory Private"
   ```

4. Test the link from the receiving laptop: `ping 192.168.50.1`. Replies mean the cable is fine.
5. In NYDA Beam press **Refresh network**, then open `http://192.168.50.1:8765` on the receiving laptop.
6. When you're done, put both back to normal:

   ```text
   netsh interface ip set address name="Ethernet" dhcp
   ```

`beam cable` prints these steps with your real adapter name filled in. If it still won't work, use a **phone hotspot** instead: there are no addresses to set at all.

*macOS.* In Terminal, find the service name with `networksetup -listallnetworkservices`, then:

```text
sudo networksetup -setmanual "USB 10/100/1000 LAN" 192.168.50.1 255.255.255.0
```

Use `.2` on the other laptop. To undo: `sudo networksetup -setdhcp "USB 10/100/1000 LAN"`.

*Linux.* Find the interface with `ip link`, then:

```text
sudo ip addr add 192.168.50.1/24 dev eth0
```

Use `.2` on the other laptop. To undo: `sudo ip addr del 192.168.50.1/24 dev eth0`.

### Option 2: Same Wi-Fi or router

Join both laptops to the same Wi-Fi network, or plug both into the same router. The router **does not need internet**, only to be switched on. Start NYDA Beam and use the **Wi-Fi** or **Ethernet** address.

Some public, hotel or school Wi-Fi stops devices from seeing each other ("client isolation"). If the other laptop can't reach the sender, use a cable or a phone hotspot instead.

### Option 3: Phone hotspot

Turn on the hotspot on a phone. Join it from both laptops like any Wi-Fi network. Transfers between the two laptops stay on the hotspot and **use no mobile data**. Then use the address labelled **Phone hotspot** (it starts with `192.168.43.`). A phone can join the same hotspot and open that address too. Addresses labelled *Virtual adapter* won't work from a phone.

### Option 4: USB-to-Ethernet adapter

A cheap adapter adds a network port to a laptop that has none. Plug it in, then follow Option 1.

## 5. Receiving, and sending files back

**On the receiving laptop**, the page lists every shared file with its size.

- Press the **download button** on a file. Your browser saves it like any download.
- Press **Download all** for a single zip. It is streamed, so it starts at once even for huge folders.
- Press **Show checksum** to see the file's SHA-256 fingerprint. Compare it with the same file on the sender to prove it arrived intact.
- If a download is interrupted, **resume it from your browser's downloads list**. It continues from where it stopped.

**Sending files the other way.** Under *Send files back*, drop files or folders on the page, or use **Choose files** and **Choose folder**. They are saved on the sender's laptop in the folder shown under *Received from the other laptop* (by default `Downloads\NYDA Beam`). Press **Open folder** to see them.

- A file that already exists is never overwritten. The new one is saved as `name (2).ext`.
- Folders keep their structure.
- The sender can turn this off with **Let the other laptop send files here**.

**Both laptops running NYDA Beam?** Each lists the other in the **Devices** tab. Press **Open** to go straight to it, with no address to type. In a terminal, `beam find` lists them.

## 6. Keeping it safe

NYDA Beam is for two laptops you control. These tools are in the **Security** panel of the sender's window.

- **The pairing code is the key.** Nothing is shared without it. After 5 wrong tries a device is locked out for 30 seconds, then longer. After 30 wrong guesses in total the code changes itself. **New code** makes a fresh code and disconnects everyone.
- **Ask me before a new device connects.** Even with the right code, a new device waits on a *Waiting for approval* screen until you press **Allow**. The sender's window shows who is asking. Use this in a shared room or on shared Wi-Fi.
- **Pause new connections.** Once your laptop is connected, switch this on so nobody else can join. Devices already connected keep working.
- **Disconnect.** Every connected device has a **Disconnect** button.
- **Encrypted connection (HTTPS).** This scrambles everything on the cable or Wi-Fi, so nobody listening can read your files. See below.
- **Recent activity** lists connections, refusals, wrong codes and lockouts with the time.

### Using encrypted mode

1. On the sender, switch on **Encrypted connection (HTTPS)**. The address now starts with `https://`. (Or start with `beam --secure`.)
2. On the other laptop, open that `https://` address.
3. The browser warns that the connection isn't private. **That is expected.** The certificate was made by NYDA Beam on the sender's laptop, and with no internet there is nobody else to vouch for it.
4. Compare the **fingerprint** shown in the sender's Security tab with the one in the browser's certificate details (usually under the padlock or warning, *Certificate*, *Details*, *SHA-256*). If they match, choose **Advanced, then Continue**.

Without encryption, everything is sent in the clear. That is fine on your own cable or your own hotspot, but **don't use it on public Wi-Fi.**

More in [../SECURITY.md](../SECURITY.md).

## 7. Command line

Everything the window does is also available in a terminal. In the examples, `beam` stands for `python beam.py`, or `NYDA-Beam-CLI.exe` if you built the optional terminal version.

```text
beam                         open the app
beam movie.mp4 "My Folder"   share these straight away
beam find                    list other laptops running NYDA Beam
beam guide                   quick guide in the terminal
beam cable                   manual steps for a direct cable link
```

| Option | What it does |
|---|---|
| `--secure` / `--no-secure` | Encrypted (HTTPS) connection on or off |
| `--approve` / `--no-approve` | Ask before each new device may connect |
| `--code 123456` | Choose your own 6-digit code |
| `--name "My laptop"` | Name shown to the other laptop |
| `--port 9000` | First port to try. If it's busy, the next free one is used and shown. |
| `--inbox D:\Received` | Where files sent back are saved |
| `--no-upload` | Don't accept files from the other laptop |
| `--no-gui` | Terminal only. Beam prints the addresses, the code and the control-panel link. |
| `--no-discovery` | Don't announce this laptop on the network |

The choices you make in the window for **Encrypted** and **Ask before connecting** are remembered in `~/.nydabeam/config.json` (on Windows, `C:\Users\<you>\.nydabeam`). A command-line option overrides them for that run.

Example for sending a big folder over a cable, encrypted, with approval:

```text
beam --secure --approve "D:\Videos\Wedding"
```

## 8. Troubleshooting

**The other laptop can't open the address.**
- Are both on the same network? Check the cable is in, or both are on the same Wi-Fi.
- Did you allow NYDA Beam through the firewall on the **Private** network? See section 2.
- Type the address exactly, with the port, like `192.168.1.5:8765`. In encrypted mode it starts with `https://`.
- If several addresses are shown, try the others. Cable addresses start with `169.254.`.
- Turn off any **VPN** on either laptop.
- Public, hotel and school Wi-Fi often blocks devices from seeing each other. Use a cable or hotspot.

**The cable doesn't show a "Direct cable" address.**
Wait a full minute and check both laptops have the cable in firmly. Turn off Wi-Fi on both. If it still doesn't appear, set the addresses by hand (section 4, option 1).

**"That code isn't right" or "Too many tries".**
Read the code again in the sender's window. After 5 wrong tries the device waits 30 seconds, then longer. If the sender pressed **New code**, the old one stops working.

**"Waiting for approval".**
The sender has *Ask me before a new device connects* switched on. They need to press **Allow** in their window.

**"The sender has paused new connections".**
The sender has *Pause new connections* switched on. Ask them to switch it off.

**"You were disconnected".**
The sender pressed **Disconnect** or **New code**, or the session timed out. Enter the code again.

**Transfers are slow.**
- Wi-Fi speed depends on distance and other devices. A cable is faster and steadier.
- An older 100 Mbit network port tops out near 11 MB/s. Gigabit ports reach about 110 MB/s.
- Encrypted mode uses some processor power. On an older laptop, switch it off if you trust the network.
- Thousands of tiny files are slower than a few big ones. Use **Download all**.

**The browser warns about the connection.**
That happens only in encrypted mode and is expected. See *Using encrypted mode* above.

**Windows says "unknown publisher".**
A program you built yourself isn't code-signed. Choose **More info, then Run anyway**, but only for a copy you built or got from someone you trust.

**"Another program already uses port 8765".**
Beam picks the next free port by itself and shows it in the address. You can also choose one with `--port`.

**Files from the other laptop aren't where I expect.**
Look at the path under *Received from the other laptop*, and press **Open folder**.

## 9. Questions people ask

**Does it need internet?** No. Nothing is sent to the internet, and nothing needs to be downloaded to use it.

**Does the other laptop need to install anything?** No, only a web browser. Any modern browser works, including on a phone.

**How big can the files be?** There is no limit set by Beam. Files are streamed in small pieces, so memory use stays low. The limit is the free space on the receiving laptop.

**Can I send to a phone?** Yes. A phone can be the receiver if it is on the same network, such as the same Wi-Fi or hotspot.

**Can two people receive at once?** Yes. Each device enters the code, and each gets its own progress. With *Ask me before a new device connects* on, you approve each one.

**Does it work between Windows, Mac and Linux?** Yes. The receiver is just a browser.

**Is it safe on my home Wi-Fi?** The code, lockouts and Allow prompt protect against other people using it. Without encrypted mode, someone able to watch the Wi-Fi could read the files in transit. Turn on encrypted mode for anything private.

**Where are my settings and certificate kept?** In `~/.nydabeam` (`C:\Users\<you>\.nydabeam` on Windows). Delete that folder to reset them.

**How do I stop it?** Close the window, or press Ctrl+C in the terminal. Nothing keeps running afterwards.
