# Paper Ghost

**Holmes CTF 2026 · Sherlock 04 · Windows forensics**

A triage-led reconstruction of a USB-delivered spyware incident using Registry, UserAssist, ConsentStore, SRUM, and Windows Search evidence.

[Download the practice pack](https://github.com/PaulChola/holmes-ctf-2026-writeups/releases/download/practice-packs-2026/PaperGhost.zip) · [All challenge files](README.md)

> The recovered account password is redacted in this public writeup.

---

## Scenario

Clara Voss's workstation, `CO-LT-0469`, was compromised through a planted Lexar USB device. The device carried a fake update package that launched spyware associated with VON BORK. The available triage collection contained Windows Registry hives, LNK and Jump List artifacts, SRUM, and the Windows Search ESE database.

## Evidence Sources

- `Windows/System32/config/SYSTEM`: USB enumeration, device serial, and device name.
- `Users/cvoss/NTUSER.DAT`: UserAssist execution history.
- `Users/cvoss/AppData/Local/Microsoft/Windows/UsrClass.dat`: application-consent activity.
- `Windows/System32/SRU/SRUDB.dat`: application and network usage.
- `ProgramData/Microsoft/search/data/applications/windows/Windows.edb`: residual indexed DIOGENES documents.
- Recent `.lnk` files: document paths and access metadata.

## Analysis

### 1. USB identification and first connection

The SYSTEM hive contained the device instance:

```text
USBSTOR\\Disk&Ven_Lexar&Prod_USB_Flash_Drive&Rev_2.00\\RS200000000627E4&0
```

The corresponding device record showed:

```text
FriendlyName: CO-USB-0091
DeviceDesc: USB Flash Drive
```

The earliest relevant device-installation timestamp was `2026-08-19 15:35:50`.

### 2. Malicious package execution

UserAssist contained the decoded executable entry:

```text
E:\CO-LT-0469 update package\update.exe
```

The UserAssist binary record included a FILETIME value. Decoding the final eight bytes as a Windows FILETIME produced:

```text
2026-08-19 15:36:25
```

This matches the removable-media execution sequence and identifies the payload path.

### 3. Audio and webcam surveillance

The UsrClass/ConsentStore evidence showed the spyware accessing the workstation sensors:

```text
Microphone start: 2026-08-19 15:38:08
Webcam start:     2026-08-19 15:42:28
Webcam duration:  126.9805499 seconds
```

The challenge validator expects the webcam duration rounded to a whole number: `127` seconds.

### 4. C2 traffic

SRUM application-network accounting attributed the outbound transfer to the malicious executable. The recorded sent-byte count was:

```text
172,064,531 bytes
```

Using decimal megabytes:

```text
172064531 / 1000000 = 172.064531 MB
```

### 5. DIOGENES credentials

Although the original DIOGENES documents were absent, Windows Search retained their indexed content in `Windows.edb`. Recovering the relevant ESENT long-value content exposed the record for Tom Ainsworth, a Software Developer supporting DIOGENES Ticketing. The record contained:

```text
tainsworth:[REDACTED]
```

## Final Answers

| # | Question | Answer |
|---:|---|---|
| 1 | First USB connection | `2026-08-19 15:35:50` |
| 2 | USB serial number | `RS200000000627E4&0` |
| 3 | Payload path | `E:\CO-LT-0469 update package\update.exe` |
| 4 | Malicious package execution | `2026-08-19 15:36:25` |
| 5 | USB asset/device name | `CO-USB-0091` |
| 6 | Microphone capture start | `2026-08-19 15:38:08` |
| 7 | Webcam stream duration | `127` |
| 8 | Decimal outbound megabytes | `172.064531` |
| 9 | Developer credentials | `tainsworth:[REDACTED]` |
