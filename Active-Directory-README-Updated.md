# Active Directory: User Management, Group Policy \& Security Auditing

## Project overview

I built an Active Directory test environment in Microsoft Azure to practice Windows domain administration, DNS configuration, account management, Group Policy, account recovery, and Windows security auditing. This walkthrough documents the configuration process and the expected result of each stage.

**Environment:** Microsoft Azure; Windows Server 2022 (`DC-1`); Windows client (`Client-1`); Active Directory Domain Services (AD DS); DNS; Group Policy Management; PowerShell; Remote Desktop; Windows Event Viewer.

**Lab domain:** `mydomain.com`

## Objectives

* Deploy and network a Windows domain controller and client in Azure.
* Configure DNS, verify connectivity, and join the client to the domain.
* Create organizational units, administrators, and employee test accounts.
* Configure and test account lockout, password reset, and account enable/disable operations.
* Inspect authentication-related Windows Security events.

## Step-by-step implementation



### Azure infrastructure and domain connectivity

#### Step 01 — Create the Azure resource groups

**Procedure:** In the Azure portal, search for **Resource groups** and select **Create**. Choose the subscription, enter the resource group name (use the resource group name shown in your Azure configuration (for example, `Active-Directory-Lab`)), select the lab region, and select **Review + create → Create**. Repeat for any additional resource group used in the environment.

**Expected result:** The resource groups appear in the Resource groups list.

!\[Step 01 — Create the Azure resource groups](images/Lab-5-Step-01.png)

#### Step 02 — Create the virtual network and subnet

**Procedure:** Search for **Virtual networks** in Azure and select **Create**. Choose the subscription and Active Directory resource group. Enter the virtual network name and select the same region planned for the VMs. Open **IP addresses**, define an address space and subnet that do not overlap other networks, then select **Review + create → Create**.

**Expected result:** The new virtual network and subnet are available for both VMs.

!\[Step 02 — Create the virtual network and subnet](images/Lab-5-Step-02.png)

#### Step 03 — Review the virtual network configuration

**Procedure:** Open **Virtual networks**, select the network just created, and inspect **Overview** and **Subnets**. Verify the address space, subnet prefix, resource group, and region before deploying the VMs.

**Expected result:** The VNet and subnet show the intended configuration.

!\[Step 03 — Review the virtual network configuration](images/Lab-5-Step-03.png)

#### Step 04 — Deploy the DC-1 Windows Server VM

**Procedure:** Go to **Virtual machines → Create → Azure virtual machine**. Select the lab resource group, enter `DC-1` as the name, choose **Windows Server 2022**, select a suitable VM size, and configure a local administrator username and password. Under **Networking**, choose the previously created virtual network and subnet. Review the inbound RDP setting for your controlled lab, then select **Review + create → Create**.

**Expected result:** Azure reports a successful deployment and DC-1 appears in Virtual machines.

!\[Step 04 — Deploy the DC-1 Windows Server VM](images/Lab-5-Step-04.png)

#### Step 05 — Assign DC-1 a static private IP

**Procedure:** In Azure, open **Virtual machines → DC-1 → Networking** and select its network interface. Open **IP configurations**, select the primary configuration, change **Private IP allocation** from Dynamic to **Static**, retain the assigned private IP, and **Save**. Record the private IP for Client-1 DNS configuration.

**Expected result:** DC-1 keeps the same private IP address across restarts.

!\[Step 05 — Assign DC-1 a static private IP](images/Lab-5-Step-05.png)

#### Step 06 — Review Windows Firewall for connectivity testing

**Procedure:** Connect to DC-1 using **Remote Desktop**. Open **Windows Defender Firewall** (for example, from Control Panel) and inspect the domain/private/public profiles. For this isolated exercise, temporarily disable the relevant firewall profile if needed to reproduce the ICMP connectivity test; restore normal protection after testing.

**Expected result:** The intended connectivity test is no longer blocked by the temporary firewall setting.

!\[Step 06 — Review Windows Firewall for connectivity testing](images/Lab-5-Step-06.png)

#### Step 07 — Configure Client-1 to use DC-1 for DNS

**Procedure:** Create a Windows client VM named `Client-1` in the **same virtual network and subnet** as DC-1, using **Virtual machines → Create**. Open Client-1’s network interface in Azure, choose **DNS servers**, select **Custom**, enter DC-1’s static private IP, and **Save**. Restart Client-1 so the DNS change takes effect.

**Expected result:** Client-1 uses the domain controller as its DNS server.

!\[Step 07 — Configure Client-1 to use DC-1 for DNS](images/Lab-5-Step-07.png)

#### Step 08 — Test connectivity between Client-1 and DC-1

**Procedure:** Connect to Client-1 over RDP, open **Command Prompt**, and run `ping <DC-1-private-IP>` using the actual DC-1 private address. If replies fail, verify the VNet/subnet, NIC configuration, NSG rules, and Windows Firewall settings.

**Expected result:** The ping returns replies from DC-1, confirming basic IP connectivity.

!\[Step 08 — Test connectivity between Client-1 and DC-1](images/Lab-5-Step-08.png)

#### Step 09 — Verify IP and DNS settings on Client-1

**Procedure:** On Client-1, open **Command Prompt** and run `ipconfig /all`. Find the active Ethernet adapter and inspect **IPv4 Address**, **Subnet Mask**, **Default Gateway**, and **DNS Servers**. Compare the DNS server value to DC-1’s static private IP.

**Expected result:** The DNS Servers entry matches DC-1’s private IP.

!\[Step 09 — Verify IP and DNS settings on Client-1](images/Lab-5-Step-09.png)



### Domain controller and user administration

#### Step 10 — Install Active Directory Domain Services

**Procedure:** On DC-1, open **Server Manager → Manage → Add Roles and Features**. Choose **Role-based or feature-based installation**, select the local server, check **Active Directory Domain Services**, accept the required features, and select **Install**. After installation, select the notification flag and **Promote this server to a domain controller**.

**Expected result:** The AD DS role is installed and the domain-controller promotion wizard is available.

!\[Step 10 — Install Active Directory Domain Services](images/Lab-5-Step-10.png)

#### Step 11 — Promote DC-1 and open Active Directory Users and Computers

**Procedure:** In the promotion wizard, choose **Add a new forest**, enter `mydomain.com`, set a Directory Services Restore Mode password, proceed through the DNS/NetBIOS/path checks, and select **Install**. Allow DC-1 to restart. Sign in with a domain administrator account, then open **Server Manager → Tools → Active Directory Users and Computers**.

**Expected result:** The `mydomain.com` domain is visible in ADUC.

!\[Step 11 — Promote DC-1 and open Active Directory Users and Computers](images/Lab-5-Step-11.png)

#### Step 12 — Create administrator and employee organizational units

**Procedure:** In **Active Directory Users and Computers**, expand `mydomain.com`, right-click the domain, and select **New → Organizational Unit**. Create `\_ADMINS` and `\_EMPLOYEES` separately.

**Expected result:** Both OUs appear under the domain and can hold their respective accounts.

!\[Step 12 — Create administrator and employee organizational units](images/Lab-5-Step-12.png)

#### Step 13 — Review the organizational unit structure

**Procedure:** In ADUC, expand the domain tree and select `\_ADMINS` and `\_EMPLOYEES`. Confirm the names and placement of the organizational units before adding users.

**Expected result:** The organizational units are visible in the domain tree.

!\[Step 13 — Review the organizational unit structure](images/Lab-5-Step-13.png)

#### Step 14 — Create and grant membership to the admin account

**Procedure:** In ADUC, right-click `\_ADMINS` and select **New → User**. Create the administrative test account `jane\_admin`, set its password according to the lab settings, and finish. Open its **Properties → Member Of → Add**, enter `Domain Admins`, select **Check Names**, and apply the change. Use elevated rights only for the lab administration tasks.

**Expected result:** The account exists in `\_ADMINS` and its membership includes Domain Admins.

!\[Step 14 — Create and grant membership to the admin account](images/Lab-5-Step-14.png)

#### Step 15 — Verify the domain user and OU layout

**Procedure:** Return to ADUC and expand the domain and OUs. Select the administrative account and inspect its **Properties**, including **Member Of**, to verify the account is in the intended location with the intended group membership.

**Expected result:** The domain structure and administrative account configuration are visible.

!\[Step 15 — Verify the domain user and OU layout](images/Lab-5-Step-15.png)

#### Step 16 — Join Client-1 to the domain and allow Remote Desktop users

**Procedure:** On Client-1, open **System Properties → Computer Name → Change**, select **Domain**, enter `mydomain.com`, and supply domain administrator credentials. Restart when prompted. Sign in with the domain administrator, open **Settings → System → Remote Desktop** (or **System Properties → Remote**), enable Remote Desktop, and use **Select users** to add **Domain Users** if required by the lab.

**Expected result:** Client-1 is domain joined and permitted domain users can connect through RDP.

!\[Step 16 — Join Client-1 to the domain and allow Remote Desktop users](images/Lab-5-Step-16.png)

#### Step 17 — Create employee test accounts with PowerShell

**Procedure:** On DC-1, open **Windows PowerShell ISE** as administrator. Open or paste the lab’s bulk-user creation script (`Generate-Names-Create-Users.ps1`), inspect it to confirm the target domain/OU and user creation behavior, then select **Run Script**. Watch the console for errors and completion messages.

**Expected result:** The script creates test domain user accounts for the employee OU.

!\[Step 17 — Create employee test accounts with PowerShell](images/Lab-5-Step-17.png)

#### Step 18 — Verify the generated accounts in \_EMPLOYEES

**Procedure:** Open **Active Directory Users and Computers**, expand `mydomain.com → \_EMPLOYEES`, and refresh the view if necessary. Review the newly created accounts and confirm they are user objects in the intended OU.

**Expected result:** Multiple test employee accounts appear in `\_EMPLOYEES`.

!\[Step 18 — Verify the generated accounts in \_EMPLOYEES](images/Lab-5-Step-18.png)

#### Step 19 — Verify the signed-in domain identity

**Procedure:** Connect to Client-1 over RDP using one of the test domain accounts. Open **Command Prompt**, enter `whoami`, and press **Enter** if that was the command used in your lab.

**Expected result:** The output identifies the signed-in domain account rather than only a local computer account.

!\[Step 19 — Verify the signed-in domain identity](images/Lab-5-Step-19.png)

#### Step 20 — Check that both Azure VMs are running

**Procedure:** In Azure, open **Virtual machines** and find `DC-1` and `Client-1`. Inspect the **Status**, network placement, and resource group before continuing with the account policy tests. Start either VM if it is stopped.

**Expected result:** Both VMs show Running and are available for the next tests.

!\[Step 20 — Check that both Azure VMs are running](images/Lab-5-Step-20.png)



### Account lockout and recovery

#### Step 21 — Configure and verify the domain account lockout policy

**Procedure:** On DC-1, open **Group Policy Management** from **Server Manager → Tools**. Expand **Forest → Domains → mydomain.com**, edit the applicable domain password/account policy GPO, and go to **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy**. Set **Account lockout threshold** to `5` invalid attempts, **Account lockout duration** to `10` minutes, and **Reset account lockout counter after** to `10` minutes. Apply the policy and run `gpupdate /force` where appropriate. In an elevated Command Prompt, run `net accounts /domain` to check the effective values.

**Expected result:** The effective domain policy reports a threshold of five attempts and 10-minute lockout/reset settings.

!\[Step 21 — Configure and verify the domain account lockout policy](images/Lab-5-Step-22.png)

#### Step 22 — Reproduce an account lockout over Remote Desktop

**Procedure:** From the RDP sign-in screen for Client-1, select a test domain account and intentionally enter an incorrect password repeatedly until the configured threshold is reached. Keep this test limited to the lab account.

**Expected result:** Windows denies sign-in and reports that the account is locked out.

!\[Step 22 — Reproduce an account lockout over Remote Desktop](images/Lab-5-Step-23.png)

#### Step 23 — Locate and unlock the affected account

**Procedure:** On DC-1, open **Active Directory Users and Computers → \_EMPLOYEES** and open the affected user’s **Properties → Account**. Inspect the lockout status and select **Unlock account** when the option is available, then apply the change.

**Expected result:** The account is no longer marked as locked out.

!\[Step 23 — Locate and unlock the affected account](images/Lab-5-Step-24.png)

#### Step 24 — Reset the test account password

**Procedure:** In ADUC, right-click the affected user and choose **Reset Password**. Enter and confirm a new lab password, set the appropriate next-logon option for the exercise, and confirm the reset. Do not publish the password in screenshots or the README.

**Expected result:** ADUC confirms the reset and the user can authenticate with the new password.

!\[Step 24 — Reset the test account password](images/Lab-5-Step-25.png)

#### Step 25 — Check the recovered user session

**Procedure:** Reconnect to Client-1 using the recovered domain account. Open **Command Prompt** and inspect the active user session; use `whoami` if verifying identity.

**Expected result:** The account can sign in after recovery; the screenshot documents the resulting user session.

!\[Step 25 — Check the recovered user session](images/Lab-5-Step-26.png)

#### Step 26 — Disable the test user account

**Procedure:** On DC-1, open **ADUC → \_EMPLOYEES**, right-click the test account, and select **Disable Account**. Confirm the action and inspect the account icon/status.

**Expected result:** The user account is disabled in Active Directory.

!\[Step 26 — Disable the test user account](images/Lab-5-Step-27.png)

#### Step 27 — Confirm the disabled account cannot sign in

**Procedure:** Sign out of the test account on Client-1. Attempt a new RDP sign-in using that disabled domain account and its correct password.

**Expected result:** Windows blocks authentication and displays a disabled-account message.

!\[Step 27 — Confirm the disabled account cannot sign in](images/Lab-5-Step-28.png)

#### Step 28 — Re-enable the test user account

**Procedure:** Return to **ADUC → \_EMPLOYEES**, right-click the disabled user, and select **Enable Account**. Confirm the account is enabled before performing any further sign-in tests.

**Expected result:** The account is active again and eligible for authentication.

!\[Step 28 — Re-enable the test user account](images/Lab-5-Step-29.png)



### Security auditing with Event Viewer

#### Step 29 — Investigate Event ID 4740 for the lockout

**Procedure:** On DC-1, open **Event Viewer → Windows Logs → Security**. Select **Filter Current Log**, enter `4740` in **Event IDs**, and inspect the matching event details for the test account and lockout time.

**Expected result:** A 4740 event records the account lockout, where auditing captured the event.

!\[Step 29 — Investigate Event ID 4740 for the lockout](images/Lab-5-Step-30.png)

#### Step 30 — Inspect Event ID 4624 on DC-1

**Procedure:** In **Event Viewer → Windows Logs → Security** on DC-1, filter for `4624`. Open a matching event and review its **Account Name**, **Logon Type**, and timestamp to understand the successful logon recorded on the domain controller.

**Expected result:** A 4624 record shows a successful authentication event on DC-1.

!\[Step 30 — Inspect Event ID 4624 on DC-1](images/Lab-5-Step-31.png)

#### Step 31 — Inspect Event ID 4776 for credential validation

**Procedure:** Still on DC-1, filter the **Security** log for `4776`. Open a relevant event and inspect the account, source workstation, and error/status fields; note that this event records NTLM credential validation and its result must be read from the status code.

**Expected result:** A matching 4776 record documents credential validation activity associated with the test.

!\[Step 31 — Inspect Event ID 4776 for credential validation](images/Lab-5-Step-32.png)

#### Step 32 — Inspect Event ID 4624 on Client-1

**Procedure:** On Client-1, open **Event Viewer → Windows Logs → Security**, filter for `4624`, and select the event associated with the test account. Check the account name, logon type, and time against the RDP session.

**Expected result:** Client-1 contains a successful logon event for the test user.

!\[Step 32 — Inspect Event ID 4624 on Client-1](images/Lab-5-Step-33.png)

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

