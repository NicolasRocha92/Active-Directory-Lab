# Active Directory: User Management, Group Policy \& Security Auditing

## Project overview

I built an Active Directory test environment in Microsoft Azure to practice Windows domain administration, DNS configuration, account management, Group Policy, account recovery, and Windows security auditing. I documented the setup, the problems I tested, and how I verified the results along the way.

**Environment:** Microsoft Azure; Windows Server 2022 (`DC-1`); Windows client (`Client-1`); Active Directory Domain Services (AD DS); DNS; Group Policy Management; PowerShell; Remote Desktop; Windows Event Viewer.

**Lab domain:** `mydomain.com`

## Objectives

* Deploy and network a Windows domain controller and client in Azure.
* Configure DNS, verify connectivity, and join the client to the domain.
* Create organizational units, administrators, and employee test accounts.
* Configure and test account lockout, password reset, and account enable/disable operations.
* Inspect authentication-related Windows Security events.

## Lab walkthrough



### Azure infrastructure and domain connectivity

#### Step 01 — Create the Azure resource groups

**What I did:** I started in the Azure portal by searching for **Resource groups** and selected **Create**. I chose the subscription, entered the lab resource group name, selected the lab region, and selected **Review + create → Create**. I repeated this for the additional resource group used in the environment.

**What I checked:** The resource groups appear in the Resource groups list.

!\[Step 1](images/Lab-5-Step-01.png)

#### Step 02 — Create the virtual network and subnet

**What I did:** I searched for **Virtual networks** in Azure and selected **Create**. I chose the subscription and Active Directory resource group. I entered the virtual network name and selected the same region planned for the VMs. I opened **IP addresses**, defined the address space and subnet, then selected **Review + create → Create**.

**What I checked:** The new virtual network and subnet are available for both VMs.

!\[Step 2](images/Lab-5-Step-02.png)

#### Step 03 — Review the virtual network configuration

**What I did:** I opened **Virtual networks**, selected the network just created, and inspected **Overview** and **Subnets**. I verified the address space, subnet prefix, resource group, and region before deploying the VMs.

**What I checked:** The VNet and subnet show the intended configuration.

!\[Step 3](images/Lab-5-Step-03.png)

#### Step 04 — Deploy the DC-1 Windows Server VM

**What I did:** I went to **Virtual machines → Create → Azure virtual machine**. I selected the lab resource group, entered `DC-1` as the name, chose **Windows Server 2022**, selected a suitable VM size, and configured a local administrator username and password. Under **Networking**, I chose the previously created virtual network and subnet. I reviewed the inbound RDP setting for the controlled lab, then selected **Review + create → Create**.

**What I checked:** Azure reports a successful deployment and DC-1 appears in Virtual machines.

!\[Step 4](images/Lab-5-Step-04.png)

#### Step 05 — Assign DC-1 a static private IP

**What I did:** In Azure, I opened **Virtual machines → DC-1 → Networking** and selected its network interface. I opened **IP configurations**, selected the primary configuration, changed **Private IP allocation** from Dynamic to **Static**, retained the assigned private IP, and selected **Save**. I recorded the private IP for Client-1 DNS configuration.

**What I checked:** Azure shows DC-1’s private IP allocation set to **Static**. This configuration is intended to retain the address across restarts.

!\[Step 5](images/Lab-5-Step-05.png)

#### Step 06 — Review Windows Firewall for connectivity testing

**What I did:** I connected to DC-1 using **Remote Desktop**. I opened **Windows Defender Firewall** (for example, from Control Panel) and inspected the domain/private/public profiles. For this isolated lab, I temporarily adjusted the firewall setting for connectivity testing. In a production environment, I would use narrowly scoped firewall rules instead.

**What I checked:** The firewall configuration used for the lab is visible; the ping results are documented separately in Step 08.

!\[Step 6](images/Lab-5-Step-06.png)

#### Step 07 — Configure Client-1 to use DC-1 for DNS

**What I did:** I created a Windows client VM named `Client-1` in the **same virtual network and subnet** as DC-1 using **Virtual machines → Create**. I opened Client-1’s network interface in Azure, chose **DNS servers**, selected **Custom**, entered DC-1’s static private IP, and **Save**. I restarted Client-1 so the DNS change takes effect.

**What I checked:** Client-1 uses the domain controller as its DNS server.

!\[Step 7](images/Lab-5-Step-07.png)

#### Step 08 — Test connectivity between Client-1 and DC-1

**What I did:** I connected to Client-1 over RDP, opened **Command Prompt**, and ran `ping <DC-1-private-IP>` using the actual DC-1 private address. I used the ping results to verify connectivity between the two machines.

**What I checked:** The ping returns replies from DC-1, confirming basic IP connectivity.

!\[Step 8](images/Lab-5-Step-08.png)

#### Step 09 — Verify IP and DNS settings on Client-1

**What I did:** On Client-1, I opened **Command Prompt** and ran `ipconfig /all`. I found the active Ethernet adapter and inspected **IPv4 Address**, **Subnet Mask**, **Default Gateway**, and **DNS Servers**. I compared the DNS server value to DC-1’s static private IP.

**What I checked:** The DNS Servers entry matches DC-1’s private IP.

!\[Step 9](images/Lab-5-Step-09.png)



### Domain controller and user administration

#### Step 10 — Install Active Directory Domain Services

**What I did:** On DC-1, I opened **Server Manager → Manage → Add Roles and Features**. I chose **Role-based or feature-based installation**, selected the local server, checked **Active Directory Domain Services**, accepted the required features, and selected **Install**. After installation, I selected the notification flag and **Promote this server to a domain controller**.

**What I checked:** The Server Manager role-selection screen shows Active Directory Domain Services selected for installation.

!\[Step 10](images/Lab-5-Step-10.png)

#### Step 11 — Verify the domain in Active Directory Users and Computers

**What I did:** In the promotion wizard, I chose **Add a new forest**, entered `mydomain.com`, set a Directory Services Restore Mode password, proceeded through the DNS/NetBIOS/path checks, and selected **Install**. I allowed DC-1 to restart. I signed in with a domain administrator account, then opened **Server Manager → Tools → Active Directory Users and Computers**.

**What I checked:** Active Directory Users and Computers displays the `mydomain.com` domain after domain-controller promotion. The screenshot documents the resulting domain, not the promotion wizard.

!\[Step 11](images/Lab-5-Step-11.png)

#### Step 12 — Create administrator and employee organizational units

**What I did:** In **Active Directory Users and Computers**, I expanded `mydomain.com`, right-clicked the domain, and selected **New → Organizational Unit**. I created `\_ADMINS` and `\_EMPLOYEES` separately.

**What I checked:** Both OUs appear under the domain and can hold their respective accounts.

!\[Step 12](images/Lab-5-Step-12.png)

#### Step 13 — Verify the administrative account in \_ADMINS

**What I did:** In Active Directory Users and Computers, I selected the `\_ADMINS` organizational unit under `mydomain.com` and inspected its contents.

**What I checked:** The `Jane Doe` user account is visible inside `\_ADMINS`. This screenshot shows the account's location, not its Domain Admins group membership.

!\[Step 13](images/Lab-5-Step-13.png)

#### Step 14 — Create and grant membership to the admin account

**What I did:** In ADUC, I right-clicked `\_ADMINS` and selected **New → User**. I created the administrative test account `jane\_admin`, set its password according to the lab settings, and finished the wizard. I opened its **Properties → Member Of → Add**, entered `Domain Admins`, selected **Check Names**, and applied the change. Use elevated rights only for the lab administration tasks.

**What I checked:** The account properties show membership in the Domain Admins group.

!\[Step 14](images/Lab-5-Step-14.png)

#### Step 15 — Verify Client-1 in the \_CLIENTS organizational unit

**What I did:** In Active Directory Users and Computers, I expanded `mydomain.com` and selected the `\_CLIENTS` organizational unit.

**What I checked:** The `Client-1` computer object appears inside `\_CLIENTS`, confirming its location in the domain's organizational unit structure.

!\[Step 15](images/Lab-5-Step-15.png)

#### Step 16 — Configure Remote Desktop access for domain users on Client-1

**What I did:** On Client-1, I opened **System Properties → Computer Name → Change**, selected **Domain**, entered `mydomain.com`, and supplied domain administrator credentials. I restarted when prompted. I signed in with the domain administrator, opened **Settings → System → Remote Desktop** (or **System Properties → Remote**), enabled Remote Desktop, and used **Select users** to add **Domain Users** if required by the lab.

**What I checked:** Remote Desktop is enabled and the user-selection dialog documents the domain-user access configuration. This screenshot does not document the domain-join dialog.

!\[Step 16](images/Lab-5-Step-16.png)

#### Step 17 — Create employee test accounts with PowerShell

**What I did:** On DC-1, I opened **Windows PowerShell ISE** as administrator. I opened or pasted the lab’s bulk-user creation script (`Generate-Names-Create-Users.ps1`), inspected it to confirm the target domain/OU and user creation behavior, then selected **Run Script**. I reviewed the console output for completion messages and errors.

**What I checked:** PowerShell ISE displays the bulk-user script and its execution output. The resulting accounts are shown in Step 18.

!\[Step 17](images/Lab-5-Step-17.png)

#### Step 18 — Verify the generated accounts in \_EMPLOYEES

**What I did:** I opened **Active Directory Users and Computers**, expanded `mydomain.com → \_EMPLOYEES`, and refreshed the view. I reviewed the newly created accounts and confirm they are user objects in the intended OU.

**What I checked:** Multiple test employee accounts appear in `\_EMPLOYEES`.

!\[Step 18](images/Lab-5-Step-18.png)

#### Step 19 — Verify a standard domain user with whoami

**What I did:** I connected to Client-1 over RDP using one of the test domain accounts. I opened **Command Prompt**, entered `whoami` to check which domain account was signed in.

**What I checked:** The `whoami` output identifies the signed-in domain account as `mydomain\\sok.dem`.

!\[Step 19](images/Lab-5-Step-19.png)

#### Step 20 — Check that both Azure VMs are running

**What I did:** In Azure, I opened **Virtual machines** and found `DC-1` and `Client-1`. I inspected the **Status**, network placement, and resource group before continuing with the account policy tests. Both VMs were running when I continued.

**What I checked:** Both VMs show Running and are available for the next tests.

!\[Step 20](images/Lab-5-Step-20.png)



### Account lockout and recovery

#### Step 22 — Configure and verify the domain account lockout policy

**What I did:** On DC-1, I opened **Group Policy Management** from **Server Manager → Tools**. I expanded **Forest → Domains → mydomain.com**, edited the applicable domain password/account policy GPO, and navigated to **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy**. I set **Account lockout threshold** to `5` invalid attempts, **Account lockout duration** to `10` minutes, and **Reset account lockout counter after** to `10` minutes. I applied the policy and ran `gpupdate /force` where appropriate. In an elevated Command Prompt, ran `net accounts /domain` to check the effective values.

**What I checked:** The effective domain policy reports a threshold of five attempts and 10-minute lockout/reset settings.

!\[Step 22](images/Lab-5-Step-22.png)

#### Step 23 — Reproduce an account lockout over Remote Desktop

**What I did:** At the RDP sign-in screen for Client-1, I selected a test domain account and intentionally entered an incorrect password repeatedly until the configured threshold is reached. I performed this test only on the lab account.

**What I checked:** Windows denies sign-in and reports that the account is locked out.

!\[Step 23](images/Lab-5-Step-23.png)

#### Step 24 — Locate and unlock the affected account

**What I did:** On DC-1, I opened **Active Directory Users and Computers → \_EMPLOYEES** and opened the affected user’s **Properties → Account**. I inspected the lockout status and selected **Unlock account** when the option was available, then applied the change.

**What I checked:** The Account tab shows the option to unlock the affected account selected; this screenshot documents the recovery action.

!\[Step 24](images/Lab-5-Step-24.png)

#### Step 25 — Reset the test account password

**What I did:** In ADUC, I right-clicked the affected user and chose **Reset Password**. I entered and confirmed a new lab password, selected the appropriate next-logon option, and confirmed the reset. I did not include the password in the README.

**What I checked:** ADUC displays a password-reset confirmation. Step 26 separately documents access to the user profile after recovery.

!\[Step 25](images/Lab-5-Step-25.png)

#### Step 26 — Open Command Prompt under the recovered user profile

**What I did:** After recovering the test account, I signed in to the Windows client as `nado.cif` and opened Command Prompt.

**What I checked:** The prompt displays `C:\\Users\\nado.cif>`, indicating the test user's profile. The screenshot does not show a `whoami` command.

!\[Step 26](images/Lab-5-Step-26.png)

#### Step 27 — Disable the test user account

**What I did:** On DC-1, I opened **ADUC → \_EMPLOYEES**, right-clicked the test account, and selected **Disable Account**. Confirm the action and inspected the account status.

**What I checked:** The user account is disabled in Active Directory.

!\[Step 27](images/Lab-5-Step-27.png)

#### Step 28 — Confirm the disabled account cannot sign in

**What I did:** I signed out of the test account on Client-1. I attempted a new RDP sign-in using that disabled domain account and its correct password.

**What I checked:** Windows blocks authentication and displays a disabled-account message.

!\[Step 28](images/Lab-5-Step-28.png)

#### Step 29 — Re-enable the test user account

**What I did:** I returned to **ADUC → \_EMPLOYEES**, right-clicked the disabled user, and selected **Enable Account**. I confirmed the account was enabled before performing any further sign-in tests.

**What I checked:** ADUC displays the confirmation that the test account was enabled again.

!\[Step 29](images/Lab-5-Step-29.png)



### Security auditing with Event Viewer

#### Step 30 — Investigate Event ID 4740 for the lockout

**What I did:** On DC-1, I opened **Event Viewer → Windows Logs → Security**. I selected **Filter Current Log**, entered `4740` in **Event IDs**, and inspected the matching event details for the test account and lockout time.

**What I checked:** A 4740 event records the account lockout, where auditing captured the event.

!\[Step 30](images/Lab-5-Step-30.png)

#### Step 31 — Inspect Event ID 4624 on DC-1

**What I did:** In **Event Viewer → Windows Logs → Security** on DC-1, filtered for `4624`. I opened a matching event and reviewed its **Account Name**, **Logon Type**, and timestamp to understand the successful logon recorded on the domain controller.

**What I checked:** Event ID 4624 documents a successful logon on DC-1; it does not necessarily represent an interactive sign-in.

!\[Step 31](images/Lab-5-Step-31.png)

#### Step 32 — Inspect Event ID 4776 for failed credential validation

**What I did:** On DC-1, I opened **Event Viewer → Windows Logs → Security** and inspected Event ID `4776`, which records NTLM credential validation attempts.

**What I checked:** The event shows a failed credential validation for the test account with status `0xC0000072`, indicating that the account was disabled. This is distinct from an incorrect-password error.

!\[Step 32](images/Lab-5-Step-32.png)

#### Step 33 — Inspect a successful network logon on Client-1

**What I did:** On Client-1, I opened **Event Viewer → Windows Logs → Security** and inspected a successful logon event, Event ID `4624`, associated with the test account.

**What I checked:** The event reports **Logon Type 3**, which indicates a successful **network logon**. It does not, by itself, prove an interactive Remote Desktop sign-in.

!\[Step 33](images/Lab-5-Step-33.png)

## Results

* Deployed and connected a Windows domain controller and client in Azure.
* Created and managed Active Directory users and organizational units.
* Applied and tested domain account lockout and account recovery procedures.
* Practiced password reset and account disable/enable workflows.
* Examined Windows Security Event IDs `4740`, `4624`, and `4776`.

## Skills demonstrated

Active Directory administration · Azure VMs · DNS · Group Policy · PowerShell · RDP · User account troubleshooting · Windows Event Viewer

## Lab safety note

This was an isolated learning environment. Temporary firewall changes, permissive RDP access, and simplified lockout settings should not be copied directly into a production environment. Credentials and private configuration details should be handled securely.

