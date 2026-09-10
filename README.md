<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0b1026,45:1f6feb,100:7c3aed&height=185&section=header&text=Mediroza%20Security%20Assessment&fontSize=38&fontColor=ffffff&animation=fadeIn&fontAlignY=35&desc=Networkwalks%20B082%20%7C%20Week%204%20Capstone%20Project&descAlignY=55&descSize=17" alt="Mediroza Security Assessment">

<p align="center">
  <strong>Black-box web application penetration test</strong><br>
  <sub>Authorized educational security assessment</sub>
</p>

<p align="center">
  <img alt="Assessment type: Black box" src="https://img.shields.io/badge/ASSESSMENT-BLACK%20BOX-1f6feb?style=for-the-badge&labelColor=0b1026">
  <img alt="Overall risk: Critical" src="https://img.shields.io/badge/OVERALL%20RISK-CRITICAL-dc2626?style=for-the-badge&labelColor=0b1026">
</p>
<p align="center">
  <img alt="Seven findings" src="https://img.shields.io/badge/FINDINGS-7-f59e0b?style=flat-square&labelColor=111827">
  <img alt="Three critical findings" src="https://img.shields.io/badge/CRITICAL-3-dc2626?style=flat-square&labelColor=111827">
  <img alt="Two high findings" src="https://img.shields.io/badge/HIGH-2-f97316?style=flat-square&labelColor=111827">
  <img alt="Authorized testing" src="https://img.shields.io/badge/STATUS-AUTHORIZED-16a34a?style=flat-square&labelColor=111827">
</p>

 **Overall risk rating: CRITICAL.** This controlled assessment identified an attack path from a public-facing login page to confidential patient documents, staff financial records, and corporate ownership information.

##  Project overview

This repository documents a black-box penetration test of the Mediroza General Hospital web infrastructure at `https://medirozahospital.com`. The assessment was performed for the Networkwalks B082 Week 4 Capstone Project to identify weaknesses, demonstrate their impact through controlled exploitation, and recommend practical remediation.

Seven findings, ranging from **Medium** to **Critical**, were identified. The central issue was a SQL injection vulnerability in the patient portal login flow. In the authorized test environment, it enabled authentication bypass, access to confidential patient lab-report PDFs, discovery of sensitive PDF metadata, and retrieval of an exposed database backup containing staff salary and shareholder information.

##  Objectives

- Assess the web application from an external, unauthenticated perspective.
- Identify weaknesses in authentication, input handling, file protection, and server configuration.
- Demonstrate the real-world impact of each finding in a controlled manner.
- Document evidence and provide prioritized remediation recommendations.

##  Authorization and scope

Testing was conducted with written authorization from the client as part of a controlled Networkwalks educational exercise.

| In scope | Excluded from scope |
| --- | --- |
| `https://medirozahospital.com` | Social engineering |
| Public facing web application behavior | Denial of service testing |
| Patient portal and discovered web paths within the agreed domain | Any testing outside the agreed domain |

##  Methodology

The assessment followed a structured black box methodology:

1. **Reconnaissance** - Passive information gathering using publicly available information and web-based tools.
2. **Vulnerability identification** - Analysis of application behavior for authentication and input-handling weaknesses.
3. **Controlled exploitation** - Demonstration of each issue's impact within the authorized environment.
4. **Documentation** - Recording of findings, evidence, and remediation recommendations.

##  Tools used

| Tool | Purpose in the assessment |
| --- | --- |
| cURL | Sending HTTP requests and reviewing web-server responses |
| Browser Developer Tools | Inspecting page source and login-form behavior |
| Networkwalks Hash Calculator | Extracting PDF password hashes |
| Networkwalks Password Cracker | Testing PDF password hashes against wordlists |
| QPDF | Decrypting password-protected PDFs after password recovery |
| ExifTool | Reading hidden metadata from PDFs |
| Wget | Downloading files from the web server in the controlled test |
| ChatGPT | Converting raw SQL data into readable tables during analysis |

##  Executive summary

The target's security posture was assessed as poor. A chain of individually preventable weaknesses enabled an unauthenticated attacker to progress from reconnaissance to patient-document access and, ultimately, access to a database backup exposed through a publicly listed directory.

The database extract contained sensitive information for **30 hospital employees** and ownership information for **10 shareholders**. To protect privacy, this README intentionally does not reproduce names, contact details, national ID numbers, salary figures, or ownership percentages from that extract.

Immediate remediation is recommended for all Critical and High findings.

##  Findings summary

| ID | Finding | Location | Risk |
| --- | --- | --- | --- |
| F-01 | Username enumeration on login page | `patient/login.php` | 🟡 Medium |
| F-02 | SQL injection login bypass | `patient/login.php` | 🔴 Critical |
| F-03 | Encrypted PDFs accessible after login bypass | `patient/reports/` | 🟠 High |
| F-04 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | 🟠 High |
| F-05 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | 🟡 Medium |
| F-06 | Forgotten backup folder with directory listing enabled | `old/` | 🔴 Critical |
| F-07 | Confidential staff salaries and shareholder data in plain text | `old/mediroza_db_backup_2019.sql` | 🔴 Critical |

##  Detailed findings

### F-01 - Username enumeration on login page

**Risk:** 🟡 Medium  
**Location:** `patient/login.php`

The patient login page returned different error messages for an unknown username and an incorrect password. This behavior allowed the assessor to confirm that a default administrative username existed, reducing the effort required for a password or authentication attack.

**Recommendation:** Return one generic failed-login response for every unsuccessful attempt, such as `Invalid credentials. Please try again.`

**Evidence placeholder:**

<img width="1365" height="734" alt="patient login using sql injection" src="https://github.com/user-attachments/assets/4afdec6d-db5c-4423-b31c-fdceaf2cc9e9" />

<img width="1365" height="736" alt="Tried again using sql injection" src="https://github.com/user-attachments/assets/70bad896-3e47-47a1-9bcf-5c960cc1443f" />

### F-02 - SQL injection login bypass

**Risk:** 🔴 Critical  
**Location:** `patient/login.php`

The login form appeared to place user-controlled input directly into a database query. A controlled test produced a MySQL syntax error, confirming that the field was injectable. The assessment then demonstrated that the password check could be bypassed and an administrative session obtained without valid credentials.

**Recommendation:** Replace dynamic query construction with parameterized queries or prepared statements. This is the most important remediation in this report.

### F-03 - Confidential PDFs accessible after login bypass

**Risk:** 🟠 High  
**Location:** `patient/reports/`

After the controlled authentication bypass, the patient portal exposed three downloadable patient lab-report PDFs. These documents should be accessible only to the named patients and authorized clinicians.

**Recommendation:** Store files outside the web root and deliver them only through server-side authorization checks.

**Evidence placeholder:**
<img width="1365" height="740" alt="Patient logingphp with sql injection sucessful" src="https://github.com/user-attachments/assets/01fff413-bdf7-430a-8e84-d8b139b69c17" />


### F-04 - Weak PDF passwords crackable with a wordlist

**Risk:** 🟠 High  
**Affected assets:** `patient_report_*.pdf`

All three PDFs were password protected, but the passwords were weak and present in commonly available wordlists. Two reports were recovered with the built-in 100-word list, while the third required a larger John the Ripper default password list. Password protection did not provide meaningful security for the medical documents.

**Recommendation:** If passwords are retained, enforce a minimum length of 12 characters with uppercase, lowercase, numbers, and symbols. Access control should remain server-side rather than relying on document passwords alone.

**Evidence placeholder:**
<img width="1351" height="725" alt="patient report" src="https://github.com/user-attachments/assets/875bb9e8-35c3-469f-a70e-0507f6c846f8" />


### F-05 - Sensitive metadata in patient PDF files

**Risk:** 🟡 Medium  
**Affected asset:** `patient_report_3.pdf`

After decryption, metadata analysis identified an internal staff comment that referenced a database backup location on the server. The information was not visible in the document's normal reading view but was extractable with metadata tooling.

**Recommendation:** Remove metadata before distributing files. Internal notes and staff comments must never remain in documents that leave the organization.


### F-06 - Forgotten backup folder with directory listing enabled

**Risk:** 🔴 Critical  
**Location:** `old/`

Reconnaissance identified `/old` in `robots.txt`; the metadata clue confirmed its relevance. The directory had listing enabled and exposed a database backup file to anyone visiting the URL. The backup was downloaded during the authorized test.

**Recommendation:** Disable directory listing (for example, with `Options -Indexes` in Apache configuration or `.htaccess`), remove the backup from the web root immediately, and store backups only in private, access-controlled locations.

<img width="1363" height="733" alt="2" src="https://github.com/user-attachments/assets/640c44e0-897c-4b4b-adde-6a68beddc411" />


### F-07 - Confidential staff salaries and shareholder data in plain text

**Risk:** 🔴 Critical  
**Affected asset:** `old/mediroza_db_backup_2019.sql`

The exposed database backup contained plain-text staff and shareholder information. The staff data included employee names, roles, contact information, national IDs, and monthly salaries; the shareholder data included names, percentages, and share classes. This README deliberately summarizes the exposure rather than reproducing the sensitive records.

**Recommendation:** Remove the publicly accessible backup, protect stored backups with strict access controls, and review backup handling, retention, and encryption practices.

**Evidence placeholder:**


<img width="1365" height="767" alt="mediroza old 1" src="https://github.com/user-attachments/assets/1a52f302-4c07-45db-b439-5450c7d7320e" />
<img width="1363" height="722" alt="mediroza 2" src="https://github.com/user-attachments/assets/4026dad2-5cc6-401d-aaa4-7bf82ea0c0f5" />
<img width="1365" height="733" alt="mediroza 3" src="https://github.com/user-attachments/assets/3d790113-fe36-4645-91b3-cc1fdaaa83c6" />


##  Attack-chain walkthrough

The following sequence shows how the findings combined into a complete exposure path:

1. `robots.txt` revealed three hidden paths: `/patient`, `/staff`, and `/old`.
2. The patient login page disclosed different error messages, confirming a valid administrative username (**F-01**).
3. A malformed username input triggered a database error, confirming injectable input handling (**F-02**).
4. Controlled SQL injection bypassed the login and provided access to the patient portal (**F-02**).
5. Three confidential patient lab-report PDFs were downloaded from the portal (**F-03**).
6. Wordlist attacks recovered the weak PDF passwords; the third report required a larger John the Ripper default list (**F-04**).
7. Metadata analysis of the unlocked third PDF revealed a staff comment pointing to `/old` (**F-05**).
8. Directory listing on `/old` exposed the database backup (**F-06**).
9. The backup exposed plain-text records for 30 employees and 10 shareholders (**F-07**).

```text
Reconnaissance -> Username Enumeration -> SQL Injection -> Portal Access
       -> Patient PDFs -> Weak PDF Passwords -> Metadata Disclosure
       -> Directory Listing -> Exposed Database Backup
```

```mermaid
flowchart LR
    A[Reconnaissance] --> B[Username Enumeration]
    B --> C[SQL Injection]
    C --> D[Patient Portal Access]
    D --> E[Confidential PDFs]
    E --> F[Weak PDF Passwords]
    F --> G[Metadata Disclosure]
    G --> H[Directory Listing]
    H --> I[Exposed Database Backup]

    classDef discovery fill:#dbeafe,stroke:#2563eb,color:#111827
    classDef critical fill:#fee2e2,stroke:#dc2626,color:#111827
    classDef high fill:#ffedd5,stroke:#ea580c,color:#111827
    classDef exposure fill:#f3e8ff,stroke:#7e22ce,color:#111827

    class A,B discovery
    class C,D,H critical
    class E,F,G high
    class I exposure
```

##  Impact

If exploited outside the controlled environment, this chain could enable an unauthenticated attacker to:

- Access confidential patient medical documents.
- Recover weakly protected PDF contents.
- Discover internal infrastructure clues through document metadata.
- Obtain exposed database backups.
- Access sensitive staff financial and personal information.
- Access corporate ownership information.

The combined effect is a serious confidentiality breach with potential privacy, regulatory, financial, and reputational consequences.

##  Remediation priorities

| Priority | Action |
| --- | --- |
| 🔴 Immediate | Replace vulnerable login queries with prepared statements / parameterized queries. |
| 🔴 Immediate | Remove the database backup from the web root and disable directory listing. |
| 🔴 Immediate | Review and revoke any exposed credentials or sessions; assess the scope of potential data exposure. |
| 🟠 High | Move patient PDFs outside the web root and enforce server-side authorization before delivery. |
| 🟠 High | Strengthen PDF-password requirements where passwords remain necessary. |
| 🟡 Medium | Strip PDF metadata before external distribution. |
| 🟡 Medium | Standardize generic failed-login responses to prevent username enumeration. |

##  Evidence gallery

<img width="1365" height="736" alt="Tried again using sql injection" src="https://github.com/user-attachments/assets/ff223e68-6daa-411b-8abb-ee76d887f003" />
<img width="1365" height="734" alt="patient login using sql injection" src="https://github.com/user-attachments/assets/2a6d717f-b960-4a12-b627-05e0bea9cc1e" />
<img width="1365" height="767" alt="patient" src="https://github.com/user-attachments/assets/0778b564-e22e-42bc-9751-b6833a9c5a81" />
<img width="1365" height="740" alt="Patient logingphp with sql injection sucessful" src="https://github.com/user-attachments/assets/69804dac-c4c9-41e5-aa11-b5c4b180ee4a" />
<img width="1365" height="767" alt="patient pdf unlocked" src="https://github.com/user-attachments/assets/a81c4bc3-26d7-4efc-b180-e5b59795e67c" />
<img width="1351" height="725" alt="patient report" src="https://github.com/user-attachments/assets/bbcfc678-4297-4fe8-ac40-9e013b9736d7" />

<img width="1365" height="734" alt="PDF 1" src="https://github.com/user-attachments/assets/2c84cdb2-641a-4976-8faf-32d4eca11a77" />
<img width="1365" height="706" alt="PDF 2" src="https://github.com/user-attachments/assets/505f162a-8f85-4206-bec0-b6564e469fac" />
<img width="1365" height="697" alt="PDF 3" src="https://github.com/user-attachments/assets/63f1d389-230d-4621-8657-4c666e13b638" />

<img width="1365" height="740" alt="PDF FILE 1 CRACKED SUCESSFULLY" src="https://github.com/user-attachments/assets/cefd74f4-690a-4cb0-9ae9-61ab2c05334e" />
<img width="1365" height="736" alt="PDF FILE 2 SUCCESSFULLY CRACKED" src="https://github.com/user-attachments/assets/08f470b4-eae2-43df-8088-5b15ed27f46a" />
<img width="1365" height="718" alt="PDF FILE 3 CRACKED SUCCESSFULLY" src="https://github.com/user-attachments/assets/33cd5f18-7fd1-4f13-9798-8bd49b87bbb3" />


<img width="1351" height="721" alt="Shareholder 1" src="https://github.com/user-attachments/assets/4cb82e2b-d678-49b5-829e-33bc4f4a794f" />
<img width="1365" height="731" alt="Shareholder 2" src="https://github.com/user-attachments/assets/7144581f-eed6-4ae1-80a9-084a800a648d" />
<img width="1365" height="767" alt="Shareholder 3" src="https://github.com/user-attachments/assets/21278fea-0670-4ccb-90bb-8c8d4e2b393c" />
<img width="1365" height="730" alt="Shareholder 4" src="https://github.com/user-attachments/assets/4ff6e7f1-6fdc-4722-82d1-8ec468dcdf7a" />


| Evidence | Suggested file |
| --- | --- |
| Scope and authorization | `docs/evidence/00-authorization-and-scope.png` |
| Username enumeration | `docs/evidence/f01-username-enumeration.png` |
| SQL injection confirmation / controlled bypass | `docs/evidence/f02-sql-injection-login-bypass.png` |
| Patient-report access | `docs/evidence/f03-patient-report-access.png` |
| PDF password assessment | `docs/evidence/f04-weak-pdf-passwords.png` |
| PDF metadata finding | `docs/evidence/f05-sensitive-pdf-metadata.png` |
| Directory-listing exposure | `docs/evidence/f06-directory-listing-backup.png` |
| Redacted database-exposure proof | `docs/evidence/f07-redacted-database-exposure.png` |


##  Full report

[Mediroza Penetration Testing Report.pdf](https://github.com/user-attachments/files/32076421/Mediroza.Penetration.Testing.Report.pdf)

##  Lessons learned

- Small security flaws can combine into a high-impact attack chain.
- Authentication errors and database errors reveal valuable information to attackers.
- Encryption is ineffective when document passwords are weak and easily guessed.
- Document metadata requires the same security review as visible content.
- Backups must never be placed in publicly accessible web directories.
- Defense in depth is essential: secure input handling, authorization, file storage, server configuration, and data governance must all work together.

##  Conclusion

This assessment demonstrated a complete path from the login page to highly sensitive internal data using well-known, preventable weaknesses. Critical and High findings should be addressed immediately before the system is used to store or serve real patient data.

##  Disclaimer

This project was produced as part of a controlled educational exercise by Networkwalks. The target was authorized for security testing, and all activity was performed within the agreed scope. The techniques described here must never be used against systems without explicit written permission from the owner.

## 👤 Credits

- **Author:** Alebiosu Oluwadamilare Samuel
- **Cybersecurity Mentor:** Waqas Karim, CCIE
- **Organization:** Networkwalks
- **Program:** B082 Cybersecurity Internship - Week 4 Capstone Project

---

<p align="center">
  <img alt="For educational and authorized testing only" src="https://img.shields.io/badge/FOR%20EDUCATIONAL%20AND%20AUTHORIZED%20TESTING%20ONLY-0b1026?style=for-the-badge">
</p>

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0b1026,45:1f6feb,100:7c3aed&height=105&section=footer" alt="">
