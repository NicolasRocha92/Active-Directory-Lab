# Active Directory: User Management, Group Policy \& Security Auditing

## Project Overview

This lab demonstrates the setup and administration of an Active Directory domain in Microsoft Azure. It covers virtual machine networking, DNS, domain joining, user management, Group Policy account lockout testing, and Windows security logs.

## Environment and Technologies

* Microsoft Azure (virtual machines, virtual networks, and network interfaces)
* Windows Server 2022 (`DC-1`) and Windows 10 Pro or Enterprise (`Client-1`)
* Active Directory Domain Services (AD DS) and DNS
* Active Directory Users and Computers (ADUC)
* Group Policy Management, PowerShell ISE, Remote Desktop, and Event Viewer
* Lab domain: `mydomain.com`

## Lab Objectives

1. Deploy and connect a domain controller and Windows client in Azure.
2. Install AD DS, configure DNS, and join the client to the domain.
3. Create and manage organizational units and domain user accounts.
4. Configure account lockout, restore account access, and test account status changes.
5. Examine authentication events on the domain controller and client.

## Step-by-Step Instructions

### Azure Infrastructure and Networking

#### Step 01 — Create Azure resource groups

**Procedure:** In the Azure portal, open **Resource groups → Create**. Select the subscription and region, enter a name for the lab resource group, and select **Review + create → Create**. Create an additional group only if needed for the lab.

**Expected result:** The resource group or groups appear in the Azure portal.

!\[Step 01](images/Lab-5-Step-01.png)

#### Step 02 — Create a virtual network and subnet

**Procedure:** Open **Virtual networks → Create**. Choose the lab resource group and region, define an address space and subnet, and complete deployment. Both virtual machines will use this network.

**Expected result:** The virtual network and subnet are available.

!\[Step 02](images/Lab-5-Step-02.png)

#### Step 03 — Verify the virtual network configuration

**Procedure:** Open the new virtual network and check **Overview** and **Subnets**. Confirm its address space, subnet, region, and resource group.

**Expected result:** The network settings match the planned lab environment.

!\[Step 03](images/Lab-5-Step-03.png)

#### Step 04 — Deploy the DC-1 virtual machine

**Procedure:** Open **Virtual machines → Create → Azure virtual machine**. Deploy `DC-1` with **Windows Server 2022**, a local administrator account, and the lab virtual network and subnet. Configure Remote Desktop access only for the controlled lab environment.

**Expected result:** Azure reports successful deployment of `DC-1`.

!\[Step 04](images/Lab-5-Step-04.png)

#### Step 05 — Assign DC-1 a static private IP address

**Procedure:** Open the network interface attached to `DC-1` in Azure. Under **IP configurations**, select the primary IP configuration, set **Private IP allocation** to **Static**, retain the assigned address, and save. Record that address for the client DNS configuration.

**Expected result:** The private IP allocation shows **Static** in Azure.

!\[Step 05](images/Lab-5-Step-05.png)

#### Step 06 — Configure Windows Firewall for the connectivity test

**Procedure:** Connect to `DC-1` with Remote Desktop and open **Windows Defender Firewall**. For this isolated exercise, the checklist calls for temporarily disabling the firewall to test connectivity. Re-enable it after testing, ensuring the necessary Active Directory, DNS, and Remote Desktop traffic is permitted by appropriate firewall rules. In production, configure only the required rules rather than disabling the firewall.

**Expected result:** The firewall configuration permits the intended connectivity test.

!\[Step 06](images/Lab-5-Step-06.png)

#### Step 07 — Deploy Client-1 and configure DNS

**Procedure:** Deploy a Windows 10 Pro or Enterprise VM named `Client-1` in the **same region and virtual network** as `DC-1`. In Azure, open the client network interface, set **DNS servers** to **Custom**, enter the static private IP of `DC-1`, and save. Restart `Client-1` from the Azure portal.

**Expected result:** The client uses `DC-1` as its configured DNS server.

!\[Step 07](images/Lab-5-Step-07.png)

#### Step 08 — Test connectivity to DC-1

**Procedure:** Sign in to `Client-1` and run `ping <DC-1-private-IP>` in Command Prompt, replacing the placeholder with the actual private IP. If the ping fails, inspect the VM networking, network security rules, and Windows Firewall settings.

**Expected result:** Replies from `DC-1` confirm basic IP connectivity; a successful ping alone does not verify DNS.

!\[Step 08](images/Lab-5-Step-08.png)

#### Step 09 — Verify Client-1 DNS settings

**Procedure:** On `Client-1`, run `ipconfig /all` and locate the active network adapter. Compare the **DNS Servers** value with the private IP address of `DC-1`.

**Expected result:** The DNS server address matches `DC-1`.

!\[Step 09](images/Lab-5-Step-09.png)

### Active Directory Setup and Domain Administration

#### Step 10 — Install Active Directory Domain Services

**Procedure:** Sign in to `DC-1`. In **Server Manager → Manage → Add Roles and Features**, choose **Role-based or feature-based installation**, select the local server, add **Active Directory Domain Services** and its required features, and complete installation. Select **Promote this server to a domain controller** when the role installation finishes.

**Expected result:** AD DS installation completes and the domain-controller promotion wizard is available.

!\[Step 10](images/Lab-5-Step-10.png)

#### Step 11 — Promote DC-1 and create the domain

**Procedure:** In the promotion wizard, choose **Add a new forest** and enter `mydomain.com`. Set a Directory Services Restore Mode password, review the remaining settings, and complete promotion. After the restart, sign in with the domain administrator account and open **Server Manager → Tools → Active Directory Users and Computers (ADUC)**.

**Expected result:** The `mydomain.com` domain is visible in ADUC.

!\[Step 11](images/Lab-5-Step-11.png)

#### Step 12 — Create the employee and administrator OUs

**Procedure:** In ADUC, expand `mydomain.com`, right-click the domain, and select **New → Organizational Unit**. Create `\_EMPLOYEES` and `\_ADMINS` as separate OUs.

**Expected result:** Both OUs appear under the domain.

!\[Step 12](images/Lab-5-Step-12.png)

#### Step 13 — Verify the organizational unit structure

**Procedure:** In ADUC, review `\_EMPLOYEES` and `\_ADMINS` to confirm that each OU exists under `mydomain.com` before creating accounts.

**Expected result:** Both OUs are visible in the expected location.

!\[Step 13](images/Lab-5-Step-13.png)

#### Step 14 — Create and verify the domain administrator

**Procedure:** In ADUC, create the user **Jane Doe** with username `jane\_admin` in `\_ADMINS`. Assign a lab password without publishing it. Open the user’s **Properties → Member Of**, add **Domain Admins**, and confirm the membership. Sign out of `DC-1` and sign back in as `jane\_admin@mydomain.com` for subsequent administration.

**Expected result:** The user belongs to **Domain Admins**. A successful domain sign-in verifies that the account can authenticate.

!\[Step 14](images/Lab-5-Step-14.png)

#### Step 15 — Join Client-1 to the domain and organize its computer account

**Procedure:** On `Client-1`, sign in using its original local administrator account. Confirm DNS points to `DC-1`. Open **System Properties → Computer Name → Change**, select **Domain**, enter `mydomain.com`, and provide authorized domain credentials. Restart when prompted. On `DC-1`, open ADUC, confirm the `Client-1` computer object exists, create the `\_CLIENTS` OU, and move the computer object into it.

**Expected result:** `Client-1` is joined to `mydomain.com` and its computer object appears in `\_CLIENTS`.

!\[Step 15](images/Lab-5-Step-15.png)

### Remote Desktop and Employee Account Management

#### Step 16 — Allow domain users to connect to Client-1 by Remote Desktop

**Procedure:** Sign in to `Client-1` as `jane\_admin@mydomain.com`. Open **Settings → System → Remote Desktop** or **System Properties → Remote**, enable Remote Desktop, and use **Select users** to allow the **Domain Users** group as required by the lab.

**Expected result:** Domain Users are permitted to use Remote Desktop on the client, subject to applicable policy and firewall settings.

!\[Step 16](images/Lab-5-Step-16.png)

#### Step 17 — Create employee accounts using PowerShell

**Procedure:** Sign in to `DC-1` as `jane\_admin`. Open **Windows PowerShell ISE** as administrator, review the [bulk-user creation script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1), and run it in the lab environment. Review the script output for errors.

**Expected result:** The script creates test user accounts in Active Directory.

!\[Step 17](images/Lab-5-Step-17.png)

#### Step 18 — Verify employee accounts in ADUC

**Procedure:** Open ADUC, expand `mydomain.com → \_EMPLOYEES`, and refresh the view. Inspect the newly created user accounts.

**Expected result:** The generated users appear in `\_EMPLOYEES`.

!\[Step 18](images/Lab-5-Step-18.png)

#### Step 19 — Test sign-in with a standard domain user

**Procedure:** Connect to `Client-1` over Remote Desktop using one of the generated domain accounts and the password configured by the script. Open Command Prompt and run `whoami`.

**Expected result:** The command displays the domain and username of the signed-in account.

!\[Step 19](images/Lab-5-Step-19.png)

### Account Lockout and Recovery

#### Step 20 — Confirm both Azure VMs are running

**Procedure:** Before continuing with the account troubleshooting exercises, open **Virtual machines** in the Azure portal and start `DC-1` and `Client-1` if either VM is stopped.

**Expected result:** Both VMs show **Running**.

!\[Step 20](images/Lab-5-Step-20.png)

#### Step 21 — Test incorrect passwords before changing the lockout policy

**Procedure:** Choose a test domain account and attempt to sign in with an incorrect password **10 times** before modifying the account lockout policy. If an existing policy locks the account earlier, record that behavior and stop the attempts. Unlock the account before the next test if necessary.

**Expected result:** This baseline test shows the account’s behavior under the existing policy. The result depends on the policy already in effect.

#### Step 22 — Configure a five-attempt domain account lockout threshold

**Procedure:** On `DC-1`, open **Group Policy Management**, expand the `mydomain.com` domain, and edit **Default Domain Policy** (or another applicable GPO linked at the domain root that defines the domain account policy). Navigate to **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy**. Set **Account lockout threshold** to **5 invalid logon attempts**. Review the related lockout duration and counter-reset settings, then apply the policy and allow time for replication and policy updates before testing.

**Expected result:** The domain account policy shows a five-attempt threshold. `net accounts /domain` can be used to inspect the effective domain account policy; this command is an optional verification step.

!\[Step 22](images/Lab-5-Step-22.png)

#### Step 23 — Test account lockout after the policy change

**Procedure:** Attempt to sign in to `Client-1` with the test account using an incorrect password **6 times**, as directed by the checklist. With the five-attempt threshold applied, the account should lock by the fifth counted failed attempt, so later attempts may be rejected as locked out.

**Expected result:** Windows reports that the account is locked out, and ADUC can be used to inspect its status.

!\[Step 23](images/Lab-5-Step-23.png)

#### Step 24 — Unlock the test account

**Procedure:** On `DC-1`, open **ADUC → \_EMPLOYEES** and locate the test user. Open **Properties → Account**, select the option to unlock the account when available, and apply the change.

**Expected result:** The account is no longer locked out.

!\[Step 24](images/Lab-5-Step-24.png)

#### Step 25 — Reset the test account password

**Procedure:** In ADUC, right-click the test account, select **Reset Password**, and assign a new lab password. Choose the appropriate next-sign-in option for the exercise. Do not publish passwords in the README.

**Expected result:** ADUC confirms the password reset.

!\[Step 25](images/Lab-5-Step-25.png)

#### Step 26 — Verify account recovery with a sign-in

**Procedure:** Attempt to sign in to `Client-1` using the recovered account and its new password. Open Command Prompt and run `whoami` to confirm the authenticated identity.

**Expected result:** A successful sign-in and matching `whoami` output verify recovery.

!\[Step 26](images/Lab-5-Step-26.png)

#### Step 27 — Disable the test user account

**Procedure:** In **ADUC → \_EMPLOYEES**, right-click the test account and select **Disable Account**.

**Expected result:** ADUC shows that the account is disabled.

!\[Step 27](images/Lab-5-Step-27.png)

#### Step 28 — Confirm the disabled account cannot sign in

**Procedure:** Sign out of the test account on `Client-1` and attempt a new sign-in using its correct password.

**Expected result:** Windows rejects authentication because the account is disabled.

!\[Step 28](images/Lab-5-Step-28.png)

#### Step 29 — Re-enable the test account and test sign-in

**Procedure:** In ADUC, right-click the disabled account and select **Enable Account**. Then attempt to sign in to `Client-1` using that account and its current password.

**Expected result:** ADUC shows the account as enabled. A successful sign-in confirms that access is restored; if sign-in fails, investigate other restrictions before claiming success.

!\[Step 29](images/Lab-5-Step-29.png)

### Security Logs and Event Viewer

#### Step 30 — Inspect account lockout events on DC-1

**Procedure:** On `DC-1`, open **Event Viewer → Windows Logs → Security** and filter for Event ID `4740`. Review the account name, timestamp, and source information for a relevant event.

**Expected result:** Event ID `4740` identifies an account lockout when the event is recorded.

!\[Step 30](images/Lab-5-Step-30.png)

#### Step 31 — Inspect successful logon events on DC-1

**Procedure:** In the Security log on `DC-1`, filter for Event ID `4624`. Review the account, timestamp, and **Logon Type**.

**Expected result:** Event ID `4624` indicates a successful logon; its logon type identifies the session category.

!\[Step 31](images/Lab-5-Step-31.png)

#### Step 32 — Inspect NTLM credential validation on DC-1

**Procedure:** In the Security log on `DC-1`, inspect Event ID `4776` and its status code. A status of `0xC0000072` means the account is disabled, not that the password is incorrect.

**Expected result:** Event ID `4776` records NTLM credential validation activity when applicable.

!\[Step 32](images/Lab-5-Step-32.png)

#### Step 33 — Inspect logon events on Client-1

**Procedure:** On `Client-1`, open **Event Viewer → Windows Logs → Security** and inspect Event ID `4624`. Check the account, timestamp, and logon type. **Logon Type 3** indicates a network logon; an interactive Remote Desktop session normally uses **Logon Type 10**.

**Expected result:** The event details distinguish network authentication from an interactive RDP sign-in.

!\[Step 33](images/Lab-5-Step-33.png)

## Skills Practiced

Azure networking · Active Directory administration · DNS · Group Policy · PowerShell · Remote Desktop · Account troubleshooting · Windows Event Viewer

## Lab Resource Management

Keep the virtual machines available for subsequent exercises. When pausing the lab, use **Stop (deallocate)** in the Azure portal to reduce VM compute charges. Other Azure resources may still incur charges.

## Security Considerations

These exercises use an isolated lab environment. Do not disable Windows Firewall, grant broad Remote Desktop access, or use simplified test passwords in production. Apply least-privilege access and protect account credentials.

