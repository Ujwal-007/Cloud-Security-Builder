# AWS Cloud Security, Governance & Infrastructure Lab

A comprehensive, production-grade implementation demonstrating enterprise-level AWS security controls, identity and access management (IAM), KMS envelope encryption, VPC network auditing, and automated CloudWatch threat alerting.

---

## 🏗️ Architecture & Implementation Domains

### 1. Identity & Access Management (IAM) & Encryption Governance
* **Customer Managed KMS Keys:** Provisioned `MyKMSKey` with strict administrative policies, custom key material, and annual automated rotation governance.
* **Least-Privilege Policies:** Created `PolicyForFinancialAdvisors` scoping granular read/write boundaries explicitly for KMS and S3 services.
* **Access Control Enforcement:** Verified boundary failures (`AccessDenied` errors via KMS decrypt restrictions) when unauthorized users like `paulo` attempt to access protected objects.

### 2. Compute Security & Secrets Management
* **EBS Encryption:** Deployed `EncryptedInstance` utilizing custom KMS customer-managed keys (`MyKMSKey`) for encrypted block storage security.
* **Secrets Manager Integration:** Utilized AWS Systems Manager Session Manager to securely query and retrieve credentials from AWS Secrets Manager via instance IAM profiles.

### 3. Network Security, Perimeter Auditing & Monitoring
* **VPC & Subnets:** Architected `LabVPC` with isolated subnets and active route mapping.
* **Security Groups & NACLs:** Hardened perimeter defenses via `WebServerSecurityGroup` (restricting SSH to specific administrative `/32` & `/29` IP ranges) and explicit Network ACL rule bounds.
* **VPC Flow Logs:** Enabled `LabVPCFlowLogs` streaming packet accept/reject telemetry into CloudWatch logs.
* **Incident Response & Alerting:** Configured metric filters and CloudWatch Alarms (`Not valid users exceeding limit on EncryptedInstance`) coupled with SNS email notifications for real-time unauthorized SSH attempt alerting.

---

## 📁 Detailed Execution Guide & Evidence Artifacts

This section documents every step executed throughout the laboratory environment, coupled with the corresponding proof-of-work screenshots.

### Step 1: Network Architecture & Storage Setup
* **Objective:** Establish an isolated virtual private cloud (`LabVPC`), configure purpose-built S3 buckets, and enforce secure baseline storage policies.
* **Execution Details:** Created public and private subnets, configured internet gateways, and provisioned S3 buckets (`data-bucket-4859` and `data-bucket-4859-b`) to store operational datasets and log archives.
* **Verification Artifacts:**
  * S3 bucket object lists, directory structures, and file uploads:  
    ![S3 Bucket Objects](screenshots/Screenshot%202026-11-07%20103955.png)
  * Handling S3 permission boundaries and listing constraints:  
    ![S3 Permissions Test](screenshots/Screenshot%202026-11-07%20104052.png)
  * Successful raw data object management (`customers.csv` and `loan-data.csv`):  
    ![S3 Customers CSV](screenshots/Screenshot%202026-11-07%20105448.png)  
    ![S3 Loan Data CSV](screenshots/Screenshot%202026-11-07%20192810.png)
  * VPC Resource Mapping showing `LabVPC` layout and subnet associations:  
    ![VPC Resource Map](screenshots/Screenshot%202026-11-07%20114658.png)
  * Public web server availability validation responding with `Hello world from WebServer!`:  
    ![WebServer Public Verification](screenshots/Screenshot%202026-11-07%20123416.png)

---

### Step 2: VPC Flow Logs & Traffic Auditing
* **Objective:** Capture network traffic metadata across network interfaces and analyze communication attempts in CloudWatch.
* **Execution Details:** Attached a VPC Flow Log (`LabVPCFlowLogs`) pointing to a CloudWatch log group, recording all accepted and rejected TCP/UDP packets.
* **Verification Artifacts:**
  * Successful creation and configuration of VPC flow logs:  
    ![VPC Flow Log Creation](screenshots/Screenshot%202026-11-07%20115114.png)
  * Raw flow log event streams showcasing `REJECT` and `ACCEPT` actions:  
    ![CloudWatch Flow Log Events](screenshots/Screenshot%202026-11-07%20121034.jpg)
  * Filtered log events querying specific traffic payloads against external IP addresses (e.g., `44.219.58.130`):  
    ![CloudWatch IP Filtered Stream](screenshots/Screenshot%202026-11-07%20122305.png)

---

### Step 3: Identity, Least-Privilege IAM, & KMS Governance
* **Objective:** Implement strict access control lists and robust encryption frameworks using AWS Key Management Service (KMS).
* **Execution Details:** Generated a customer-managed key (`MyKMSKey`), enforced annual automated rotation, and authored a custom IAM policy (`PolicyForFinancialAdvisors`) to limit downstream user capabilities.
* **Verification Artifacts:**
  * KMS Customer Managed Key general configuration and policy view:  
    ![KMS Key Configuration](screenshots/Screenshot%202026-11-07%20191501.png)
  * IAM Policy `PolicyForFinancialAdvisors` permissions definition targeting KMS and S3:  
    ![IAM Policy Definition](screenshots/Screenshot%202026-11-07%20191758.png)
  * Explicit access denial evaluation (`AccessDenied`) triggered when user `paulo` lacks `kms:Decrypt` permissions:  
    ![AccessDenied Error XML](screenshots/Screenshot%202026-11-07%20193114.png)
  * KMS Key material and rotation tab displaying active 365-day rotation settings:  
    ![KMS Key Rotation Active](screenshots/Screenshot%202026-11-08%20004833.png)

---

### Step 4: Compute Hardening & Secrets Management
* **Objective:** Secure EC2 virtual machines using encrypted root volumes and abstract sensitive configuration data via AWS Secrets Manager.
* **Execution Details:** Launched an encrypted EC2 instance (`EncryptedInstance`) bound to `MyKMSKey` and interacted with Secrets Manager via AWS CLI inside a Session Manager shell.
* **Verification Artifacts:**
  * Standard EC2 volume attachment and unencrypted state comparison:  
    ![EC2 Unencrypted Volume](screenshots/Screenshot%202026-11-07%20193701.png)
  * Encrypted EC2 instance (`EncryptedInstance`) root volume linked to KMS ID:  
    ![EC2 Encrypted Volume KMS](screenshots/Screenshot%202026-11-07%20195022.jpg)
  * AWS CLI execution inside Session Manager listing and fetching secure strings from Secrets Manager:  
    ![Session Manager Secrets CLI](screenshots/Screenshot%202026-11-07%20200200.png)
  * S3 default bucket encryption server-side configuration mapping:  
    ![S3 SSE KMS Default Encryption](screenshots/Screenshot%202026-11-07%20200440.png)

---

### Step 5: CloudTrail Logging & Athena Query Analytics
* **Objective:** Centralize API auditing trails and parse audit logs using serverless SQL queries in Amazon Athena.
* **Execution Details:** Enabled a multi-region CloudTrail trail (`data-bucket-reads-writes`) and configured an Athena table structure to query S3 data access events.
* **Verification Artifacts:**
  * Successful Athena query execution parsing CloudTrail logs for `GetObject` events against sensitive files:  
    ![Athena SQL Query CloudTrail](screenshots/Screenshot%202026-11-07%20214826.png)
  * Active CloudTrail management console verifying data event coverage and logging status:  
    ![CloudTrail Trails Console](screenshots/Screenshot%202026-11-07%20215015.png)

---

### Step 6: Perimeter Security Hardening & Automated Incident Response
* **Objective:** Restrict administrative access vectors and establish automated CloudWatch metric alarms for suspicious behavior.
* **Execution Details:** Locked down security group ingress rules, tuned Network ACL boundaries, and routed threat metrics to an SNS notification topic.
* **Verification Artifacts:**
  * EC2 Security Groups dashboard inventory view:  
    ![EC2 Security Groups List](screenshots/Screenshot%202026-11-08%20004347.png)
  * `WebServerSecurityGroup` inbound rules explicitly restricting SSH to authorized IP ranges (`44.219.58.130/32` & `18.206.107.24/29`) while allowing public HTTP traffic (`80`):  
    ![Security Group Inbound Rules](screenshots/Screenshot%202026-11-08%20004420.png)
  * `LabNACL` network ACL rule configuration mapped to subnets:  
    ![Network ACL Inbound Rules](screenshots/Screenshot%202026-11-08%20004719.png)
  * CloudWatch Alarm triggering an **In alarm** state for excessive unauthorized attempts (`Not valid users exceeding limit on EncryptedInstance`):  
    ![CloudWatch Metric Alarm In Alarm](screenshots/Screenshot%202026-11-08%20002510.png)
  * Automated SNS alert dispatch confirmation received via Gmail notification:  
    ![SNS Alarm Gmail Notification](screenshots/Screenshot%202026-11-08%20002453.png)

---

## 🛡️ Security Best Practices Applied
1. **Least Privilege Principle:** IAM policies are scoped tightly to explicit resource namespaces and specific action sets.
2. **Defense in Depth:** Combined security groups, network ACLs, VPC flow logs, and IAM permission boundaries.
3. **Automated Monitoring & Alerting:** Continuous log parsing paired with metric filters and real-time SNS notifications.
