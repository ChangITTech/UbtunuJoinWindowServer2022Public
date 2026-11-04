### IMPORTANT: THIS IS JUST A ROUGH DRAFT, I AM STILL MISSING IMGS
# Ubuntu Linux Integration with Windows Server 2022 Active Directory

## Project Overview

This homelab demonstrates how to integrate an Ubuntu Linux virtual machine with a Windows Server 2022 Active Directory domain using Oracle VirtualBox.

The objective is to configure network connectivity, DNS resolution, Kerberos authentication, and centralized user management between Windows Server and Linux.

By completing this project, I successfully joined Ubuntu to an Active Directory domain and authenticated using an Active Directory user account created on Windows Server 2022.

## Technologies Used

- Oracle VirtualBox
- Windows Server 2022
- Ubuntu Linux
- Active Directory Domain Services (AD DS)
- DNS
- Kerberos
- SSSD (System Security Services Daemon)
- realmd
- PAM (Pluggable Authentication Modules)
- Linux CLI

## Lab Environment

| Component | Configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain Controller | Windows Server 2022 |
| Linux Client | Ubuntu Linux |
| Active Directory Domain | changserver.com |
| Domain Controller IP | 10.0.2.5 |
| Ubuntu Hostname | changlinux |
| Network | VirtualBox NAT Network |
| Test Account | testguy@changserver.com |

## Prerequisites

1. Windows Server 2022 installed in VirtualBox.
2. Active Directory Domain Services configured.
3. DNS configured on the domain controller.
4. Ubuntu Linux installed in VirtualBox.
5. Both virtual machines connected to the same NAT Network.
6. Domain Administrator credentials available.

## Network Architecture

```text
        Oracle VirtualBox
                |
          NAT Network
                |
       -------------------
       |                 |
 Windows Server 2022   Ubuntu Linux
 Domain Controller     Domain Client
       |                 |
   10.0.2.5          changlinux
       |                 |
 Active Directory    Kerberos / SSSD
       |                 |
       -------------------
                |
         changserver.com
                |
      Centralized Authentication
```

---

# Implementation Steps

## Step 1: Configure VirtualBox Networking

<details>
<summary>View Step 1 - Configure Network Adapters</summary>

Configure both virtual machines to use the same VirtualBox NAT Network.

1. Open Oracle VirtualBox.
2. Select Windows Server 2022.
3. Navigate to Settings > Network.
4. Set Adapter 1 to NAT Network.
5. Select the appropriate NAT Network.
6. Repeat for Ubuntu Linux.

Both machines must communicate over the same virtual network.

![VirtualBox Network Configuration](images/step01-network.png)

</details>

## Step 2: Verify Windows Server IP Configuration

<details>
<summary>View Step 2 - Check Domain Controller IP</summary>

On Windows Server 2022, open Command Prompt and execute:

```cmd
ipconfig /all
```

Verify the domain controller's IPv4 address and DNS configuration.

In this lab, the domain controller uses:

```text
IP Address: 10.0.2.5
Domain: changserver.com
```

![Windows Server IP Configuration](images/step02-server-ip.png)

</details>

## Step 3: Verify Ubuntu Network Configuration

<details>
<summary>View Step 3 - Check Ubuntu IP Address</summary>

Open the Ubuntu terminal.

Run:

```bash
ip a
```

Identify the network interface and assigned IPv4 address.

In this lab, the Ubuntu network interface is:

```text
enp0s3
```

Verify that Ubuntu and Windows Server are connected to the same virtual network.

![Ubuntu IP Configuration](images/step03-ubuntu-ip.png)

</details>

## Step 4: Test Connectivity to the Domain Controller

<details>
<summary>View Step 4 - Ping Windows Server</summary>

From Ubuntu, run:

```bash
ping -c 4 10.0.2.5
```

Expected result:

```text
4 packets transmitted
4 received
0% packet loss
```

This confirms IP connectivity between Ubuntu and the Windows Server domain controller.

![Ping Test](images/step04-ping.png)

</details>

## Step 5: Verify DNS Resolution

<details>
<summary>View Step 5 - Resolve Active Directory DNS Records</summary>

Ubuntu must use the Active Directory DNS server to locate the domain controller.

Query the domain controller hostname:

```bash
resolvectl query changserver.com
```

Expected result:

```text
changserver.com: 10.0.2.5
```
If I getting a different ip, it means my dns is not configured correctly.
use this "sudo resolvectl dns enp0s3 10.0.2.5"
        -Note the ip above can be check with ipconfig in window server

Next, verify the LDAP service record:

```bash
resolvectl query -t SRV _ldap._tcp.changserver.com
```

Expected result includes:

```text
_ldap._tcp.changserver.com IN SRV 0 100 389 changserver.changserver.com
```

These records help Ubuntu discover Active Directory services.

![DNS Resolution](images/step05-dns.png)

![LDAP SRV Record](images/step05-ldap.png)

</details>

## Step 6: Update Ubuntu Packages

<details>
<summary>View Step 6 - Update Package Lists</summary>

Update Ubuntu package information:

```bash
sudo apt update
```

Verify the update completes successfully.

![Ubuntu Package Update](images/step06-update.png)

</details>

## Step 7: Install Active Directory Integration Packages

<details>
<summary>View Step 7 - Install realmd, SSSD, and Supporting Packages</summary>

Install the packages required for domain discovery, authentication, and identity resolution:

```bash
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin packagekit -y
```

Key components:

- realmd: Discovers and joins Active Directory domains.
- SSSD: Resolves domain identities and supports authentication.
- Kerberos: Provides domain authentication.
- adcli: Performs Active Directory domain join operations.
- PAM: Integrates authentication into Linux login services.
- Samba: Provides supporting Windows interoperability components.

![Package Installation](images/step07-packages.png)

</details>

## Step 8: Discover the Active Directory Domain

<details>
<summary>View Step 8 - Discover changserver.com</summary>

Run:

```bash
realm discover changserver.com
```

Expected information:

```text
changserver.com
  type: kerberos
  realm-name: CHANGSERVER.COM
  domain-name: changserver.com
  configured: no
  server-software: active-directory
  client-software: sssd
```

This confirms Ubuntu can discover the Active Directory environment.

![Domain Discovery](images/step08-discovery.png)

</details>

## Step 9: Join Ubuntu to Active Directory

<details>
<summary>View Step 9 - Join the Domain</summary>

Execute:

```bash
sudo realm join --user=Administrator changserver.com
```

Enter the Active Directory Administrator password when prompted.

A successful join typically returns to the terminal without an error.

Verify membership:

```bash
realm list
```

Expected output includes:

```text
changserver.com
  type: kerberos
  realm-name: CHANGSERVER.COM
  domain-name: changserver.com
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  login-formats: %U@changserver.com
  login-policy: allow-realm-logins
```

![Domain Join](images/step09-join.png)

![Realm Verification](images/step09-realm.png)

</details>

## Step 10: Verify Active Directory User Resolution

<details>
<summary>View Step 10 - Query a Domain User</summary>

Verify Ubuntu can resolve the Active Directory Administrator account:

```bash
id administrator@changserver.com
```

A successful result displays:

- UID
- GID
- Domain Users group
- Other Active Directory security group memberships

This confirms that SSSD can resolve domain user and group information.

![Domain Identity Verification](images/step10-identity.png)

</details>

## Step 11: Test Administrator Authentication

<details>
<summary>View Step 11 - Authenticate with a Domain Account</summary>

Switch to the domain Administrator account:

```bash
su - administrator@changserver.com
```

Enter the Administrator password.

Verify the authenticated identity:

```bash
whoami
```

Expected:

```text
administrator@changserver.com
```

During the initial test, authentication succeeded but the user's home directory did not exist.

This issue was addressed by enabling automatic home directory creation through PAM.

![Administrator Authentication](images/step11-admin.png)

</details>

## Step 12: Configure Automatic Home Directory Creation

<details>
<summary>View Step 12 - Configure PAM</summary>

Run:

```bash
sudo pam-auth-update
```

Locate:

```text
Create home directory on login
```

Enable this option using the Space key.

Use Tab to select OK, then press Enter.

This configures PAM to automatically create a home directory when a domain user logs in for the first time.

![PAM Configuration](images/step12-pam.png)

![Home Directory Option](images/step12-home.png)

</details>

## Step 13: Verify Administrator Home Directory

<details>
<summary>View Step 13 - Confirm Home Directory Creation</summary>

From the original local Ubuntu account, run:

```bash
su - administrator@changserver.com
```

Then execute:

```bash
pwd
```

Expected:

```text
/home/administrator@changserver.com
```

Verify identity:

```bash
whoami
```

Expected:

```text
administrator@changserver.com
```

This confirms domain authentication and automatic home directory creation.

![Administrator Home Directory](images/step13-admin-home.png)

</details>

## Step 14: Create an Active Directory Test User

<details>
<summary>View Step 14 - Create testguy in Windows Server</summary>

On Windows Server 2022:

1. Open Server Manager.
2. Navigate to Tools.
3. Open Active Directory Users and Computers.
4. Select the Users container.
5. Right-click and select New > User.
6. Create a new account named testguy.
7. Configure the account password.
8. Ensure the account is enabled.

Test account:

```text
testguy@changserver.com
```

![Create AD Test User](images/step14-create-user.png)

</details>

## Step 15: Authenticate the Test User on Ubuntu

<details>
<summary>View Step 15 - Test Active Directory Credentials</summary>

From an Ubuntu terminal, switch to the newly created domain account:

```bash
su - testguy@changserver.com
```

Enter the password configured in Active Directory.

Verify the account:

```bash
whoami
```

Expected:

```text
testguy@changserver.com
```

Check the home directory:

```bash
pwd
```

Expected:

```text
/home/testguy@changserver.com
```

Successful authentication demonstrates that Ubuntu can use credentials managed by Windows Server Active Directory.

![Test User Authentication](images/step15-testguy.png)

![Test User Verification](images/step15-verify.png)

</details>

## Step 16: Verify Kerberos Integration in Ubuntu

<details>
<summary>View Step 16 - Inspect GNOME Online Accounts</summary>

After configuring domain integration, inspect Ubuntu's online account settings.

Navigate to:

```text
Settings > Online Accounts
```

The Kerberos account for the Active Directory test user was visible in the Ubuntu environment.

This provides additional evidence of Kerberos account integration.

Note: GNOME Online Accounts is not itself proof of a successful desktop login. Terminal authentication and domain membership were verified separately.

![Kerberos Account](images/step16-kerberos.png)

</details>

---

# Troubleshooting and Lessons Learned

## Issue 1: Incorrect DNS Name

An initial DNS lookup failed because the queried hostname was incorrect.

Example:

```text
changerserver.changserver.com
```

The correct domain controller name was:

```text
changserver.changserver.com
```

Resolution:

Verified the correct DNS names and queried the Active Directory DNS records.

## Issue 2: Domain Discovery Requirements

Active Directory integration requires functioning DNS records and connectivity to the domain controller.

Resolution:

Verified:

```bash
ping -c 4 10.0.2.5
resolvectl query changserver.com
resolvectl query -t SRV _ldap._tcp.changserver.com
realm discover changserver.com
```

## Issue 3: Missing Linux Home Directory

After authenticating with the domain Administrator account, Ubuntu displayed:

```text
cannot change directory to /home/administrator@changserver.com
```

Cause:

The domain user had not yet been assigned a local Linux home directory.

Resolution:

Executed:

```bash
sudo pam-auth-update
```

Enabled:

```text
Create home directory on login
```

The next domain login successfully created the user's home directory.

## Issue 4: Verifying Domain Authentication

Joining a domain alone does not demonstrate that user authentication works.

Resolution:

Created an additional Active Directory account named testguy and authenticated using:

```bash
su - testguy@changserver.com
```

Verified the account and home directory using:

```bash
whoami
pwd
```

---

# Project Results

The following objectives were achieved:

- [x] Configured VirtualBox networking.
- [x] Established connectivity between Windows Server and Ubuntu.
- [x] Verified Active Directory DNS resolution.
- [x] Verified LDAP SRV records.
- [x] Installed Linux domain integration packages.
- [x] Discovered the Active Directory domain.
- [x] Joined Ubuntu to Windows Server 2022 Active Directory.
- [x] Verified domain membership.
- [x] Resolved Active Directory user identities.
- [x] Authenticated using domain Administrator credentials.
- [x] Configured automatic home directory creation.
- [x] Created an additional Active Directory test account.
- [x] Authenticated the test account on Ubuntu.
- [x] Verified the domain user's Linux home directory.

# Skills Demonstrated

- Windows Server administration
- Active Directory Domain Services
- Linux system administration
- Virtualization with Oracle VirtualBox
- TCP/IP networking
- DNS troubleshooting
- Kerberos authentication
- SSSD configuration
- PAM configuration
- Identity and access management
- Cross-platform infrastructure integration
- Technical troubleshooting and documentation

# Conclusion

This project demonstrates the integration of Ubuntu Linux with a Windows Server 2022 Active Directory environment.

By configuring DNS, Kerberos, SSSD, realmd, and PAM, I successfully established centralized identity resolution and authentication between Windows Server and Linux.

The project also provided hands-on experience troubleshooting DNS, domain membership, and user login issues.

These are foundational skills relevant to IT support, systems administration, and infrastructure engineering.
