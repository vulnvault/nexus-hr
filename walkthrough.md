### Official Walkthrough: NexusHR Black Box Breach

Lab Difficulty: Easy

Objective: Breach the NexusHR internal portal, gain initial access without provided credentials, and escalate privileges to retrieve the CEO's confidential data.

---

### **Phase 1: Initial Access (Reconnaissance)**

Step 1: The Dead End

Navigate to the target URL http://<TARGET_IP>:3000. You are presented with a login screen. Common default credentials will fail.

Step 2: Enumeration

Since we have no credentials, we must look for hidden resources. Run a directory scan using a tool like Gobuster or Dirb.

Bash

```
gobuster dir -u http://<TARGET_IP>:3000 -w /usr/share/wordlists/dirb/common.txt
```

**Observation:** The scan reveals a hidden directory: `/[REDACTED]` (Status: 200).

Step 3: The Leak

Navigate to the hidden directory in your browser. Because the server has Directory Listing Enabled, you will see a list of files.

- Click and download the backup file (e.g., `[REDACTED].bak`).
    
- Open the file to reveal valid credentials:
    
    > `User: [REDACTED] | Pass: [REDACTED]`
    

Step 4: Login & Flag 1

Return to the login page and authenticate with the leaked credentials. You will be redirected to the employee dashboard.

- **Success:** The **User Flag** is visible in the "Welcome Back" section.
    
- **Flag 1:** `VulnOS{[REDACTED]}`
    

---

### **Phase 2: Privilege Escalation (Logic Flaw)**

Step 5: Traffic Analysis

Click the "View Latest Payslip" button on the dashboard. Observe the network traffic.

- **Request:** `GET /api/payslip?id=[REDACTED]`
    
- **Response:** Your payslip details.
    

Step 6: The Blocked Attack

Attempt a standard IDOR attack by changing the ID to ceo.

- **Result:** `{"error": "Access Denied"}`. The application blocks this because your session does not match the requested ID.
    

Step 7: The Exploit (HTTP Parameter Pollution)

We need to bypass the security filter while still instructing the database to fetch the CEO's record. We can achieve this by supplying the id parameter twice.

- **The Logic Flaw:** The security middleware checks the **first** parameter and approves it. However, the database logic uses the **last** parameter to fetch the data.
    

Payload:

Append &id=ceo to the end of the URL.

- Final Attack URL:
    
    http://<TARGET_IP>:3000/api/payslip?id=[REDACTED]&id=ceo
    

Step 8: Success

Load the URL in your browser. The server returns the CEO's confidential salary data.

- **Flag 2:** `VulnOS{[REDACTED]}`
