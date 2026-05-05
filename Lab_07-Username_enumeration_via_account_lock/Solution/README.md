# Lab Solution: Username Enumeration via Account Lockout

## Overview
This document outlines the step-by-step solution for exploiting a login page vulnerable to username enumeration via account lockout mechanisms. By brute-forcing usernames and observing lockout error messages, we can identify a valid username. Once identified, a standard brute-force attack is used to deduce the correct password.

## Prerequisites
* **Burp Suite Professional** (or Community Edition, though Intruder is rate-limited)
* A list of candidate usernames
* A list of candidate passwords

## Step-by-Step Guide

### Part 1: Enumerating the Username
1. **Intercept the Login Request:** Ensure Burp Suite is running and intercepting traffic. Submit an invalid username and password on the target's login page. Send the resulting `POST /login` request to **Burp Intruder**.
2. **Configure Payload Positions:** * In Intruder, change the attack type to **Cluster bomb**.
   * Clear all auto-assigned payload positions.
   * Add a payload marker to the value of the `username` parameter.
   * Add a second, blank payload marker to the very end of the request body by clicking **Add §**. 
   * Your request body should look similar to this:
     ```http
     username=§invalid-username§&password=example§§
     ```
3. **Configure Payloads:** Go to the **Payloads** side panel.
   * **Payload set 1:** Paste your list of candidate usernames.
   * **Payload set 2:** Select **Null payloads** as the payload type and configure it to generate **5** payloads. This forces Intruder to submit each username 5 times consecutively, triggering the account lockout mechanism for valid users.
   * Click **Start attack**.
4. **Analyze Results:** Once the attack finishes, sort the results by Length. You will notice that responses for one specific username are longer than the rest. Inspect the response body for this username to find the following distinct error message: 
   > `You have made too many incorrect login attempts.`
   
   Make a note of this valid username.

### Part 2: Brute-Forcing the Password
5. **Set Up Sniper Attack:** Send the original `POST /login` request to Burp Intruder again. 
   * Change the attack type to **Sniper**.
   * Set the `username` parameter to the valid username you just discovered.
   * Add a single payload marker to the value of the `password` parameter.
6. **Configure Password Payloads:**
   * Go to the **Payloads** tab and paste your list of candidate passwords.
   * Go to the **Settings** tab and create a **Grep - Extract** rule to capture the error message from the response body (e.g., "Invalid username or password").
   * Click **Start attack**.
7. **Identify the Password:** When the results populate, review the grep extract column. Most attempts will show the standard error message. Look for the single response that **does not** contain any error message (or yields a 302 redirect). Make a note of this password.

### Part 3: Accessing the Account
8. **Final Login:** Wait for approximately one minute to allow the account lockout to reset. Log in normally via the web interface using the valid username and password you identified to access the user account page and solve the lab.
