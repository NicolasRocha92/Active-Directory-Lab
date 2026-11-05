# Active Directory: User Management, Group Policy & Security Auditing

## Project overview
Built an Active Directory lab using Microsoft Azure virtual machines to practice domain setup, DNS configuration, user administration, account lockout policies, account recovery, and Windows security event analysis.

**Environment:** Microsoft Azure, Windows Server 2022 (DC-1), Windows client (Client-1), Active Directory Domain Services, DNS, Group Policy Management, PowerShell, Remote Desktop, Event Viewer.

**Lab domain:** `mydomain.com`

## Objectives
- Deploy a domain controller and domain-joined client in Azure.
- Configure DNS and test connectivity.
- Manage domain users and organizational units.
- Apply and verify account lockout policy.
- Practice password resets, account disabling, and account recovery.
- Inspect Windows Security event logs.

## Implementation and screenshots

> Screenshots are original lab evidence. Image filenames retain their original step numbers; Step 21 was omitted because it duplicated Step 18.

### Azure infrastructure and domain connectivity

**Step 01 — Created Azure resource groups for the Active Directory lab.**

![Step 01: Created Azure resource groups for the Active Directory lab.](images/Lab-5-Step-01.png)

**Step 02 — Created the virtual network and subnet.**

![Step 02: Created the virtual network and subnet.](images/Lab-5-Step-02.png)

**Step 03 — Reviewed the virtual network configuration.**

![Step 03: Reviewed the virtual network configuration.](images/Lab-5-Step-03.png)

**Step 04 — Deployed the DC-1 Windows Server virtual machine.**

![Step 04: Deployed the DC-1 Windows Server virtual machine.](images/Lab-5-Step-04.png)

**Step 05 — Configured DC-1 private IP addressing.**

![Step 05: Configured DC-1 private IP addressing.](images/Lab-5-Step-05.png)

**Step 06 — Reviewed Windows Firewall settings for lab connectivity testing.**

![Step 06: Reviewed Windows Firewall settings for lab connectivity testing.](images/Lab-5-Step-06.png)

**Step 07 — Configured Client-1 to use DC-1 for DNS.**

![Step 07: Configured Client-1 to use DC-1 for DNS.](images/Lab-5-Step-07.png)

**Step 08 — Tested connectivity between the client and domain controller.**

![Step 08: Tested connectivity between the client and domain controller.](images/Lab-5-Step-08.png)

**Step 09 — Verified Client-1 network and DNS settings.**

![Step 09: Verified Client-1 network and DNS settings.](images/Lab-5-Step-09.png)

### Domain controller and user administration

**Step 10 — Installed Active Directory Domain Services on DC-1.**

![Step 10: Installed Active Directory Domain Services on DC-1.](images/Lab-5-Step-10.png)

**Step 11 — Opened Active Directory Users and Computers after promoting DC-1 to a domain controller.**

![Step 11: Opened Active Directory Users and Computers after promoting DC-1 to a domain controller.](images/Lab-5-Step-11.png)

**Step 12 — Created organizational units to separate administrator and employee accounts.**

![Step 12: Created organizational units to separate administrator and employee accounts.](images/Lab-5-Step-12.png)

**Step 13 — Reviewed the Active Directory organizational structure.**

![Step 13: Reviewed the Active Directory organizational structure.](images/Lab-5-Step-13.png)

**Step 14 — Configured the administrative account and group membership.**

![Step 14: Configured the administrative account and group membership.](images/Lab-5-Step-14.png)

**Step 15 — Reviewed the domain user and organizational unit structure.**

![Step 15: Reviewed the domain user and organizational unit structure.](images/Lab-5-Step-15.png)

**Step 16 — Configured Remote Desktop access for domain users on Client-1.**

![Step 16: Configured Remote Desktop access for domain users on Client-1.](images/Lab-5-Step-16.png)

**Step 17 — Ran a PowerShell script to create multiple test employee accounts.**

![Step 17: Ran a PowerShell script to create multiple test employee accounts.](images/Lab-5-Step-17.png)

**Step 18 — Verified the generated users in the _EMPLOYEES organizational unit.**

![Step 18: Verified the generated users in the _EMPLOYEES organizational unit.](images/Lab-5-Step-18.png)

**Step 19 — Used whoami to confirm a domain user identity.**

![Step 19: Used whoami to confirm a domain user identity.](images/Lab-5-Step-19.png)

**Step 20 — Verified DC-1 and Client-1 were running in Azure.**

![Step 20: Verified DC-1 and Client-1 were running in Azure.](images/Lab-5-Step-20.png)

### Account lockout and recovery

**Step 22 — Configured a domain account lockout threshold of five invalid attempts, with a 10-minute lockout duration and reset interval; verified settings with net accounts /domain.**

![Step 22: Configured a domain account lockout threshold of five invalid attempts, with a 10-minute lockout duration and reset interval; verified settings with net accounts /domain.](images/Lab-5-Step-22.png)

**Step 23 — Confirmed that repeated invalid sign-in attempts caused an RDP account lockout.**

![Step 23: Confirmed that repeated invalid sign-in attempts caused an RDP account lockout.](images/Lab-5-Step-23.png)

**Step 24 — Inspected the locked account in Active Directory Users and Computers and selected the unlock option.**

![Step 24: Inspected the locked account in Active Directory Users and Computers and selected the unlock option.](images/Lab-5-Step-24.png)

**Step 25 — Reset the test user password in Active Directory.**

![Step 25: Reset the test user password in Active Directory.](images/Lab-5-Step-25.png)

**Step 26 — Checked the user session after account recovery.**

![Step 26: Checked the user session after account recovery.](images/Lab-5-Step-26.png)

**Step 27 — Disabled the test user account in Active Directory.**

![Step 27: Disabled the test user account in Active Directory.](images/Lab-5-Step-27.png)

**Step 28 — Confirmed that a disabled user account was blocked from signing in over RDP.**

![Step 28: Confirmed that a disabled user account was blocked from signing in over RDP.](images/Lab-5-Step-28.png)

**Step 29 — Re-enabled the test user account.**

![Step 29: Re-enabled the test user account.](images/Lab-5-Step-29.png)

### Security auditing with Event Viewer

**Step 30 — Reviewed Windows Security Event ID 4740, indicating the test account was locked out.**

![Step 30: Reviewed Windows Security Event ID 4740, indicating the test account was locked out.](images/Lab-5-Step-30.png)

**Step 31 — Reviewed Security Event ID 4624 for a successful logon recorded on DC-1.**

![Step 31: Reviewed Security Event ID 4624 for a successful logon recorded on DC-1.](images/Lab-5-Step-31.png)

**Step 32 — Reviewed Security Event ID 4776 for a failed credential validation attempt.**

![Step 32: Reviewed Security Event ID 4776 for a failed credential validation attempt.](images/Lab-5-Step-32.png)

**Step 33 — Reviewed Security Event ID 4624 on Client-1, showing a successful logon event for the test account.**

![Step 33: Reviewed Security Event ID 4624 on Client-1, showing a successful logon event for the test account.](images/Lab-5-Step-33.png)

## Results
- Created and managed Active Directory users in an Azure-hosted lab.
- Verified domain identity and reviewed network/DNS configuration.
- Demonstrated an account lockout after repeated unsuccessful sign-in attempts.
- Practiced account recovery, password reset, disabling, and re-enabling.
- Located relevant Windows Security events: **4740** (account lockout), **4624** (successful logon), and **4776** (credential validation).

## Skills demonstrated
Active Directory administration · Azure virtual machines · DNS fundamentals · Group Policy · PowerShell · Remote Desktop troubleshooting · Account security · Windows Event Viewer

## Notes
This was a controlled training environment. Firewall changes and simplified account policies shown in the lab are not production recommendations.
