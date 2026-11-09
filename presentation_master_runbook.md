# Client Presentation Master Runbook: Closed-Loop Automation Demo

**Architecture:** ServiceNow ITSM <--> Ansible AWX <--> Multi-Cloud (AWS LocalStack & Cisco IOS-XE)  
**Speaker Role:** Lead Solutions Architect / Automation Engineer  
**Target Duration:** 15–20 minutes  

---

## 1. Pre-Meeting Setup (10 Minutes Before the Call)

Have these three windows open and arranged on your screen:

```
+-------------------------------------------------------------------+
|  WINDOW 1 (Browser Tab 1): Interactive Simulator                  |
|  File: C:\Users\sarat\Downloads\snow-awx-demo\demo-simulator.html |
+-------------------------------------------------------------------+
|  WINDOW 2 (Browser Tab 2): Real ServiceNow PDI                    |
|  URL: https://dev439213.service-now.com                           |
|  Logged in as: admin                                              |
+-------------------------------------------------------------------+
|  WINDOW 3 (PowerShell / Terminal): AWS LocalStack Inspector       |
|  Working Directory: C:\Users\sarat\Downloads\snow-awx-demo        |
+-------------------------------------------------------------------+
```

### Pre-Flight Checklist:
1. [ ] **Verify LocalStack Server is Running:**  
   Open PowerShell and run:
   ```powershell
   python C:\Users\sarat\Downloads\snow-awx-demo\scripts\inspect_aws.py
   ```
   *Expected:* Prints the green table with `sg-f84749a20d87453e3` and `API Query Status: 200 OK`.
2. [ ] **Open `demo-simulator.html`:**  
   Double-click the file in Chrome or Edge. Ensure tabs are responsive.
3. [ ] **ServiceNow Ready:**  
   In your ServiceNow tab, keep the search bar ready, or have **`CHG0030004`** open.

---

## 2. Minute-by-Minute Presentation Script

### Part 1: The Executive Hook (Minutes 00:00 – 02:30)

> **What to Say:**  
> *"Good morning / afternoon everyone. Today, I am excited to demonstrate our **Closed-Loop Multi-Cloud Automation Pipeline** integrating **ServiceNow ITSM**, **Ansible AWX**, **Cisco Networking**, and **AWS Cloud Infrastructure**.*  
>  
> *In most enterprise organizations today, simple infrastructure changes—like opening a firewall port or configuring a router interface—require 3 to 5 business days of manual handoffs across ticketing queues, network teams, and cloud admins. This introduces human configuration drift, prolonged MTTR, and zero automated rollback if something breaks.*  
>  
> *Today, we’re showing you a **fully governed, closed-loop solution**: an engineer requests a change in ServiceNow, Ansible automates pre-checks, policy validation, execution, and post-validation, and writes a complete audit trail back into the ticket—**reducing MTTR from 3 days to under 45 seconds** with zero human error."*

---

### Part 2: Interactive Simulator Demonstration (Minutes 02:30 – 08:00)

*Switch screen to: **`demo-simulator.html`***

> **What to Say:**  
> *"To give you a real-time, side-by-side view of how ServiceNow communicates with Ansible AWX under the hood, we are looking at our interactive simulation engine.*  
> *On the left, you see the **ServiceNow ITSM Portal**. On the right, you see the live **Ansible AWX Execution Console**."*

#### Scenario A: Cisco IOS-XE Interface Change (Pre/Post Validation & Diff)
1. Click the tab: **`Cisco Interface Change`**.
2. Click the blue button: **`Submit & Run Playbook`**.
3. **Point to the Right Screen:**  
   > *"Notice AWX immediately picks up the job. It connects to the Cisco Catalyst router, runs a read-only pre-check, computes a unified configuration diff, applies the change, and runs a 3-point post-check verification."*
4. **Point to the Left Screen:**  
   > *"Once AWX finishes, notice how the ServiceNow ticket status automatically flips to **SUCCESS**, and the Work Notes are populated with the exact before/after diff and verification checklist. No human had to log into a CLI."*

#### Scenario B: AWS Security Policy Guardrail (Rejection)
1. Click the tab: **`AWS Policy Rejection`**.
2. Click **`Submit & Run Playbook`**.
3. **Talk Track:**  
   > *"Now, what if a developer asks for a dangerous rule, such as opening port 22 (SSH) to the entire public internet (`0.0.0.0/0`)?*  
   > *Watch what happens: AWX intercepts the change against corporate security policy `SEC-POL-04`. It immediately **REJECTS** the ticket and halts execution **before touching cloud infrastructure**. The ticket is updated with remediation guidance."*

#### Scenario C: Automated Self-Healing Rollback
1. Click the tab: **`AWS Auto-Rollback`**.
2. Click **`Submit & Run Playbook`**.
3. **Talk Track:**  
   > *"What happens if a change is approved, applied, but fails operational health checks?*  
   > *Here, port 9090 was added, but our synthetic post-check detected high latency. Rather than waiting for users to complain, AWX automatically executed an inverse API call, revoked the rule, restored the baseline state, and marked the ticket as **ROLLED BACK** in under 10 seconds."*

---

### Part 3: Proof of Reality — Live ServiceNow PDI & Local AWS (Minutes 08:00 – 13:00)

*Switch screen to: **ServiceNow Tab (`dev439213.service-now.com`)***

> **What to Say:**  
> *"Now let's verify that this isn't just a mock UI. Let's look at our real, live ServiceNow production developer instance."*

1. **Show Ticket `CHG0030004` (AWS Live Change):**
   - In the top search bar, type **`CHG0030004`** and press Enter.
   - Scroll down to the **Work Notes**:
     > *"Here is Change Request `CHG0030004`. Notice our custom fields (`u_automation_status`, `u_sg_id`) and the live Work Note table showing baseline Port 443 alongside newly authorized Port 8443."*

2. **Show Live Cloud State in Terminal (LocalStack API):**
   - Switch to your PowerShell window and run:
     ```powershell
     python scripts/inspect_aws.py
     ```
   - **Talk Track:**  
     > *"To prove 100% cloud API integration without incurring unnecessary public cloud costs or risks during testing, our pipeline interfaces with an enterprise **LocalStack AWS sandbox**.  
     > As you can see, querying the AWS EC2 REST API directly confirms the Security Group `sg-f84749a20d87453e3` in `ap-south-1` matches the exact state recorded in ServiceNow."*

3. **Show Ticket `CHG0030001` (Cisco Change & Real Attachment):**
   - In ServiceNow search, type **`CHG0030001`**.
   - Show the green success card and the **Attachments bar** at the top:
     > *"Notice that AWX attached the actual `CHG0030001_unified_diff.txt` directly to the ticket record for permanent compliance and audit readiness."*

---

### Part 4: Production Architecture & Scaling (Minutes 13:00 – 15:00)

> **What to Say:**  
> *"To take this from our demo environment into your enterprise production:*
> 1. ***Security & Credentials:** In production, AWX connects to your AWS accounts via **IAM STS AssumeRole**—no static keys are ever stored. Cisco credentials use TACACS+/SSH keypairs injected into isolated execution environments.*
> 2. ***Zero Pipeline Changes:** The exact same Ansible playbooks and ServiceNow business rules you saw today run in production; we simply change the target inventory endpoint from our dev sandbox to your production VPC and routers.*
> 3. ***Auditing & Compliance:** Every single action has bidirectional traceability: the ServiceNow sys_id is tagged in AWS CloudTrail and Cisco syslog, and the AWX job ID is permanently recorded on the ServiceNow ticket."*

---

## 3. Client Q&A / Objection Handling Cheat Sheet

| Question | Strong Answer |
| :--- | :--- |
| **"Is LocalStack different from real AWS?"** | *"LocalStack uses the exact same AWS Boto3 SDK and REST endpoints as production AWS EC2. The playbook code is 100% identical. In production, we simply point the AWX credential to your AWS IAM Role ARN."* |
| **"What if ServiceNow approval is required before running?"** | *"In ServiceNow, the business rule can be configured to fire on `State = Scheduled` or `Approval = Approved` rather than on insert. This ensures full CAB approval before AWX triggers."* |
| **"Can this roll back Cisco network changes too?"** | *"Yes! For Cisco, Ansible takes a snapshot of `running-config` during pre-check. If post-validation fails, it immediately deploys the rollback candidate config using `cisco.ios.ios_config`."* |
| **"How are credentials secured?"** | *"Credentials are never stored in tickets or git. They reside inside AWX Credential Stores (or CyberArk/HashiCorp Vault) and are injected dynamically in memory at execution time."* |
