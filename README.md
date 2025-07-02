
# 🧠 HA: FORENSICS — VULNHUB WALKTHROUGH

---

## 📑 TABLE OF CONTENTS
- [🕵️ Overview](#overview)
- [🧰 Tools Used](#tools-used)
- [📦 Step 1: Extracting the OVA Contents](#step-1-extracting-the-ova-contents)
- [🔍 Step 2: Analyzing the Disk with Autopsy](#step-2-analyzing-the-disk-with-autopsy)
- [🖼 Step 3: Finding Flag 1](#step-3-finding-flag-1)
- [🧩 Step 4: Discovering the Clues for Flag 2](#step-4-discovering-the-clues-for-flag-2)
- [🔓 Step 5: Cracking the Password & Finding Flag 2](#step-5-cracking-the-password--finding-flag-2)
- [🐳 Step 6: Finding Flag 3 in Docker Overlay](#step-6-finding-flag-3-in-docker-overlay)
- [🗝 Step 7: Finding Flag 4 (Root)](#step-7-finding-flag-4-root)
- [✅ Conclusion](#conclusion)

---

## 🕵️ OVERVIEW

**Name:** HA: Forensics  
**Type:** Digital Forensics Challenge (OVA)  
**Objective:** Locate and extract **4 hidden flags** from a compromised system snapshot using forensic tools.

Tasks included:
- Extracting the `.vmdk` from an OVA image
- Using forensic tools (Autopsy, FTK Imager) to examine a disk image
- Recovering deleted data and analyzing file artifacts
- Cracking passwords and inspecting embedded metadata

---

## 🧰 TOOLS USED

| Tool               | Purpose                                               |
|--------------------|--------------------------------------------------------|
| 🔍 **Autopsy**      | File system and image analysis                        |
| 💽 **FTK Imager**   | Mounting and exporting content from the OVA           |
| 🔐 **crunch**        | Wordlist generator for brute-force                    |
| 🔓 **fcrackzip**     | Cracking the password-protected ZIP file              |
| 🧾 **Online PGP Tool** | Decrypting PGP-encrypted messages (used manually)   |
| 🖥 **Kali Terminal** | Cracking, scripting, and misc. CLI operations         |

---

## 📦 STEP 1: EXTRACTING THE OVA CONTENTS

Loaded the `.ova` file using **FTK Imager**:

```text
File → Add Evidence Item → Select OVA → Mount as Physical Drive
```

Exported:
- `disk.vmdk` (Virtual Disk)
- `*.ovf` descriptor file
- `*.mf` manifest file

These were then imported into **Autopsy** for analysis.

---

## 🔍 STEP 2: ANALYZING THE DISK WITH AUTOPSY

**Autopsy Workflow:**
1. Create a new case
2. Add data source → `disk.vmdk`
3. Let Autopsy scan and parse metadata, file structures

Target folders:
- `/var/www/html`
- `/home/`
- `/var/lib/docker/`
- `/root/`

---

## 🖼 STEP 3: FINDING FLAG 1

In Autopsy, navigated to:
```
/var/www/html/images/fingerprint.jpg
```

Opened image metadata and found a hidden comment with a flag.

🧾 **Flag 1:**  
```text
{bc02d4ffbeeab9f57c5e03de1098ff31}
```

---

## 🧩 STEP 4: DISCOVERING THE CLUES FOR FLAG 2

Navigated to:
```
/var/www/html/igolder/
```

Discovered a file named `clue.txt`.

📂 **Full Path:**  
`/img_forensics-disk1.vmdk/vol_vol2/var/www/html/igolder/clue.txt`

![Autopsy Screenshot](autopsy.png)

📜 `clue.txt` Contents (PGP key snippet shown):

```
-----BEGIN PGP PRIVATE KEY BLOCK-----
Version: BCPG C# v1.6.1.0

IQOsBF9g7OBCACvH5sL2p9nyVlbVk4yraUmDKbPJNIttUJqcR2d+UDEORbDI
...
OytlTNXi/KpnOt+XfGmjXz+yKt7+82eOsI4/jDFZW64NfijWGeP+9Q8FA
-----END PGP PRIVATE KEY BLOCK-----
```

🧠 Decryption Hint (from the message):
> Password is 6 characters long. Starts with `"for"` and ends with 3 digits.

🖼 **Clue file screenshot**:

![Clue File](clue.png)

---

## 🔓 STEP 5: CRACKING THE PASSWORD & FINDING FLAG 2

```bash
crunch 6 6 -t for%%% -o dict.txt
fcrackzip -u -D -p dict.txt flag.zip
```

✅ Password: `for007` → Opened the ZIP and found a PDF inside.

🧾 **Flag 2:**  
```text
{4a3232c59ecda21ac71bebe3b329bf36}
```

---

## 🐳 STEP 6: FINDING FLAG 3 IN DOCKER OVERLAY

Path found via Autopsy:
```
/var/lib/docker/overlay2/.../diff/var/ftp/pub/saboot.001
```

Extracted `saboot.001`, added to Autopsy, and found `flag3.txt`.

🧾 **Flag 3:**  
```text
{8442460f48338fe60a9497b8e0e9022f}
```

---

## 🗝 STEP 7: FINDING FLAG 4 (ROOT)

Navigated to `/root/root.txt` and revealed the final flag.

🧾 **Flag 4:**  
```text
{9440aee508b6215995219c58c8ba4b45}
```

---

## ✅ CONCLUSION

| Flag | Description                            | Location                              |
|------|----------------------------------------|---------------------------------------|
| 1    | Metadata in fingerprint.jpg            | `/var/www/html/images`                |
| 2    | Cracked password in ZIP > PDF          | `/var/www/html`                       |
| 3    | Found in Docker overlay FTP dump       | `/var/lib/docker/.../saboot.001`      |
| 4    | Plaintext flag in root user directory  | `/root/root.txt`                      |

🎉 All 4 flags captured. Case closed.
