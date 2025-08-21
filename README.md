# Automated Secret Rotation for APIGEE DIY Credentials using AWS

## 1. Overview
This document describes the automation of secret rotation for APIGEE DIY credentials using AWS services.  
The solution replaces a manual, error-prone process with an automated workflow that securely rotates secrets every 3 months and provides status notifications to application users.

---

## 2. Problem Statement (Current Manual Process)
Currently, APIGEE DIY credentials are stored in **AWS Secrets Manager**, but no automated secret rotation is in place.  
The existing process is manual and involves the following steps:
1. User logs into the **APIGEE DIY app** with username and password.  
2. Manually generates a new secret.  
3. Copies the new secret and pastes it into **Secrets Manager**.  
4. Uses the old key in the DIY UI to manually delete the previous secret.  

**Challenges with this approach:**
- Error-prone and time-consuming.  
- Lack of regular rotation increases security risks.  
- No automated notifications on status or failures.  

---

## 3. Proposed Solution (Automation)
To address the manual inefficiencies, the process is automated using AWS services. The automated solution:
- Rotates secrets **every 3 months** without manual intervention.  
- Leverages an **SSM Automation Runbook** to orchestrate the flow.  
- Calls APIGEE DIY APIs (exposed via proxy) to create and delete credentials.  
- Updates the new secret in **Secrets Manager**.  
- Notifies application users via **Amazon SNS** email with detailed status.  

---

## 4. Detailed Architecture (Step-by-Step Flow)
1. **Amazon EventBridge (Scheduled Rule)** triggers the workflow every 3 months.  
2. **SSM Automation Runbook** is invoked to orchestrate the process.  
3. Inside the Runbook, the following **3 Lambda functions** execute sequentially:  
   - **Lambda 1:** Calls the **Apigee proxy endpoint** to generate a new key-secret pair.  
   - **Lambda 2:** Updates the **new key-secret pair** in AWS **Secrets Manager**.  
   - **Lambda 3:** Calls the Apigee proxy endpoint to **delete the old key-secret pair**.  
4. After execution, the **Runbook reports status** (success/failure) to **Amazon SNS**.  
5. **Amazon SNS** sends email notifications to application users with detailed status.  

---

## 5. AWS Services Used
- **AWS Secrets Manager** → Stores application credentials.  
- **Amazon EventBridge** → Scheduled rule triggers the automation every 3 months.  
- **AWS Systems Manager (SSM) Automation Runbook** → Orchestrates the Lambda sequence.  
- **AWS Lambda** → Performs create/update/delete actions for secrets using Apigee proxy.  
- **Amazon SNS** → Publishes success/failure messages and notifies users via email.  

---

## 6. Workflow Diagram
```text
[Secrets Manager] → (Every 3 months) → [EventBridge Rule] → [SSM Automation Runbook] 
     └─> [Lambda 1: Create new secret via Apigee Proxy] 
     └─> [Lambda 2: Update Secrets Manager] 
     └─> [Lambda 3: Delete old secret via Apigee Proxy] 
     └─> [SNS Topic] → [Email Notification to Users]
7. Benefits & Outcomes

Eliminates manual effort in secret creation, update, and deletion.

Ensures regular rotation every 3 months.

Reduces human errors in secret handling.

Improves security posture by enforcing automated credential freshness.

Provides transparency with detailed success/failure notifications to users
