# Active Directory Home Lab

A fully functional Active Directory home lab built on a personal laptop using VirtualBox. Demonstrates core enterprise IT skills including domain setup, user/group management, Group Policy, and Linux domain integration.

---

## Lab Overview

| Component | Details |
|---|---|
| Host OS | Ubuntu 24.04.2 LTS |
| Hypervisor | Oracle VirtualBox |
| Domain Controller | Windows Server 2022 (DC01) |
| Client Machine | Ubuntu 26.04 (UbuntuClient) |
| Domain Name | lab.local |
| DC IP | 192.168.100.10 |
| Client IP | 192.168.100.20 |

---

## Network Diagram

```
┌─────────────────────────────────────────┐
│         VirtualBox Internal Network      │
│                  'adlab'                 │
│                                          │
│  ┌─────────────┐      ┌───────────────┐ │
│  │    DC01     │      │ UbuntuClient  │ │
│  │ 192.168.100 │◄────►│ 192.168.100   │ │
│  │    .10      │      │    .20        │ │
│  │  Win Server │      │  Ubuntu 26.04 │ │
│  │    2022     │      │               │ │
│  └──────┬──────┘      └──────┬────────┘ │
│         │                    │          │
└─────────┼────────────────────┼──────────┘
          │ NAT                │ NAT
          ▼                    ▼
     Internet              Internet
  (updates/downloads)  (package installs)
```

---

## What I Built

### Phase 1 — Windows Server 2022 VM
- Created VirtualBox VM with 4GB RAM, 50GB disk
- Configured two network adapters: Internal Network (`adlab`) and NAT
- Installed Windows Server 2022 Standard Evaluation (Desktop Experience)
- Installed VirtualBox Guest Additions
- Renamed server to `DC01`
- Set static IP `192.168.100.10` via Network Adapter settings

### Phase 2 — Promoted to Domain Controller
- Installed Active Directory Domain Services (AD DS) role via Server Manager
- Promoted server to Domain Controller
- Created new AD forest with root domain `lab.local`
- DNS role installed automatically alongside AD DS

### Phase 3 — AD Objects and Group Policy
- Created Organisational Units (OUs): `IT`, `HR`, `Workstations`
- Created user accounts:
  - `jsmith` (John Smith) in IT OU
  - `sjones` (Sarah Jones) in HR OU
- Created Security Group `IT-Staff` and added jsmith as member
- Created and linked GPO `IT-Policy` to IT OU
- Configured GPO to prohibit access to Control Panel for IT OU users

### Phase 4 — Ubuntu Client Joined to Domain
- Created Ubuntu 26.04 VM with 4GB RAM, 20GB disk
- Configured Internal Network adapter with static IP `192.168.100.20`
- Set DNS to point to DC01 (`192.168.100.10`)
- Installed domain join tools
- Discovered and joined `lab.local` domain
- Verified domain user authentication on Linux

---

## Key Commands

### Windows Server (PowerShell)

```powershell
# Check Group Policy application
gpupdate /force
gpresult /r

# Check DNS resolution
nslookup lab.local 127.0.0.1

# Set static IP on internal adapter
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.100.10 -PrefixLength 24

# Check IP configuration
ipconfig /all

# Check DNS service
Get-Service DNS
```

### Ubuntu Client (Terminal)

```bash
# Install domain join tools
sudo apt install realmd sssd sssd-tools adcli packagekit -y

# Discover the domain
realm discover lab.local

# Join the domain
sudo realm join lab.local -U Administrator

# Verify domain join
realm list

# Check AD user on Linux
id jsmith@lab.local

# Log in as domain user
su - jsmith@lab.local

# Configure DNS
sudo resolvectl dns enp0s3 192.168.100.10
sudo resolvectl domain enp0s3 lab.local

# Set static IP
sudo nmcli con mod "netplan-enp0s3" ipv4.addresses 192.168.100.20/24 ipv4.method manual
sudo nmcli con mod "netplan-enp0s3" ipv4.dns 192.168.100.10
sudo nmcli con up "netplan-enp0s3"

# Restart SSSD service
sudo systemctl restart sssd
```

---

## Key Results

- Domain Controller successfully promoted and serving `lab.local`
- OU structure created mirroring real enterprise layout
- GPO applied to IT OU restricting Control Panel access
- Ubuntu client successfully joined to Windows AD domain
- AD domain user `jsmith` authenticated on Linux with correct group memberships:

```
uid=1766001107(jsmith) gid=1766000513(domain users)
groups=1766000513(domain users),1766001106(it-staff)
```
<img width="874" height="91" alt="image" src="https://github.com/user-attachments/assets/b0809810-d7bc-4179-96dd-db9873b815d6" />


---

## Concepts Demonstrated

- **Active Directory** — domain setup, forest/domain structure
- **DNS** — AD-integrated DNS, name resolution across platforms
- **DHCP vs static IP** — why DCs need static IPs
- **Organisational Units** — logical grouping of AD objects
- **Security Groups** — controlling access and applying policies
- **Group Policy Objects (GPOs)** — enforcing settings across domain users
- **realmd / sssd** — Linux integration with Active Directory
- **Kerberos authentication** — cross-platform single sign-on
- **Network configuration** — VirtualBox networking, static IPs, DNS

---

## Tools Used

| Tool | Purpose |
|---|---|
| VirtualBox | Hypervisor for running VMs |
| Windows Server 2022 | Domain Controller OS |
| Ubuntu 26.04 | Linux client OS |
| Active Directory Users and Computers (ADUC) | Managing AD objects |
| Group Policy Management Console (GPMC) | Creating and linking GPOs |
| realmd | Discovering and joining AD domains from Linux |
| sssd | Authenticating AD users on Linux |
| nmcli | Network configuration on Ubuntu |
| resolvectl | DNS configuration on Ubuntu |
| PowerShell | Windows Server administration |

---

## Next Steps

- [ ] Set up Samba file sharing — shared folder on DC01 accessible from Ubuntu
- [ ] Configure SSH access control using AD groups
- [ ] Grant Domain Admins sudo rights on Ubuntu via sssd
- [ ] Create a second Ubuntu VM to test GPO enforcement on a proper workstation
- [ ] Set up a second Domain Controller (DC02) for redundancy

---

## Author

**Konstantin Mezin**
CompTIA A+ | AZ-900 | London, UK
[LinkedIn](https://linkedin.com/in/konstantin-mezin-8425ab2b3) | [GitHub](https://github.com/konmez) | [Portfolio](https://konmez.github.io/my_portfolio)
