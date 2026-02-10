# Authenticated SQL Injection (Time-based Blind) in Product Expiry Alert Management System

## 🛡️ Vulnerability Overview
- **Vulnerability Type**: SQL Injection (CWE-89)
- **Affected Component**: `edit-admin.php`
- **Vulnerable Parameter**: `id` (GET method)
- **Software Version**: 1.0
- **Vendor**: SourceCodester
- **Attack Vector**: Remote (Network)
- **Privileges Required**: Low (Authenticated User)
- **Impact**: Database Compromise, Information Disclosure, Privilege Escalation

## 📝 Description
A critical **SQL Injection** vulnerability was discovered in the `edit-admin.php` component of the "Product Expiry Alert Management System". The application allows authenticated users to edit profiles but fails to properly sanitize the `id` GET parameter before using it in a SQL query.

Specifically, the `$id` variable is directly concatenated into the SQL string without using prepared statements or input validation (e.g., `intval()`). This allows an attacker to inject arbitrary SQL commands, leading to a **Time-based Blind SQL Injection**.

Successful exploitation allows an attacker to dump the entire database, including the plaintext credentials of the Super Administrator.

---

## 🛠️ Proof of Concept (PoC)

### 1. Vulnerable Code Analysis
The source code of `edit-admin.php` (Line 17 and Line 37/42) demonstrates the vulnerability. The `id` parameter is taken directly from the URL and spliced into the query:
`$sql = "select * from users where ID='$id'"` and `$sql = "update users set ... where ID='$id'"`.

<img width="1443" height="569" alt="03a47cea987196e40b76c2cac4720d35" src="https://github.com/user-attachments/assets/282af43e-2e80-4940-ae90-480be3ca0fb9" />

### 2. Vulnerability Verification (sqlmap)
Using `sqlmap` with an authenticated session cookie, the `id` parameter is confirmed to be vulnerable. The payload used relies on `SLEEP()` commands to verify the injection.
* **Payload Type**: Boolean-based blind & Time-based blind
* **Command**: `python sqlmap.py -u "http://localhost/product_expiry/edit-admin.php?id=1" --cookie="PHPSESSID=..." -p id --batch --dbs`

<img width="2551" height="1260" alt="7e9abb09c058b2de59b1433183062985" src="https://github.com/user-attachments/assets/cf0e0d77-f3a8-4f86-9daa-7f3f575ea3ab" />

### 3. Database Enumeration
The attacker can successfully enumerate the backend database structure. As shown below, the database `product_expiry_goodness` contains critical tables such as `users`.

<img width="2555" height="1155" alt="18d8fead8e482bc8bd3647b3ec2f5df6" src="https://github.com/user-attachments/assets/a0ab07ef-b4f3-4fc2-a486-a0ec4c2d18c0" />

### 4. Data Exfiltration (Impact)
By exploiting this flaw, the attacker can extract sensitive data from the `users` table. The screenshot below demonstrates the retrieval of the **Super Admin's** details, including:
- **Full Name**: Goodness Monday
- **Email**: newleastpaysolution@gmail.com
- **Password**: (Plaintext credentials exposed)

<img width="2542" height="1424" alt="cff726a6367be83fa84aca23cd3f885c" src="https://github.com/user-attachments/assets/4c4eb86f-27d1-4c8b-a9e8-f39c9c3acdb2" />

---

## 💥 Impact Analysis
- **Confidentiality**: Critical. Attackers can read all data in the database, including admin passwords.
- **Integrity**: High. Since the injection point is also in an `UPDATE` statement logic, attackers could potentially modify data.
- **Availability**: High. Time-based blind injection (e.g., `SLEEP(100)`) can be used to exhaust database connections, causing Denial of Service (DoS).

## 🚀 Remediation
1.  **Use Prepared Statements**: Replace direct query concatenation with `mysqli_prepare` and `bind_param`.
2.  **Input Validation**: Ensure the `id` parameter is cast to an integer using `(int)$_GET['id']` or `intval()`.
3.  **Access Control**: Implement server-side checks to ensure users can only edit their own ID, preventing IDOR combined with SQLi.

## ⚠️ Disclaimer
This report is for educational and ethical testing purposes only. The vulnerability was discovered in a local test environment.
