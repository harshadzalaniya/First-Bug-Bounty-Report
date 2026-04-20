# Insecure Direct Object Reference (IDOR) in TryPresentMe Website

**Room:** IDOR - Santa’s Little IDOR (Advent of Cyber 2025 - Day 5)  
**Platform:** TryHackMe  
**Vulnerability Type:** Broken Access Control - Insecure Direct Object Reference (IDOR)  
**Severity:** High  
**Date:** April 20, 2026  
**Researcher:** Harshad Zalaniya

## Executive Summary
While testing the **TryPresentMe** website (a Christmas-themed platform for parents to manage gift vouchers), a critical **Insecure Direct Object Reference (IDOR)** vulnerability was identified in the account information endpoint.

The application allows any authenticated user to view private account details of other parents by simply modifying the `user_id` parameter. This vulnerability enables **horizontal privilege escalation** and can lead to exposure of sensitive family and voucher information.

This is a classic IDOR issue commonly found in bug bounty programs.

## Vulnerability Description
IDOR occurs when an application references internal objects (such as user accounts) using a direct, user-controllable identifier (`user_id`) without properly validating whether the requesting user is authorized to access that object.

In this room, the endpoint trusts the `user_id` parameter completely and returns sensitive data belonging to other users without performing ownership or permission checks.

## Affected Asset
- Target Application: TryPresentMe[](http://[MACHINE_IP])
- Vulnerable Endpoint: `/api/parents/view_accountinfo?user_id=<ID>`
- Login used: `niels` / `TryHackMe#2025`

## Impact
- Unauthorized access to other parents' private information (names, family details, number of children, voucher data).
- Mass data leakage through enumeration of `user_id` values.
- Potential abuse of the gift voucher system.
- In a real production environment, this could result in privacy violations, data breaches, and loss of user trust.

## Steps to Reproduce (PoC)

1. Deploy the TryHackMe machine and access the TryPresentMe website.
2. Log in using the provided credentials:  
   **Username:** `niels`  
   **Password:** `TryHackMe#2025`
3. Navigate to the account or family information section.
4. Observe the request in Browser DevTools (Network tab) – it contains a parameter like `user_id=5`.
5. Modify the `user_id` value to another number (e.g., `1`, `10`, `15`, etc.).
6. The application returns another parent's private account details without any error.

### Finding the Parent with 10 Children
- Use Burp Suite Intruder (Sniper attack) on the `user_id` parameter with a payload list (1–50).
- Sort results by response length to quickly identify the account with the most children/vouchers.
- The target account (with 10 children) was found at `user_id = [REPLACE WITH THE ID YOU FOUND]`.

**Proof of Concept Screenshots:**    
- Screenshot 1: Logged in as `niels` [own account](./Screenshot1.png)
- Screenshot 2: After changing `user_id` → [Another parent's data is displayed](./Screenshot2.png)  
- Screenshot 3: Burp Intruder results showing different response lengths [Respnose](Screenshot3.png)  
- Screenshot 4: Account with 10 children successfully accessed [Success](Screenshot4.png)

### Expected Behavior
- The server should perform proper authorization checks on every request before returning sensitive data. 
- Specifically, it must verify that the requested user_id belongs to the currently authenticated user or that the user has sufficient privileges (e.g., admin role) to view other accounts. 
- If the user is not authorized, the request should be denied with an appropriate error message.

### Actual Behavior
- The application directly uses the client-supplied user_id parameter to query and display account information without validating ownership or permissions. 
- As a result, any authenticated user can successfully access private data of other users by simply changing the numeric ID in the request.

## Remediation Suggestions

- Implement server-side authorization checks for every object access. Before returning data, always verify if (requested_user_id != current_session_user_id && !isAdmin) { deny access; }
- Use indirect object references (e.g., random UUIDs) instead of predictable sequential numeric IDs where possible.
- Follow the principle of least privilege and enforce context-based access control on all sensitive endpoints.
- Add proper logging and rate limiting on endpoints that handle user IDs to detect enumeration attempts.
- Adopt secure coding practices and regularly review access control logic.
- Reference: OWASP Top 10 2021 – A01:2021 Broken Access Control (Broken Object Level Authorization - BOLA).

## Tools Used

- TryHackMe Attack Machine
- Mozilla Firefox / Google Chrome with Developer Tools (Network tab)
- Burp Suite Community Edition (for request interception and Intruder attacks)
- Manual testing with URL parameter manipulation

## What I Learned

- How a seemingly simple parameter like user_id can lead to serious access control failures if not properly validated.
- The critical difference between Authentication and Authorization in web applications.
- Practical use of Burp Intruder for efficient ID enumeration.
- Why IDOR remains one of the most frequently reported vulnerabilities in bug bounty programs.
- The importance of always performing server-side ownership validation.

This room provided an excellent, story-based introduction to IDOR and helped strengthen my understanding of Broken Access Control vulnerabilities.

## References

- TryHackMe Room: https://tryhackme.com/room/idor-aoc2025-zl6MywQid9
- OWASP Broken Access Control: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- PortSwigger Web Security Academy – Insecure Direct Object References
