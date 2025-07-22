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

🔹Phase 1: Install and Configure Active Directory Domain Services

✅Step 1: Install AD DS on Your Domain Controller

- Log into your Windows Server (future DC)

- Open Server Manager → Add roles and features

- Install Active Directory Domain Services (AD DS)

<img width="786" height="559" alt="Screenshot 2025-07-22 at 1 30 51 PM" src="https://github.com/user-attachments/assets/52214dee-3451-47b4-8d22-a7110a34a4e6" />
<img width="784" height="557" alt="Screenshot 2025-07-22 at 1 35 57 PM" src="https://github.com/user-attachments/assets/329fe07e-0d6c-467f-be55-940c9f33fb2a" />

✅Step 2: Promote the Server to a Domain Controller

- After installation, click “Promote this server to a domain controller”

- Select Add a new forest

- Enter your root domain name (e.g., yourcompany.local)

- Set a DSRM password

- Complete the wizard and restart

<img width="759" height="559" alt="Screenshot 2025-07-22 at 1 54 33 PM" src="https://github.com/user-attachments/assets/c21b530a-7530-4eab-a338-36db697c7cca" />

✅ Step 3: Log In to the Domain

- After reboot, log in using:

yourdomain\Administrator or the account you promoted

🔹 Phase 2: Set Up Organizational Units and Admin Users

✅ Step 4: Create Organizational Units (OUs)

- Open Active Directory Users and Computers (ADUC)

- Create the following OUs by right clicking the domain:

_EMPLOYEES

_ADMINS

_CLIENTS


<img width="753" height="532" alt="Screenshot 2025-07-22 at 2 51 32 PM" src="https://github.com/user-attachments/assets/a03957dd-838b-4c18-9744-fca8ac74b685" />

✅ Step 5: Create an Admin Account
In _ADMINS, create a user:

Name: (your own)

Username: (your own)

Password: (your own)

<img width="750" height="531" alt="Screenshot 2025-07-22 at 2 55 46 PM" src="https://github.com/user-attachments/assets/d75209d6-738c-4af1-b2a0-f9dbf342693c" />

Add this user to the Domain Admins security group by going to the users properties
<img width="412" height="538" alt="Screenshot 2025-07-22 at 2 58 23 PM" src="https://github.com/user-attachments/assets/669bafc0-0027-4d51-abed-3815e0bd246a" />
<img width="457" height="252" alt="Screenshot 2025-07-22 at 2 58 43 PM" src="https://github.com/user-attachments/assets/fc641058-515e-44b4-9c28-f186d1017436" />

Log out and back in as:
yourdomain.com\jane



🧠 From now on, perform all admin tasks using this account.

🔹 Phase 3: Join Client Machines to the Domain

✅ Step 6: Prepare DNS and Network Settings

- On your client machine:

- Set the Preferred DNS Server to the Domain Controller’s IP

- Restart the client to apply settings

(Recommended to do through azure for labs)

✅ Step 7: Join the Client to the Domain
Log in as local admin

System → About → Rename this PC

<img width="1391" height="800" alt="Screenshot 2025-07-22 at 3 17 17 PM" src="https://github.com/user-attachments/assets/89404c6b-a660-4921-b297-ecd04b6cfc03" />

Click Change... → Select “Domain” and enter: yourdomain.local

- Authenticate using domain admin credentials (e.g., jane_admin)

<img width="1385" height="798" alt="Screenshot 2025-07-22 at 3 18 27 PM" src="https://github.com/user-attachments/assets/e20bab36-99d2-4fa6-9de1-25260f192e4e" />

Restart after joining

✅ Step 8: Verify the Join in ADUC
On the Domain Controller:

- Open ADUC

- Locate the new computer object (usually under Computers)

- Drag it into the _CLIENTS OU (Optional)

🔹 Phase 4: Enable RDP Access for Domain Users
✅ Step 9: Allow Remote Desktop for Domain Users

- Log into the client as yourdomain\jane_admin

Open:

System Properties → Remote Desktop

<img width="1389" height="798" alt="Screenshot 2025-07-22 at 3 44 44 PM" src="https://github.com/user-attachments/assets/0c939294-f5d0-422b-8cd0-b3b9bd402cf5" />

Click Select Users → Add → Domain Users


<img width="374" height="333" alt="Screenshot 2025-07-22 at 3 45 24 PM" src="https://github.com/user-attachments/assets/04b8fb36-a779-4957-97a6-5b33def3ddda" />

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

Use the password defined in the script



