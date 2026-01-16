# Datavul-sql

**1.Title**
Unauthenticated SQL Injection Leading to Privilege Escalation in DataGear Authentication API

**2. Overview**
A critical SQL injection vulnerability exists in the authentication API endpoint of DataGear. This allows unauthenticated remote attackers to bypass login checks, escalate privileges to administrator level, and execute arbitrary SQL statements, leading to full compromise of the application and its underlying database.

**3. Affected Versions**
Confirmed Affected Version: 5.5.0
Note: The full range of affected versions is currently unknown. This vulnerability has been confirmed to exist in version 5.50, and other versions may also be affected.

**4. Technical Details**
Vulnerability Type: SQL Injection (CWE-89)
Attack Vector: Remote, Unauthenticated
Root Cause: The authentication endpoint does not properly sanitize user-supplied input before constructing SQL queries, allowing an attacker to inject malicious SQL payloads.
Exploit Example: 
#### Step 1: Verify Unauthenticated Access Restriction
To confirm the access control mechanism, an unauthenticated request was sent to a protected query endpoint. The server returned a **Permission denied** response, indicating that administrator privileges are required to access this endpoint. This is demonstrated in the attached screenshot, which shows the raw HTTP request and the `Permission denied` response.
<img width="1413" height="583" alt="189b25a4-daf6-4904-954d-f95cbf8f2ad6" src="https://github.com/user-attachments/assets/680c0539-a121-4a58-9c8b-594e10ca539e" />

#### Step 2: Exploit SQL Injection to Bypass Authentication
<img width="1414" height="638" alt="fa31b721-5103-42b3-a471-095483260f2b" src="https://github.com/user-attachments/assets/8c49fe08-8232-47de-aa2c-9a0bd8e235d1" />
<img width="1394" height="631" alt="6304fe79-76f2-48c2-b1cd-644d5f00e909" src="https://github.com/user-attachments/assets/91b76d47-bc20-4daf-a07a-a4b3c1ae8831" />

#### Step 3: Reproduction on Domains with "da" Prefix
The vulnerability was further verified on multiple target domains with prefix **"da"**.
When sending unauthenticated requests to the protected query endpoints of these domains **without SQL injection payloads**, no data was returned.
<img width="1386" height="614" alt="450e77a8-7503-45d5-8866-863db02fe17c" src="https://github.com/user-attachments/assets/5ed252e6-6389-411e-b608-a7bedbfe93af" />

By sending the same malicious SQL injection payload to these endpoints, the authentication check was successfully bypassed. The server returned valid administrator-level sensitive data, which was not accessible in the unexploited state.
<img width="1398" height="644" alt="96c0bdcc-333c-4588-831a-e2e7ff712532" src="https://github.com/user-attachments/assets/9e807ea7-9648-4dce-8099-cefa06bd881f" />

#### Step 4: Verify Boolean-Based Blind SQL Injection
<img width="1406" height="635" alt="96e9dc16-c12a-421a-b8d5-db8e180d1a40" src="https://github.com/user-attachments/assets/a5ba7283-9cb0-4d47-b0c9-952dc09ea67e" />

The attached screenshot demonstrates the use of a SQL statement to determine whether the number of records in the `DATAGEAR_SCHEMA` table is greater than 1. This confirms the presence of boolean-based blind SQL injection, where the application's response changes based on the truthfulness of the injected SQL condition, even without direct data exfiltration.
<img width="1393" height="634" alt="9840a904-6d27-4f2a-bf13-a12fc45fe83f" src="https://github.com/user-attachments/assets/250c6bac-5d7b-45bf-8797-3454b2311902" />

**5. Impact Analysis**
Privilege Escalation: Successfully bypassed authentication to gain administrator-level access via SQL injection, confirmed through reproduction.
Data Exfiltration: The vulnerability allows unauthorized reading of sensitive enterprise data from the database, including user credentials, personal information, and internal business records.
Data Manipulation: Potential to modify or delete database records (not fully verified in this reproduction, but inherent risk of SQL injection).
Code Execution: Not observed or confirmed in this test; further exploitation may be required to assess this risk.
<img width="1393" height="634" alt="9840a904-6d27-4f2a-bf13-a12fc45fe83f" src="https://github.com/user-attachments/assets/229aee91-d5be-47ac-a5ab-4801a916caca" />

**6. Remediation**
Immediate Fix: Implement parameterized queries (prepared statements) in the authentication API to prevent SQL injection.
Long-term Mitigation: Conduct regular security audits of all API endpoints, implement a Web Application Firewall (WAF), and follow secure coding practices.
