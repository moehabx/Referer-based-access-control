# Lab: Access Control Bypass via Referer Trust (PortSwigger)

**Short summary**  
In this lab we find an access-control bypass that arises because the application trusts the `Referer` header to gate admin-only functionality. By replaying requests and injecting a forged `Referer` pointing to the admin area, a regular user can gain admin privileges. This repo documents the manual steps, request analysis, and mitigation notes.  


---

## Table of contents
- [Summary](#summary)  
- [Prerequisites](#prerequisites)  
- [Walkthrough](#walkthrough)  
  - Landing & login  
  - Discover admin functionality  
  - Capture and analyze requests  
  - Forge referer and escalate privileges  

---

## Summary
An application endpoint enforces access control by checking the `Referer` header. Because the `Referer` can be controlled by the client, this trust is insufficient: a normal user can replay an authenticated request and add a `Referer` value from the admin area — causing the server to treat the request as if it came from an admin page. The result is privilege escalation (regular → admin).

---

## Prerequisites
- PortSwigger lab instance (this write-up targets the lab environment)  
- A Burp proxy or an HTTP interception tool (to capture and replay requests)  
- Two accounts provided by the lab: a normal user and admin credentials (or steps to obtain them in-lab)

---

## Walkthrough

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5719e01c-06d7-4d41-9622-bd9363ff5760" />


### 1. Landing & login
- Start at the target application home page and locate **My Account** / login.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f8434219-0094-4998-a214-64c954f8c783" />


- Use the lab-provided admin credentials to log in (or follow lab steps to obtain them).  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b37beb8b-aade-4f1e-a3aa-5d6180b8af2c" />


- After logging in as admin, open the **admin panel**. Confirm you can upgrade/degrade users.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f4ad92e7-be40-4ed8-bdd8-f3b670c4ff67" />


### 2. Discover admin functionality & record request
- Use the admin panel to perform a user role change (upgrade or downgrade). While doing this, capture the request in Burp (Proxy → HTTP history). Note the request method, URL and headers (especially `Referer`).
-  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0d4563e9-de67-4403-b690-460c422cb894" />

the user is upgraded

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c0aace77-c3e6-4246-9cd4-3db229f2550d" />


### 3. Login as a normal user
- Log out and then log back in as a regular (non-admin) user. Confirm limited privileges.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/afbe37fa-5cfd-46ac-a3ec-1ba2cdb9166e" />



### 4. Replay request without referer → unauthorized
- Take the captured admin request URL and try to open it directly in a new browser tab or send it from Repeater *without* the original `Referer` header. You should receive `401 Unauthorized`

this is the admin request we get a copy of the get method and try to use it 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fe2315ea-7a5b-415c-9c92-d61e706070ea" />

we got unauthorized
<img width="994" height="214" alt="image" src="https://github.com/user-attachments/assets/9824c153-aa8b-459d-941c-11de0d26ede5" />


### 5. Replay request with admin `Referer` → success
- Copy the `Referer` value that originally pointed to the admin panel (from the admin session capture).
   
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/00445a42-ad59-466e-8363-7b27ad05fb90" />


- Add that `Referer` header to the replayed request you issue and replace carlos with wienet(your account) to solve the lab.  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/54de21e1-bacd-44d6-b60b-18c472d2cba0" />


- The server treats the request as if it came from the admin page and allows the privilege change we got a 302 found and The normal user is upgraded to admin.  

**Result:** You now have admin-level access while using a regular account. This completes the lab.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8867e65b-2d1a-4c0c-aabc-5c63dc94c44e" />


---


