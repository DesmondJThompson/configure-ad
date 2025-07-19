<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>

This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used</h2>

- Windows Server 2022
- Windows 10 (21H2)

## Prerequisites
- Azure account with resource deployment permissions
- Two VMs created and running in the same virtual network:
- DC-1 (Windows Server)
- Client-1 (Windows 10 or 11)
- DNS for Client-1 must point to DC-1's **private IP**
- Static IP preferred for DC-1

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Install AD DS on Your Domain Controller 
- Step 2: Promote the Server to a Domain Controller
- Step 3: Log In to the Domain
- Step 4: Create OU's (Organizational Units)
- Step 5: Create an Admin Account
- Step 6: Prepare DNS and Network Settings on your client machine
- Step 7: Join the client to the domain, login as local admin
- Step 8: Verify the Join in ADUC on the Domain Controller
- Step 9: Allow Remote Desktop for Domain Users/Log into the client as yourdomain\jane_admin
- Step 10: Create Multiple Users via PowerShell (OPTIONAL)
- Step 11: Test Login with New Domain Users on the client

<h2>Deployment and Configuration Steps</h2>

Phase 1: Install and Configure Active Directory Domain Services

✅ Step 1: Install AD DS on Your Domain Controller
Log into your Windows Server (future DC)

Open Server Manager → Add roles and features

Install Active Directory Domain Services (AD DS)

✅ Step 2: Promote the Server to a Domain Controller
After installation, click “Promote this server to a domain controller”

Select Add a new forest

Enter your root domain name (e.g., yourcompany.local)

Set a DSRM password

Complete the wizard and restart

✅ Step 3: Log In to the Domain
After reboot, log in using:
yourdomain\Administrator or the account you promoted

🔹 Phase 2: Set Up Organizational Units and Admin Users
✅ Step 4: Create Organizational Units (OUs)
Open Active Directory Users and Computers (ADUC)

Create the following OUs:

_EMPLOYEES

_ADMINS

_CLIENTS

✅ Step 5: Create an Admin Account
In _ADMINS, create a user:

Name: Jane Doe

Username: jane_admin

Password: Cyberlab123! (or your own)

Add this user to the Domain Admins security group

Log out and back in as:
yourdomain\jane_admin

🧠 From now on, perform all admin tasks using this account.

🔹 Phase 3: Join Client Machines to the Domain
✅ Step 6: Prepare DNS and Network Settings
On your client machine:

Set the Preferred DNS Server to the Domain Controller’s IP

Restart the client to apply settings

✅ Step 7: Join the Client to the Domain
Log in as local admin

Right-click This PC → Properties → Change settings

Click Change... → Select “Domain” and enter: yourdomain.local

Authenticate using domain admin credentials (e.g., jane_admin)

Restart after joining

✅ Step 8: Verify the Join in ADUC
On the Domain Controller:

Open ADUC

Locate the new computer object (usually under Computers)

Drag it into the _CLIENTS OU

🔹 Phase 4: Enable RDP Access for Domain Users
✅ Step 9: Allow Remote Desktop for Domain Users
Log into the client as yourdomain\jane_admin

Open:

System Properties → Remote Desktop

Click Select Users → Add → Domain Users

🔐 Best practice: Do this with Group Policy if managing many systems.

🔹 Phase 5: Bulk User Creation and Login Testing
✅ Step 10: Create Multiple Users via PowerShell (OPTIONAL)
On the Domain Controller, open PowerShell ISE as Administrator

Use a script like this to create users in _EMPLOYEES:

powershell
Copy
Edit
for ($i=1; $i -le 10; $i++) {
    $username = "user$i"
    $password = ConvertTo-SecureString "Password123!" -AsPlainText -Force
    New-ADUser -Name $username -AccountPassword $password -Path "OU=_EMPLOYEES,DC=yourdomain,DC=local" -Enabled $true
}
✅ Step 11: Test Login with New Domain Users on the client:

Log out

Attempt to log in with:
yourdomain\user1

Use the password defined in your script

✅ You’ve now confirmed domain authentication and remote access.


