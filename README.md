# Phantom Corp AD — Vulnerable Active Directory Lab

A single-server, deterministic **vulnerable Active Directory** lab for hands-on offensive-security training and exams. One Windows Server 2016 Domain Controller is seeded with ~40 realistic attack paths — from anonymous enumeration all the way to Domain Admin and persistence.

> **Deterministic build (seed 1337):** every student who imports the OVA — or runs the script — gets the **exact same users, passwords, groups and attack paths**. No randomness between machines.

**Design by Emre Koybaşı**

---

## What's in the box

- `PhantomCorp.ps1` — the build script (run once on a fresh DC)
- `Phantom_AD_Answer_Key.html` — instructor answer key & attack-path map (English)
- `Phantom_AD_Answer_Key_TR.html` — same, in Turkish
- `ovafile/` — the prebuilt, ready-to-import VM appliance (see **Download the OVA** below)

---

## Domain facts

| Item | Value |
|------|-------|
| Domain | `phantom.local` |
| Host | DC1, Windows Server 2016 (build 14393.693) |
| Users | ~50 employees + service/plant accounts |
| Company (documents) | Phantom Corp |
| CA | Enterprise CA on the DC |
| MachineAccountQuota | 10 |

---

## Planted vulnerabilities

**Recon / Initial access**
- Anonymous LDAP bind + null-session SMB share (`\\DC1\Common`)
- Credentials leaked in file shares (runbook, config, password CSV)
- Pre-staged computer with guessable password (`WS-INTAKE$` / `ws-intake`)
- Autologon plaintext credential in the registry (`svc_deploy`)
- Password spraying (predictable default-password pattern)

**Kerberos**
- Kerberoasting (RC4-only SPN accounts, weak passwords)
- AS-REP Roasting (pre-auth disabled)
- Constrained delegation + protocol transition (`websvc`)
- Resource-Based Constrained Delegation (RBCD)
- Unconstrained delegation (enumeration object)

**ACL / privilege escalation**
- Group→group and user→user ACL abuse chains → **DCSync**
- Shadow Credentials (`msDS-KeyCredentialLink` write)
- AdminSDHolder backdoor
- Backup Operators / Server Operators / DnsAdmins membership
- Readable gMSA (`ReadGMSAPassword`)
- GPO abuse on a DC-linked policy

**AD CS (Certified / ESC)**
- ESC1 (enrollee-supplies-subject template)
- ESC4 (weak template ACL)
- ESC6 (`EDITF_ATTRIBUTESUBJECTALTNAME2`)
- ESC7 (ManageCA / ManageCertificates)
- ESC8 (HTTP Web Enrollment relay target)
- Certifried (CVE-2022-26923)

**Coercion / relay**
- Print Spooler (MS-RPRN / PrinterBug)
- PetitPotam (MS-EFSRPC)
- SMB signing off, NTLMv1 enabled, LDAP signing not enforced

**Network CVEs**
- **Zerologon** (CVE-2020-1472) — exploitable on this build
- **EternalBlue** (MS17-010) — exploitable on this build
- HFS 2.3 (CVE-2014-6287) — optional web-app RCE → SYSTEM

**Local privesc & persistence**
- Unquoted service path, AlwaysInstallElevated, writable scheduled task
- GPP cpassword in SYSVOL
- LAPS-style local-admin password in a readable attribute
- DSRM backdoor (`DsrmAdminLogonBehavior=2`)
- Domain Admin service account cached in LSASS (WDigest) + LSA secrets

Full exploit commands for every item are in the **answer keys**.

---

## Option A — Import the prebuilt VM (fastest)

1. Download the OVA (see **Download the OVA** below).
2. VirtualBox → **File → Import Appliance** → select the `.ova`.
3. Boot it. Host-only / internal network recommended. Attack from a Kali VM on the same network.
4. **Take a snapshot before you start** — Zerologon resets the DC machine password.

Everything is already built; just attack.

---

## Option B — Build it yourself (PowerShell)

On a **clean** Windows Server 2016, in an **elevated 64-bit PowerShell** (not the x86 one), in order:

```powershell
# 1) Rename the machine to DC1 (reboots)
Rename-Computer -NewName "DC1" -Restart -Force

# 2) After reboot, confirm
hostname        # -> DC1

# 3) Install AD DS and create the forest (reboots)
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest `
  -DomainName "phantom.local" `
  -DomainNetbiosName "PHANTOM" `
  -InstallDns `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "DSRMpass123!" -AsPlainText -Force) `
  -Force

# 4) (Optional) place a vulnerable HFS 2.3 / 2.3b build at C:\HFS\hfs.exe for CVE-2014-6287

# 5) After reboot, run the lab script once (elevated, 64-bit)
.\PhantomCorp.ps1
```

Notes:
- Run **once** on a clean DC — the build is not idempotent.
- Keep Windows Update **off** so Zerologon/EternalBlue stay in scope.
- The script writes a full cheat sheet to `C:\VulnAD-Build`. **Copy it out, then delete it before sharing the VM.**

---

## Download the OVA

> GitHub limits regular files to 100 MB and release assets to 2 GB, so a multi-GB OVA does **not** fit in the repo directly. Pick one:

- **Split archive on a GitHub Release** (if under ~2 GB per part):
  ```bash
  # split into 1900 MB parts
  zip -s 1900m Phantom2016_lab.zip ovafile/Phantom2016_lab.ova
  # upload the .zip / .z01 / .z02 ... as Release assets, then:
  # zip -s 0 Phantom2016_lab.zip --out joined.zip && unzip joined.zip
  ```
- **External host** (Google Drive / Mega / your own server) and put the link here.

*(Edit this section with your actual download link.)*

---

## Credentials / answer key

Exploit walkthroughs and the planted-credential reference are in:
- `Phantom_AD_Answer_Key.html` (English)
- `Phantom_AD_Answer_Key_TR.html` (Türkçe)

The exact randomized identities for a given build are in `generate.log` / `VulnAD_Lab_Report.txt` (written to `C:\VulnAD-Build` during the build).

---

## Disclaimer

For **authorized training, education and CTF use only**, on systems you own or are permitted to test. The author accepts no liability for misuse. Do not connect this VM to a production or internet-facing network.

**Design by Emre Koybaşı**
