# TryHackMe — Room 404 Writeup

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
| **Room Name** | Room 404 |
| **Category** | Web |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |

### Description

> *He booked the quiet room. It's not on the floor plan, not in the brochure, not on any door. But port 8080 is wide open, and the rooms it never lists are the ones worth finding.*

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

This task simply requires clicking the **Completed** button.

---

# Task 2

## Objective

The challenge description hints that an undisclosed service is running on **port 8080**.

Our objective is straightforward:

1. Dump the exposed source code.
2. Locate the flag.

---

## Initial Enumeration

I first visited the web service running on **port 8080**.

![Website](/assets/web.webp)

The website appeared to be a simple landing page with no interactive functionality. None of the links were clickable, and there wasn't much to investigate manually.

This suggested that the real vulnerability might lie behind the application rather than the visible interface.

---

## Enumerating the Web Server

To discover any exposed files or directories, I used **Nikto**.

```bash
nikto -host http://<IP_ADDRESS>:8080
```

After the scan completed, Nikto reported an exposed **`.git` directory**.

This is a common web security misconfiguration where the Git repository is accidentally left accessible from the web server.

---

## Dumping the Git Repository

To recover the source code, I used **git-dumper**.

Repository:

https://github.com/arthaud/git-dumper

### Installation

```bash
python3 -m venv venv
source venv/bin/activate
pipx install git-dumper
```

### Usage

```bash
git-dumper http://<IP_ADDRESS>:8080/.git output-dir
```

The tool successfully reconstructed the Git repository into the specified output directory.

---

## Reviewing the Source Code

After dumping the repository, I inspected its contents.

```bash
cd output-dir
thunar .
```

The repository contained several files, including:

- `app.js` — Front-end JavaScript logic
- `index.html` — Landing page
- `README.md` — Internal staging documentation

The first two files contained only the application's front-end code.

However, while inspecting **README.md**, I discovered the **flag** embedded within the internal documentation.

The flag can then be submitted to complete the room.

---

# Methodology Summary

1. Visit the web application on port **8080**.
2. Notice that the website contains little useful functionality.
3. Enumerate the web server using **Nikto**.
4. Discover the exposed **`.git` directory**.
5. Dump the Git repository using **git-dumper**.
6. Inspect the recovered source code.
7. Open **README.md**.
8. Retrieve the flag.

---

# Key Takeaways

- Never expose the **`.git`** directory on a production web server.
- Enumeration is often more valuable than manually browsing a website.
- Git repositories may contain sensitive files, documentation, secrets, or previous commits that are not publicly accessible through the website itself.
- Tools such as **Nikto** and **git-dumper** are extremely useful for identifying and exploiting common web misconfigurations.

---

# Conclusion

**Room 404** is a beginner-friendly web challenge that highlights the dangers of exposing version control repositories. Although the website itself appears uninteresting, proper enumeration quickly reveals an exposed **`.git`** directory. By dumping the repository and reviewing the recovered files, the hidden flag is easily located within the project's internal documentation. The room reinforces an important lesson in web security: always enumerate thoroughly, and never leave development artifacts such as Git repositories accessible on production servers.

---

**Author:** Soumyabrata Mukherjee  
**Platform:** TryHackMe  
**Category:** Web
