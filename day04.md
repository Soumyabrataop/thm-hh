# TryHackMe — Packed Light Writeup

> **Author:** **Soumyabrata Mukherjee**  
> TryHackMe Profile: https://tryhackme.com/p/soumyabratamukherjee64

---

## Acknowledgements

This writeup has been prepared by **Soumyabrata Mukherjee**.

Special thanks to the **TryHackMe Discord community** (https://discord.gg/tryhackme) and everyone who helped with discussions and hints throughout the challenge.

---

# Challenge Information

| Field | Details |
|-------|---------|
| **Room Name** | Packed Light |
| **Category** | Forensics |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |

### Description

> *Tiny packets. Odd hours. Suspiciously regular. Someone's smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.*

---

# Previous Writeups

- **Publisher Room**  
  https://app.notion.com/p/TryHackMe-Publisher-Room-2d4c8aaddc7380ff94bbf03a910ae404?source=copy_link

- **Advent of Cyber 2025 – Side Quest 3**  
  https://app.notion.com/p/TryHackMe-Side-Quest-3-Solution-Advent-of-Cyber-2025-2d3c8aaddc738032ac42f4f2f9b29407?source=copy_link

---

# Task 1

## Overview

![Task 1](https://cdn-images.tryhackme.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1784111354645.png)

This task only requires clicking the **Completed** button.

---

# Task 2

## Objective

We are provided with a packet capture (`traffic.pcapng`) and need to determine how data is being exfiltrated to recover the flag.

> **Note:** I haven't started the TryHackMe Forensics module yet, so I took some guidance from Gemini while solving this room.

---

## Inspecting the Traffic

After opening `traffic.pcapng` in Wireshark, I followed the HTTP stream.

One HTTP response contained a Python script named:

```
updates.py
```

The script was being downloaded from the server.

After reviewing the source code, it became clear that it was a simple **keylogger**.

---

## Understanding the Malware

The interesting part of the script is shown below.

```python
def getkey():
    p1 = "H0t3lSt@ff0Nly"
    p2 = "K3epS3cr3t!"
    return p1 + p2
```

The encryption routine is:

```python
def xor(data: bytes, key: bytes) -> bytes:
    return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))
```

Every key pressed by the victim is processed using:

```python
sendltr(character)
```

The character is:

1. XOR encrypted
2. Base64 encoded
3. Sent inside the HTTP Cookie header

```python
Cookie: hotel_sess_state=<Base64 Data>
```

---

## Why Only the First Character of the Key?

At first glance, it appears that the XOR key is:

```
H0t3lSt@ff0NlyK3epS3cr3t!
```

However, the malware encrypts **one character at a time**.

Suppose the victim presses:

```
T
```

The data length is only **one byte**.

During encryption:

```
index = 0
key[0 % len(key)] = key[0] = H
```

Therefore:

```
'T' XOR 'H'

84 XOR 72 = 28
```

The encrypted byte becomes:

```
0x1C
```

Encoding that single byte using Base64 gives:

```python
import base64

print(base64.b64encode(bytes([28])))

# HA==
```

Since every HTTP request contains only **one encrypted character**, the XOR routine always uses the first character of the key (`H`).

The remainder of the key is never used.

---

## Extracting the Cookies

Next, I extracted every Cookie value from the packet capture.

```bash
strings traffic.pcapng | grep -oP 'hotel_sess_state=\K[^;\s\r\n]+' > raw_cookies.txt
```

The extracted file looked similar to:

```text
HA==
AA==
BQ==
Mw==
Hg==
ew==
Og==
...
```

Each Base64 string represents one encrypted keystroke.

---

## Decoding the Data

I used **GCHQ CyberChef**.

Pipeline:

```
From Base64
↓

XOR
Key: H
```

Because every character is encrypted using only the first byte of the XOR key, decrypting with **H** immediately reconstructs the original keystrokes.

The recovered text contains the flag.

```
THM{REDACTED}
```

Submitting the flag completes the room.

---

# Methodology Summary

1. Open the packet capture in Wireshark.
2. Follow the HTTP stream.
3. Recover the downloaded Python keylogger.
4. Understand the XOR encryption routine.
5. Notice that only one character is encrypted per request.
6. Determine that only the first key byte (`H`) is used.
7. Extract all `hotel_sess_state` cookie values.
8. Decode Base64.
9. XOR using `H`.
10. Recover the flag.

---

# Key Takeaways

- Following HTTP streams often reveals downloaded malware or scripts.
- Reading source code is frequently easier than reversing encrypted traffic directly.
- Base64 is an encoding scheme, not encryption.
- Understanding how XOR is implemented is more important than simply knowing XOR itself.
- When encryption is applied one byte at a time, only the corresponding key byte is used.

---

# Conclusion

**Packed Light** is an enjoyable beginner-friendly forensics challenge that combines packet analysis with basic malware analysis. Rather than relying on complex reverse engineering, the solution comes from understanding how the downloaded Python keylogger encrypts each keystroke before transmitting it. By recognizing that each HTTP request contains only a single encrypted character, it becomes clear that only the first byte of the XOR key is ever used. Extracting the cookie values, decoding them from Base64, and XORing with the correct key quickly reveals the original keystrokes and ultimately the flag.

---

**Author:** Soumyabrata Mukherjee  
**Platform:** TryHackMe  
**Category:** Forensics
