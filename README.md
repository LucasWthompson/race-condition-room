# TryHackMe - Race Conditions

This write-up covers the final challenge in the [Race Conditions](https://tryhackme.com/room/raceconditionsattacks) room on TryHackMe, where the goal was to exploit a race condition vulnerability in a banking application to amass over $1000 in a single account.

---

## 🧠 Challenge Overview

The application simulates a banking platform with three accounts, each starting with $100. You are given credentials for each account and must exploit a race condition to get one account over $1000.

**Target URL:** `http://10.10.103.108:5000`  
**User Credentials:**

| Name             | Username | Password     |
|------------------|----------|--------------|
| Rasser Cond      | 4621     | blueApple    |
| Zavodni Stav     | 6282     | whiteHorse   |
| Warunki Wyscigu  | 9317     | greenOrange  |

---

## 🧪 Exploitation Process

I had three accounts with $100 each and needed to increase one account’s balance to over $1000. I began by transferring $45 from Rasser Cond’s account to Zavodni Stav’s.

I captured the request in Burp Suite’s Repeater, grouped the request, and sent 25 parallel copies using the “Send group (parallel)” feature. This resulted in 8 successful transfers due to a race condition on the server side.

![Screenshot 364](./Screenshot%20(364).png)

Next, I repeated the process by transferring $45 from Warunki Wyscigu’s account to Zavodni Stav’s. This time I increased the number of parallel requests to 50, but again, only 8 were successful. This seemed to indicate a cap or consistency in how many race conditions would succeed.

![Screenshot 365](./Screenshot%20(365).png)

After the second wave of race condition abuse, Zavodni Stav’s account had $698.50.

![Screenshot 366](./Screenshot%20(366).png)

Since both Cond’s and War’s accounts had negative balances at this point, I decided to transfer $325 from Stav’s account back to Cond’s. Again, I sent 25 requests in parallel. Like before, 8 were successful.

![Screenshot 367](./Screenshot%20(367).png)

That pushed Cond’s account over the $1000 threshold, hitting $1862.25 and revealing the final flag.

![Screenshot 368](./Screenshot%20(368).png)

---

## 🛡️ Detection, Mitigation & Prevention

### 🔍 Detection
- **Unusual Transaction Patterns**: Sudden rapid-fire requests from a single source or multiple transfers within milliseconds should raise alerts.
- **Logging and Rate Analysis**: Monitor and log transaction timestamps, source IPs, and session IDs for behavioral analysis.
- **Inconsistent State Behavior**: Look for discrepancies in account states before and after transaction batches.

### 🛠️ Mitigation
- **Database-Level Locks**: Use row-level or table-level locking to prevent simultaneous updates on the same resource.
- **Atomic Transactions**: Leverage ACID-compliant operations where the debit and credit are bundled in a single atomic operation.
- **Request Throttling**: Implement rate limiting on critical endpoints like fund transfers.

### ✅ Prevention
- **Server-Side Validation**: Never rely on client-side enforcement for balances or constraints.
- **Synchronous Handling**: Queue and process fund transfers synchronously with proper locking.
- **Concurrency Testing**: Include race condition scenarios in QA testing using tools like Burp Suite, OWASP ZAP, or custom scripts.

---

✅ **Flag Obtained:** `THM{BANK-RED-FLAG}`  
🎯 **Account Reached:** `Rasser Cond` with `$1862.25`

---
