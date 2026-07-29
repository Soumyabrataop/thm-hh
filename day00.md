# TryHackMe — The Brochure Writeup

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
| **Room Name** | The Brochure |
| **Category** | OSINT |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |

### Description

> *The brochure's hero photo has an AI fingerprint. Follow the account that posted it, and the trail doesn't end at the hotel; it ends at someone the hotel never mentioned.*

---

# Previous Writeups

- **Publisher Room**  
  https://app.notion.com/p/TryHackMe-Publisher-Room-2d4c8aaddc7380ff94bbf03a910ae404?source=copy_link

- **Advent of Cyber 2025 - Side Quest 3**  
  https://app.notion.com/p/TryHackMe-Side-Quest-3-Solution-Advent-of-Cyber-2025-2d3c8aaddc738032ac42f4f2f9b29407?source=copy_link

---

# Task 1

## Overview

![Task 1](https://cdn-images.tryhackme.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1784111354645.png)

This task only requires clicking the **Completed** button to proceed.

---

# Task 2

## Objective

An attachment is provided with the room. The downloaded ZIP archive contains a single image:

```
![The Brochure](/assets/thebrochure.png)
```

---

## Initial Analysis

Since the challenge description hinted that the brochure image contained an AI fingerprint, the first assumption was that hidden metadata or steganographic content might be present.

To investigate this possibility, I analyzed the image using **Aperi'Solve**.

### About Aperi'Solve

Aperi'Solve is an online forensic platform that automates several image analysis techniques by running tools such as:

- zsteg
- steghide
- outguess
- exiftool
- binwalk
- foremost
- strings

This makes it useful for detecting hidden files, embedded data, metadata, and common steganography techniques.

Unfortunately, the analysis did **not** reveal any useful information that could lead to the flag.

---

## Switching to OSINT

Since the steganography route produced no results, I shifted my focus toward **Open Source Intelligence (OSINT)**.

The room description mentions following the account that posted the brochure, suggesting that social media investigation is the intended path.

---

## Finding the Resort

The official Instagram page of the resort is:

> https://www.instagram.com/thebytelotusresort

After opening the profile, I inspected its activity.

Instead of analyzing the posts immediately, I checked the **Following** list.

---

## Interesting Discovery

The resort account follows only **one** Instagram user:

> https://www.instagram.com/veratheconcierge

This immediately stood out because the challenge description specifically hinted that the trail **doesn't end at the hotel**.

![Vera Profile](/assets/vera.png)

---

## Investigating Vera's Profile

Vera's Instagram profile contains **three separate posts**.

At first glance they appear unrelated, but together they form a single **Base64-encoded string**.

The intended solution is to:

1. Read the contents of all three posts.
2. Concatenate them in the correct order.
3. Decode the resulting Base64 string.

---

## Decoding

After combining the three fragments, decode the final string using any Base64 decoder.

The decoded output reveals the **TryHackMe flag**, completing the challenge.

---

# Methodology Summary

1. Download the provided ZIP archive.
2. Extract `thebrochure.png`.
3. Analyze the image using Aperi'Solve.
4. Determine that no useful hidden data exists.
5. Pivot to OSINT investigation.
6. Visit the resort's Instagram profile.
7. Inspect the account's **Following** list.
8. Discover **@veratheconcierge**.
9. Read the three Instagram posts.
10. Concatenate the contents.
11. Decode the resulting Base64 string.
12. Obtain the final flag.

---

# Key Takeaways

- Not every challenge involving an image requires steganography.
- Always pay close attention to the wording of the challenge description; it often hints at the intended methodology.
- OSINT investigations frequently involve examining social media relationships such as followers, following lists, and linked accounts.
- When one investigative path fails, pivoting to another methodology is often the fastest way forward.
- Encodings such as Base64 are commonly used to split secrets across multiple sources, requiring careful collection before decoding.

---

# Conclusion

**The Brochure** is a straightforward but well-designed OSINT challenge that encourages participants to think beyond traditional image forensics. Although the supplied image appears to invite steganographic analysis, the real solution lies in following the digital trail left on social media. By investigating the resort's Instagram account, identifying the only account it follows, and reconstructing a fragmented Base64 string from Vera's posts, the final flag can be recovered. The room serves as an excellent reminder that successful OSINT often depends more on observation and logical pivots than on complex tools.

---

**Author:** Soumyabrata Mukherjee  
**Platform:** TryHackMe  
**Category:** OSINT
