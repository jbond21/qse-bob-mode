# LAB-2883
## Automate Quantum-safe Discovery & Remediation with IBM Quantum Safe Explorer CLI & IBM Bob

**Hands-on lab guide**

---

> References in this document to IBM products, programs, or services does not imply that IBM intends to make such products, programs or services available in all countries in which IBM operates or does business.
>
> Workshops, sessions and associated materials may have been prepared by independent session speakers, and do not necessarily reflect the views of IBM. All materials and discussions are provided for informational purposes only, and are neither intended to, nor shall constitute legal or other guidance or advice to any individual participant or their specific situation.
>
> It is the customer's responsibility to ensure its own compliance with legal requirements and to obtain advice of competent legal counsel as to the identification and interpretation of any relevant laws and regulatory requirements that may affect the customer's business and any actions the customer may need to take to comply with such laws. IBM does not provide legal advice or represent or warrant that its services or products will ensure that the customer follows any law.

### Notices and disclaimers (Continued)

> Questions on the capabilities of non-IBM products should be addressed to the suppliers of those products. IBM does not warrant the quality of any third-party products, or the ability of any such third-party products to interoperate with IBM's products. IBM expressly disclaims all warranties, expressed or implied, including but not limited to, the implied warranties of merchantability and fitness for a purpose.
>
> The provision of the information contained herein is not intended to, and does not, grant any right or license under any IBM patents, copyrights, trademarks or other intellectual property right.
>
> IBM, the IBM logo, and ibm.com are trademarks of International Business Machines Corporation, registered in many jurisdictions worldwide. Other product and service names might be trademarks of IBM or other companies. A current list of IBM trademarks is available on the Web at "Copyright and trademark information" at: www.ibm.com/legal/copytrade.shtml.

---

## 1  Introduction

As quantum computing advances, cryptographic algorithms that protect data today — including RSA, DSA, ECDSA, DES, 3DES, RC4, and MD5 — will become vulnerable to attack. IBM Quantum Safe Explorer CLI (QSE) is a developer tool that scans source code to identify all cryptographic API usage, flags vulnerabilities against NIST and FIPS 140-3 standards, and guides developers toward quantum-safe replacements.

In this lab you will use QSE directly inside IBM Bob (VS Code) to scan a multi-language sample codebase, interpret the findings, and apply cryptographic remediations to Python source files — replacing weak and quantum-vulnerable algorithms with approved, NIST-compliant alternatives.

### 1.1 About this hands-on lab

By the end of this lab you will be able to:

- Run a Quantum Safe Explorer API CLI directory scan from within IBM Bob on a multi-language codebase (Go, C++, C#, Python, Java).
- Interpret QSE scan results including the dashboard, findings JSON, and crypto inventory reports.
- Identify high-severity vulnerabilities: broken hash algorithms (MD5, SHA-1), weak symmetric ciphers (DES, 3DES, RC4, Blowfish), insecure cipher modes (ECB), weak key sizes, non-quantum-safe asymmetric algorithms (RSA, DSA, ECDSA), insecure random number generation, and hardcoded secrets.
- Remediate all flagged cryptographic vulnerabilities in Python source code per NIST SP 800-131A Rev.2, NIST SP 800-175B Rev.1, FIPS 140-3, and OWASP Cryptographic Storage Cheat Sheet.
- Understand the post-quantum migration roadmap: FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), and FIPS 205 (SLH-DSA).

**Total estimated time: 90 minutes   |   Hands-on activity time: 75 minutes**

---

## 2  Lab Environment and Prerequisites

Before starting the lab tasks, confirm the following are available in your environment:

- Instructor Provided Techzone VM IBM Bob (VS Code-based IDE) is open

---

## Part 0 — GitHub Setup

**Full GitHub Setup: Fork + Pull Request Workflow**

If your GitHub environment allows repository forking, fork the lab repository and create a dedicated feature branch. All remediation changes will be committed to this branch, and you will open a pull request at the end of the lab to experience IBM Bob's pull request workflow.

**Alternative Setup: Clone the Repository**

If forking is unavailable or restricted in your GitHub environment, clone the lab repository.

---

### Full GitHub Setup

#### 0.1  Fork the Lab Repository

1. Navigate to the repository URL: <https://github.com/jbond21/qse-bob-mode>
2. Click the **Fork** button in the top-right corner
3. Select your account or organization as the destination
4. Wait for the fork to complete

#### 0.2  Create a GitHub PAT for your fork

You need a GitHub Personal Access Token (PAT) to authenticate when you `git push` your changes to your fork. To do this, you will create a fine-grained token limited to your fork, so it can't touch any of your other repositories.

1. Go to <https://github.com/settings/personal-access-tokens/new> (or **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**)
2. Set:
   - **Token name:** `techxchange_2026_workshop`
   - **Resource owner:** your own account
   - **Expiration:** 7 days
   - **Repository access:** Only select repositories → choose your fork (`qse-bob-mode`)
3. Under **Permissions → Repository permissions**, click the **Add permissions** button and select **Contents**.
4. Ensure **Contents** is set to **Read and write**. Leave everything else as **No access** (Metadata: Read-only is added automatically).
5. Click **Generate token**.
6. Copy the token now — GitHub won't show it to you again. Save it somewhere secure (a password manager is recommended).

> **Fine-grained tokens unavailable?** If your account or organization doesn't allow them, create a classic token instead: Generate new token (classic) at <https://github.com/settings/tokens>, check only `repo`, and set a 7-day expiration. Classic tokens cover all of your repositories, so revoke it as soon as the workshop ends.

#### 0.3  Clone your fork

You will clone your fork to the Bob IDE desktop machine.

```bash
git clone https://github.com/jbond21/qse-bob-mode.git
```

Open IBM Bob → File → Open → Desktop → Create and open new folder (the new folder might be restricted; if so, click **Restricted Mode** at the bottom left of the screen to unrestrict it) → Terminal → New Terminal

#### 0.4  Open QSE-BOB-MODE folder in Bob IDE

1. Open the cloned repository by using the menu options: **File → Open Folder**, select the **QSE-BOB-MODE**, and click **Open**
2. Verify Bob can see the repository files in the Explorer sidebar

#### 0.5  Configure Git (if needed)

1. If this is your first time using Git on this machine, configure your identity.
2. You can run the following commands from an existing terminal, or open a terminal directly within the IBM Bob IDE:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

3. Each `git push` in the labs asks for your GitHub username and your PAT (as the password). To avoid re-entering the PAT on every push, run this once before your first push:

```bash
git config --global credential.helper 'cache --timeout=10800'
```

4. Git will now remember your credentials for 3 hours. You enter the PAT on your first push, and later pushes won't ask again.

   > **Note:** The cache is held in memory only. Nothing is written to disk. It's lost on reboot or after 3 hours, and you'll just be prompted for the PAT again.
   >
   > **Prefer not to cache?** Skip the command above. Git will then ask for your username and PAT on every `git push` (about 7–8 times across the labs), and you paste the PAT as the password each time.

5. Next, you will create your working branch, where all your updates will be committed. Run the following commands:

```bash
# Create your branch and push it
git checkout -b user1-labs
git push -u origin user1-labs
```

---

### Alternative Setup

1. Open IBM Bob → File → Open → Desktop → Create and open new folder (the new folder might be restricted; if so, click **Restricted Mode** at the bottom left of the screen to unrestrict it) → Terminal → New Terminal
2. `git clone https://github.com/jbond21/qse-bob-mode.git`
3. Open the cloned repository by using the menu options: **File → Open Folder**, select the **QSE-BOB-MODE**, and click **Open**
4. Verify Bob can see the repository files in the Explorer sidebar

---

## Task 1 — Scanning the Codebase with Quantum Safe Explorer CLI

In this task you will trigger a Quantum Safe Explorer scan of the `sample_files/` directory directly using IBM Bob. The scan analyses all supported languages in the directory (Go, C++, C#, Python, Java) and writes its results to the `qs_scan_result/` folder.

### 1  Run the QSE API Directory Scan

1. Before prompting Bob, switch to the **Agent Mode** if not already. Click the mode selector at the bottom of the Bob chat panel (it shows the current mode name).
2. Prompt IBM Bob (Copy and Paste into prompt box):

```
Please run the Quantum Safe Explorer scan on the sample files:

1. Run: qse -i ./sample_files -o ./qs_explorer_result -l .go,.java,.py,.cpp,.cs
2. Write to workspace root. If qs_explorer_result/qs_scan_result.zip exists, unzip it to ./qs_scan_result/
3. Verify that both qs_explorer_result/ and qs_scan_result/ exist, and give me a brief 3-sentence summary of the total findings. Do not print detailed line-by-line breakdowns.
```

3. Bob may start a TODO list — if so, approve the action. As IBM Bob is scanning with QSE, he will ask you to approve a series of execute commands. Approve them to continue until Bob provides a summary of findings.
4. When complete, confirm the following output directory has been populated in the workspace root:
   - `qs_explorer_result/quantum_safe_api_discovery_cbom.json`
   - `qs_explorer_result/quantum_safe_api_discovery_crypto_inventory.json`
   - `qs_explorer_result/quantum_safe_api_discovery_findings.json`
   - `qs_explorer_result/quantum_safe_api_discovery_report.json`

### 2  Understanding the Scan Output Structure

The scan produces output organized by language under `qs_explorer_result/`. The key files are:

- **`report.json`** — A summary of all crypto API calls found, grouped by language. Contains statistics (lines scanned, files scanned, crypto calls) and a flat list of all findings with function name, library, and source location.
- **`findings.json`** — The full detailed findings for every crypto artifact detected. Each entry includes the crypto properties (algorithm name, primitive, OID, block size), vulnerability details (CWE classification, severity score, debug message), compliance status (FIPS 140-3-L1, Quantum Safe), and precise source location (file, line, column).
- **`cbom.json`** — A Cryptography Bill of Materials (CBOM) in CycloneDX format cataloguing every cryptographic asset found.
- **`discovery_crypto_inventory.json`** — Lists every crypto function call by language and source file, with its library and line/column locations.
- In the `qs_scan_result` folder: **`metrics/metrics-summary.json`** — Code metrics per file: total lines, active lines, methods defined, method calls, and variables.

---

## Task 2 — Reviewing the Scan Results

The QSE scan of the `sample_files/` directory produced 185 crypto artifacts across 7 source files in 4 languages. In this task you will ask Bob to review the findings and describe the vulnerabilities found.

### 1  Switch Bob to Quantum Safe Mode

Before prompting Bob, switch to the **Quantum Safe** mode. This mode is optimized for cryptographic vulnerability analysis and remediation — it knows the NIST PQC standards, CWE classifications, and approved replacement algorithms out of the box.

1. Click the mode selector at the bottom of the Bob chat panel (it shows the current mode name).
2. Select **Quantum Safe** from the list.

### 2  Prompt Bob to Describe the Vulnerabilities

With the scan complete and Quantum Safe mode active, TYPE the following prompt in the Bob chat panel. Note: use `@` files or path to point Bob directly to what you want to evaluate:

```
Look at the information in @qs_explorer_result/ and describe the vulnerabilities found
```

Bob will ask for approvals to read both result directories, parse the files, and return a full breakdown of every vulnerability found — grouped by language, severity, CWE classification, and compliance status.

### 3  Scan Summary

Review Bob's response will vary, but should identify in more detail the 214 crypto artifacts across the following files:

| Language | Files Scanned | Crypto API Calls | Key Algorithms Found |
|----------|:---:|:---:|---|
| Go | 2 | 47 | MD5, SHA-1, RSA, DSA, ECDSA, DES, 3DES, RC4, AES |
| C++ | 1 | 45 | MD5, SHA-1, RSA, DSA, ECDSA, DES, 3DES, RC4, AES (via OpenSSL) |
| C# | 2 | 66 | MD5, SHA-1, HMAC-MD5, HMAC-SHA1, RSA, DSA, ECDSA, DES, 3DES, RC2, AES |
| Python | 2 | 56 | MD5, SHA-1, RSA, DSA, ECC, DES, 3DES, RC4, Blowfish, AES-ECB, HMAC-MD5 |
| Java | 2 | 0 | No crypto calls detected — source-only scan; compiled `.class` files required for Java findings (`-cf` flag) |

### 4  Vulnerability Categories

Bob's response may vary but will organize findings into vulnerability categories, show where the findings are, and/or provide remediation guidance. Review each one in its output.

**Key outputs to look for:**

**MD5 and SHA-1** are cryptographically broken due to known collision attacks. They are not approved by NIST SP 800-175B Rev.1 for any security purpose and fail FIPS 140-3-L1 compliance. Flagged debug message: *"Use of broken or risky cryptographic algorithm: MD5"* and *"Use of weak hash algorithm: MD5"*.

**Quantum-Vulnerable Public Key Algorithms (CWE-327 — High)**

RSA, DSA, and ECDSA are all vulnerable to Shor's algorithm on a cryptographically-relevant quantum computer. QSE flags these as "Not Quantum Safe" in the `complianceLevels` array. Flagged debug message: *"Use of non-quantum-resistant public-key algorithms (must be ML-KEM, ML-DSA, SLH-DSA)"*. Additionally, RSA keys smaller than 2048 bits trigger a CWE-326 (inadequate key strength) finding.

**Weak Symmetric Encryption (CWE-327 — High)**

DES (56-bit key), 3DES (deprecated by NIST SP 800-131A Rev.2 after 2023), RC4 (prohibited by RFC 7465), RC2, and Blowfish are all prohibited or non-standard symmetric algorithms. All fail FIPS 140-3-L1 compliance.

**ECB Cipher Mode (CWE-327 — High)**

AES or DES used in ECB mode is deterministic — identical plaintext blocks always produce identical ciphertext blocks, leaking data patterns. NIST SP 800-38A requires an IV-based mode. Use AES-GCM (authenticated encryption) instead.

### 5  Compliance Status

Each finding Bob reports includes two compliance checks. Review these in Bob's output:

- **`fips140-3-l1`** — Indicates whether the algorithm is FIPS 140-3 Level 1 approved. Broken ciphers (MD5, SHA-1, DES, RC4) are marked non-compliant. RSA is marked "FIPS capable" only if approved key sizes (2048, 3072, 4096 bits) are used.
- **`quantum_safe`** — Indicates whether the algorithm is safe against quantum attack. All asymmetric algorithms (RSA, DSA, ECDSA, ECC) are marked "Not Quantum Safe". Symmetric algorithms and hashes are marked "Not Applicable" (quantum safety only applies to asymmetric primitives at this classification level).

---

## Task 3 — Remediating Python Cryptographic Vulnerabilities

The scan identified crypto API findings across the two Python sample files. Rather than fixing each issue manually, you will prompt Bob to remediate the Python files — exactly as you would in a real development workflow.

**Files to remediate:**

- `sample_files/python_files/vulnerable_crypto.py` — 38 findings covering weak hashes, weak symmetric ciphers, weak asymmetric keys, insecure random generation, hardcoded secrets, and insufficient key lengths.
- `sample_files/python_files/web_crypto_vulnerabilities.py` — 18 findings covering Django/Flask web framework patterns: password hashing, session tokens, JWT, API keys, cookies, database encryption, password reset tokens, file encryption, 2FA, and OAuth.

### 1  Prompt Bob to Remediate the Python Files

Remain in **Quantum Safe** mode. In the Bob chat panel, enter the following prompt:

```
Lets remediate only the python files
```

Bob will automatically: (1) read both Python source files in full, (2) cross-reference the QSE findings for each function, (3) apply fixes for every vulnerability category, (4) add inline comments citing the relevant CWE, NIST standard, and OWASP reference for each fix, and (5) report a complete summary of every change made.

> Bob may try to open a pull request when complete, if so **REJECT** the request.

> **TIP:** Bob applies the minimal correct change for each vulnerability — it does not restructure or rewrite code beyond what is required to remediate the specific finding.

### 2  What Bob Fixes — Category Reference

Use this section to verify Bob's output covers all expected vulnerability classes.

#### Category 1 — Replace Broken Hash Algorithms (MD5, SHA-1)

1. For `weak_password_hash_md5()` and `weak_password_hash_sha1()`, replace the bare MD5/SHA-1 digest with `hashlib.pbkdf2_hmac('sha256', password, salt, 600_000)`. Generate a random salt using `secrets.token_bytes(32)` and return `salt.hex() + ":" + derived_key.hex()`.
2. In `web_crypto_vulnerabilities.py`, apply the same fix to `InsecureDjangoPasswordHasher.hash_password_md5()` and `hash_password_sha1()`. Also replace the `random.randint()` salt with `secrets.token_bytes(32)`.
3. Replace all remaining `hashlib.md5()` / `hashlib.sha1()` calls (API key generation, reset tokens, client secrets, file hashing, sensitive data hashing) with `hashlib.sha256()` or `secrets.token_urlsafe(32)`.
4. Replace all `hmac.new(..., hashlib.md5)` and `hmac.new(..., hashlib.sha1)` with `hmac.new(..., hashlib.sha256)`.

> **Standards:** NIST SP 800-175B Rev.1 (MD5/SHA-1 not approved) · NIST SP 800-131A Rev.2 (SHA-1 deprecated for security applications) · OWASP Password Storage Cheat Sheet (use memory-hard KDF)

#### Category 2 — Replace Weak Symmetric Ciphers and ECB Mode

Replace all uses of DES, 3DES, RC4, RC2, and Blowfish with AES-256-GCM. Also replace all ECB mode usage (AES or DES in `MODE_ECB`) with GCM. AES-GCM is an AEAD (Authenticated Encryption with Associated Data) mode — it provides both confidentiality and integrity in a single operation.

1. Remove imports of `DES`, `DES3`, `ARC4`, and `Blowfish` from the `Crypto.Cipher` imports line.
2. For each cipher function, replace the cipher construction with: `aes_key = get_random_bytes(32)` then `cipher = AES.new(aes_key, AES.MODE_GCM)`
3. Replace `cipher.encrypt(padded_text)` with: `ciphertext, tag = cipher.encrypt_and_digest(plaintext.encode())`
4. Return `base64.b64encode(cipher.nonce + tag + ciphertext).decode()` — packing the 16-byte GCM nonce, the 16-byte authentication tag, and the ciphertext together so the receiver can split them for decryption.
5. For the static IV vulnerability (`encrypt_with_static_iv`), remove the `static_iv = b'\x00' * 16` line entirely. `AES.new()` in `MODE_GCM` automatically generates a random nonce — no IV needs to be supplied.

> **Standards:** NIST SP 800-131A Rev.2 (DES/3DES prohibited) · RFC 7465 (RC4 prohibited) · NIST SP 800-38D (AES-GCM) · NIST SP 800-38A (IV-based modes required — ECB prohibited)

#### Category 3 — Address Quantum-Vulnerable Asymmetric Algorithms

RSA, DSA, and ECDSA are all vulnerable to Shor's algorithm on a quantum computer. The NIST Post-Quantum Cryptography standards (FIPS 203 ML-KEM, FIPS 204 ML-DSA, FIPS 205 SLH-DSA), finalised in August 2024, are the long-term replacements. As an interim measure while PQC library support matures, upgrade all RSA key sizes to a minimum of 3072 bits per NIST SP 800-131A Rev.2.

1. Replace `RSA.generate(512)` and `RSA.generate(1024)` with `RSA.generate(3072)`. These key sizes are prohibited by NIST SP 800-131A Rev.2 (disallowed after 2013).
2. Replace `RSA.generate(2048)` with `RSA.generate(3072)` to give a stronger classical security margin.
3. Replace `DSA.generate(1024)` and `ECC.generate(curve=...)` with `RSA.generate(3072)` as an interim measure. Remove DSA and ECC imports.
4. Replace DSS-based ECDSA/DSA signatures with RSA-PSS + SHA-256 using `Crypto.Signature.pss.new(private_key).sign(hash)`.
5. Add a `TODO` comment on each asymmetric function noting the required migration path: FIPS 203 (ML-KEM) for key encapsulation and FIPS 204 (ML-DSA) or FIPS 205 (SLH-DSA) for digital signatures.

> **Standards:** NIST SP 800-131A Rev.2 (RSA < 2048 prohibited) · NIST SP 800-57 Part 1 Rev.5 (RSA-3072 = 128-bit security) · NIST FIPS 203/204/205 (PQC standards, August 2024)

#### Category 4 — Replace Insecure Random Number Generation

Python's `random` module is a Mersenne Twister PRNG — it is not cryptographically secure and must never be used for security-sensitive values (keys, tokens, salts, nonces, OTP codes). Replace all uses with the `secrets` module.

1. Remove `import random` and `import time` from all files.
2. Replace `random.randint(0, N)` with `secrets.randbelow(N)`.
3. Replace `random.seed(...) + random.randint()` key generation with `secrets.token_bytes(32)`.
4. Replace all session token, API key, reset token, TOTP secret, backup code, and OAuth state generation using `random` with `secrets.token_urlsafe(32)` or `secrets.token_hex(32)`.

> **Standards:** NIST SP 800-90A Rev.1 (approved DRBG algorithms) · OWASP Cryptographic Storage Cheat Sheet

#### Category 5 — Remove Hardcoded Secrets

Hardcoded encryption keys, IVs, JWT secrets, and passwords in source code are CWE-321/CWE-798 violations. They are visible in version control history and must be loaded from the environment at runtime.

1. Add `import os` to the imports section.
2. Replace `HARDCODED_KEY = b'SuperSecretKey16'` with:
   ```python
   _raw_key = os.environ.get('CRYPTO_KEY')
   HARDCODED_KEY = bytes.fromhex(_raw_key) if _raw_key else get_random_bytes(32)
   ```
3. Apply the same pattern to `HARDCODED_IV`, `SECRET_KEY`, `JWT_SECRET`, and `DB_ENCRYPTION_KEY` — each loaded from a named environment variable with a securely-generated fallback.
4. Change the AES mode for `encrypt_with_hardcoded_key()` from `MODE_CBC` (with the hardcoded IV) to `MODE_GCM`. GCM auto-generates a fresh random nonce per call, eliminating the IV reuse vulnerability.

> **Standards:** CWE-321 (hardcoded cryptographic key) · CWE-798 (hardcoded credentials) · OWASP Secrets Management Cheat Sheet

#### Category 6 — Additional Web-Framework Fixes (`web_crypto_vulnerabilities.py`)

The following additional fixes apply only to `web_crypto_vulnerabilities.py`:

- **Timing attacks** — Replace all `==` comparisons on secrets (passwords, tokens, 2FA codes) with `hmac.compare_digest()` which is guaranteed constant-time.
- **JWT 'none' algorithm** — Replace the manually constructed unsigned JWT with `jwt.encode(payload, JWT_SECRET, algorithm='HS256')`. Add an `exp` claim of `utcnow() + timedelta(hours=1)` to all JWT payloads.
- **Insecure cookie flags** — Change `httponly=False` to `True`, `secure=False` to `True`, and `samesite='None'` to `'Strict'`.
- **XOR encryption** — Replace the XOR field encryption loop with AES-256-GCM.
- **base64 as encryption** — Replace `base64.b64encode()` used as 'encryption' in `encrypt_file_weak_key()` and `encrypt_cookie_weak()` with AES-256-GCM.

> **Standards:** OWASP Authentication Cheat Sheet (constant-time comparison) · RFC 7519 (JWT `exp` claim) · OWASP Session Management Cheat Sheet (cookie flags) · RFC 6749 §10.12 (OAuth state)

---

## Task 4 — Verifying the Remediation

After applying all fixes, verify your changes are correct before re-scanning. Note: IBM Bob will perform verifications on its own, here is a manual way to check as well. You can skip to step 3 or run manually.

1. Run a quick syntax check on both Python files from the Bob terminal:

```bash
python3 -m py_compile sample_files/python_files/vulnerable_crypto.py && echo OK
python3 -m py_compile sample_files/python_files/web_crypto_vulnerabilities.py && echo OK
```

2. Confirm no broken imports remain:

```bash
grep -n 'import DES\|import ARC4\|import Blowfish\|import DSA\|import ECC\|import random' sample_files/python_files/*.py
```

This should return no output.

3. Confirm no weak algorithms remain:

```bash
grep -n 'hashlib.md5\|hashlib.sha1\|DES.new\|ARC4\|Blowfish\|MODE_ECB\|RSA.generate(5\|RSA.generate(1\|RSA.generate(2\|DSA.generate\|ECC.generate\|random.seed\|random.randint' sample_files/python_files/*.py
```

This should return no output.

4. Re-run the QSE scan. Switch Bob to **Agent Mode**.

   Prompt Bob to: `Re-run QSE scan to confirm the findings are cleared.`

The Python file sections should show a **63% reduction** in vulnerability instances (52 → 19). All broken algorithms (MD5, SHA-1, DES, 3DES, RC4, Blowfish), ECB-mode ciphers, and static IVs should be fully cleared.

The **19 remaining findings** are expected and intentional — they fall into three categories:

- **Quantum-vulnerable public-key algorithms** (RSA, DSA, ECDSA) — kept as transitional classical algorithms with TODO comments referencing NIST FIPS 203/204/205 migration to ML-KEM/ML-DSA when `oqs-python` support is available.
- **Key size advisories** — QSE flags RSA/DSA keys against its >1024-bit heuristic; the remediated code already meets the NIST SP 800-131A Rev.2 minimum of 3072 bits.
- **Unsalted SHA-256** — QSE flags SHA-256 used without a salt; these are general-purpose hash calls (not password hashing), where a salt is not required. Password hashing functions were upgraded to PBKDF2-SHA256 with a random salt.

None of these 19 are regressions. If you see any findings outside these three categories, a fix may have been missed — re-run Task 3.

---

## Task 5 — Creating a Pull Request for the Remediated Python Files

With the Python cryptographic vulnerabilities remediated and the fixes verified, the final step is to commit your changes to your feature branch and open a pull request so that the fixes can be reviewed and merged into the main branch.

### 1  Stage and Commit the Remediated Files

1. In the Bob terminal, confirm the files that were changed:

```bash
git status
```

You should see both Python files listed under "Changes not staged for commit":

```
        modified:   sample_files/python_files/vulnerable_crypto.py
        modified:   sample_files/python_files/web_crypto_vulnerabilities.py
```

2. Stage only the remediated Python files (do not stage the QSE scan output directories):

```bash
git add sample_files/python_files/vulnerable_crypto.py sample_files/python_files/web_crypto_vulnerabilities.py
```

3. Commit with a descriptive message:

```bash
git commit -m "fix: remediate Python cryptographic vulnerabilities (QSE Lab 2883)

- Replace MD5/SHA-1 with SHA-256 and PBKDF2-SHA256 for password hashing
- Replace DES/3DES/RC4/Blowfish/ECB with AES-256-GCM
- Upgrade RSA key sizes to 3072-bit minimum
- Replace random module with secrets module
- Remove hardcoded keys and IVs; load from environment
- Fix timing attack comparisons, JWT 'none' algorithm, insecure cookie flags"
```

> Update the commit message prefix (`fix:`) to match your team's commit message convention (e.g. Conventional Commits, JIRA ticket prefix, or custom format).

### 2  Push the Branch to the Remote

1. Push your feature branch to the remote repository:

```bash
git push origin studentxx-qse
```

Confirm the push succeeded. GitHub will print a URL to create a pull request directly from the terminal output.

### 3  Create the Pull Request with Bob

1. Switch Bob to **Agent** mode. Click the mode selector at the bottom of the Bob chat panel and select **Agent**.
2. Enter the following prompt in the Bob chat panel:

```
Create a pull request for my studentxx-qse branch (replace xx with your assigned student number). Base branch is main. Title: "fix: Remediate Python cryptographic vulnerabilities (QSE Lab 2883)". In the description summarize the fixes made, include the total Python finding count (56), the files changed, and the CWE/NIST standards addressed.
```

3. Bob will generate the PR description from the git diff and open the pull request directly to your fork. Approve any actions Bob requests to proceed.
4. Share the pull request URL with your lab instructor to confirm completion of the lab.

---

## 7  Post-Quantum Migration Roadmap

The interim RSA-3072 fixes applied in this lab address the immediate classical security concerns but do not eliminate the quantum risk. The table below summarises the full migration path for each algorithm category:

| Current Algorithm | Quantum Threat | NIST-Approved PQC Replacement |
|---|---|---|
| RSA (key encapsulation / key exchange) | Shor's algorithm breaks in polynomial time | ML-KEM (FIPS 203) — Module Lattice Key Encapsulation Mechanism |
| RSA / DSA / ECDSA (digital signatures) | Shor's algorithm breaks in polynomial time | ML-DSA (FIPS 204) or SLH-DSA (FIPS 205) |
| AES-128 (symmetric) | Grover's algorithm halves effective key strength to 64 bits | AES-256 (doubles Grover attack cost back to 128-bit security) |
| SHA-256 (hashing) | Grover's algorithm reduces collision resistance to 128 bits | SHA-256 remains acceptable; SHA-384 or SHA-512 for higher assurance |

The NIST PQC standards (FIPS 203, 204, 205) were finalised on 13 August 2024. Python support is available via the `oqs-python` bindings to `liboqs`. Monitor <https://csrc.nist.gov/projects/post-quantum-cryptography> for library readiness guidance.

---

## 8  Feedback

We value your feedback! Please take a few minutes to complete the TechXchange 2026 lab survey using the QR code or link provided by your lab instructor. Your feedback helps us improve future sessions.

---

## 9  Getting Help and Troubleshooting

### 9.1  Reach out to the Lab Instructor

If you are attending a proctored session, raise your hand or approach the instructor station. Lab instructors are available throughout the session.

### 9.2  Common Troubleshooting Tips

- **QSE Command Palette entry is missing** — Open Extensions (`Cmd + Shift + X`), search for Quantum Safe, and verify the extension is enabled. Reload the window (`Cmd + Shift + P` → `Developer: Reload Window`).
- **Scan produces no output** — Ensure the directory you selected contains supported source files (`.go`, `.cpp`, `.cs`, `.py`, `.java`). The scanner requires at least one recognised file extension.
- **Python syntax error after edits** — Run `python3 -m py_compile <file.py>` to identify the exact line. Common causes: missing closing parenthesis after refactoring `encrypt_and_digest()`, or leaving a stale `pad()` call that is no longer needed with GCM mode.
- **`ModuleNotFoundError: No module named 'Crypto'`** — Run `pip install pycryptodome` in the Bob terminal. Do **not** install `pycrypto` (the older, unmaintained fork).

### 9.3  Getting Help (Self-Paced)

If you are completing this lab in the self-paced room after the session, you can continue using the lab environment for the duration of the TechXchange 2026 event. IBM documentation for Quantum Safe Explorer is available at <https://www.ibm.com/docs/quantum-safe-explorer>.

---

## 10  Lab Reference Materials

- IBM Quantum Safe Explorer documentation — <https://www.ibm.com/docs/quantum-safe-explorer>
- NIST Post-Quantum Cryptography standards (FIPS 203/204/205) — <https://csrc.nist.gov/projects/post-quantum-cryptography>
- NIST SP 800-131A Rev.2 — Transitioning the Use of Cryptographic Algorithms and Key Lengths — <https://csrc.nist.gov/publications/detail/sp/800-131a/rev-2/final>
- NIST SP 800-175B Rev.1 — Guideline for Using Cryptographic Standards in the Federal Government — <https://csrc.nist.gov/publications/detail/sp/800-175b/rev-1/final>
- NIST SP 800-38D — Recommendation for Block Cipher Modes: GCM — <https://csrc.nist.gov/publications/detail/sp/800-38d/final>
- OWASP Cryptographic Storage Cheat Sheet — <https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>
- OWASP Password Storage Cheat Sheet — <https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html>
- CWE-327 (Use of a Broken or Risky Cryptographic Algorithm) — <https://cwe.mitre.org/data/definitions/327.html>
- TechXchange 2026 lab download site — link provided by your instructor

---

*IBM TechXchange 2026  |  Lab 2883: Quantum Safe Code Remediation  |  © 2026 IBM Corporation*
