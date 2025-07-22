
<p align="center">
  <img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo" />
</p>

<h1 align="center">On-Premises Active Directory Deployed in the Cloud (Azure)</h1>

This tutorial demonstrates how to deploy and configure a fully functional, on-premises-style Active Directory environment in the cloud using Microsoft Azure. You'll walk through installing and promoting a domain controller, configuring organizational units and users, and joining a Windows 10 client to the domain. This lab simulates a real-world enterprise environment using cloud infrastructure and is ideal for IT professionals building hands-on Active Directory experience.

---

## 🧰 Environments and Technologies Used

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop Protocol (RDP)
- Active Directory Domain Services (AD DS)
- PowerShell

## 💻 Operating Systems Used

- Windows Server 2022 (DC-1)
- Windows 10 (21H2) (Client-1)

---

## ✅ Prerequisites

- Azure account with permission to deploy and configure virtual machines
- Two VMs created in the same Virtual Network:
  - `DC-1` (Windows Server)
  - `Client-1` (Windows 10 or 11)
- DNS on `Client-1` must point to the private IP of `DC-1`
- Static IP is preferred for `DC-1`

---

## 🛠️ High-Level Deployment and Configuration Steps

- Step 1: Install AD DS on your Domain Controller  
- Step 2: Promote the server to a Domain Controller  
- Step 3: Log in to the domain  
- Step 4: Create Organizational Units (OUs)  
- Step 5: Create an admin account  
- Step 6: Prepare DNS and network settings on your client  
- Step 7: Join the client to the domain  
- Step 8: Verify the join in ADUC on the Domain Controller  
- Step 9: Enable Remote Desktop access for Domain Users  
- Step 10: Create multiple users via PowerShell (optional)  
- Step 11: Test login with one of the new domain users  

---

## Deployment and Configuration Steps

### 🔹 Phase 1: Install and Configure AD DS

#### ✅ Step 1: Install AD DS on DC-1

- Log into `DC-1`
- Open Server Manager → Add Roles and Features
- Select Active Directory Domain Services

![Install ADDS](https://github.com/user-attachments/assets/52214dee-3451-47b4-8d22-a7110a34a4e6)
![Add Roles](https://github.com/user-attachments/assets/329fe07e-0d6c-467f-be55-940c9f33fb2a)

---

#### ✅ Step 2: Promote to Domain Controller

- Click "Promote this server to a domain controller"
- Select: "Add a new forest"
- Set your domain (e.g. `yourdomain.local`)
- Choose a DSRM password
- Complete the wizard and restart the VM

![Promote to DC](https://github.com/user-attachments/assets/c21b530a-7530-4eab-a338-36db697c7cca)

---

#### ✅ Step 3: Log In to the Domain

After reboot, log in as:

```
yourdomain\Administrator
```

---

### 🔹 Phase 2: Create OUs and Admin Users

#### ✅ Step 4: Create Organizational Units

- Open Active Directory Users and Computers (ADUC)
- Right-click your domain and create:
  - `_EMPLOYEES`
  - `_ADMINS`
  - `_CLIENTS`

![Create OUs](https://github.com/user-attachments/assets/a03957dd-838b-4c18-9744-fca8ac74b685)

---

#### ✅ Step 5: Create an Admin Account

- In `_ADMINS`, create a user (e.g. `jane_admin`)
- Set a secure password
- Right-click the user → Properties → "Member Of"
- Add the user to **Domain Admins**

![Create Admin](https://github.com/user-attachments/assets/d75209d6-738c-4af1-b2a0-f9dbf342693c)
![Member Of](https://github.com/user-attachments/assets/669bafc0-0027-4d51-abed-3815e0bd246a)
![Add to Domain Admins](https://github.com/user-attachments/assets/fc641058-515e-44b4-9c28-f186d1017436)

🔁 **Log out and log back in as:**

```
yourdomain\jane_admin
```

 From this point on, use `jane_admin` for all domain admin tasks.

---

### 🔹 Phase 3: Join Client Machine to the Domain

#### ✅ Step 6: Set DNS on Client-1

- Go to Azure Portal → Client-1 → Networking
- Click the Network Interface → DNS servers → Select "Custom"
- Set Preferred DNS to DC-1’s private IP
- Save and restart the VM

 **Alternative (inside Windows):**  
Go to "Network Adapter Properties" → TCP/IPv4 → Set Preferred DNS manually

---

#### ✅ Step 7: Join the Client to the Domain

- Log in to Client-1 as `labuser`
- Go to: System → About → Rename this PC
- Click "Change..." → Select "Domain"
- Enter your domain name (e.g. `yourdomain.local`)
- Authenticate using: `jane_admin`

![Rename PC](https://github.com/user-attachments/assets/89404c6b-a660-4921-b297-ecd04b6cfc03)
![Join Domain](https://github.com/user-attachments/assets/e20bab36-99d2-4fa6-9de1-25260f192e4e)

🔁 Restart the VM

---

#### ✅ Step 8: Verify the Domain Join

- On DC-1, open ADUC
- Confirm that `Client-1` shows up under "Computers"
- Drag it into the `_CLIENTS` OU (optional)

---

### 🔹 Phase 4: Enable RDP for Domain Users

#### ✅ Step 9: Enable RDP Access on Client-1

- Log into Client-1 as `yourdomain\jane_admin`
- Open: System Properties → Remote Desktop
- Enable RDP → Click "Select Users"
- Add: `Domain Users`

![RDP Access](https://github.com/user-attachments/assets/0c939294-f5d0-422b-8cd0-b3b9bd402cf5)
![Add Users](https://github.com/user-attachments/assets/04b8fb36-a779-4957-97a6-5b33def3ddda)

📌 **Note:** In production environments, configure this via Group Policy

---

### 🔹 Phase 5: Bulk User Creation

#### ✅ Step 10: Create Users via PowerShell

- On DC-1, open PowerShell ISE as Administrator  
- Paste and run this script:

```powershell
for ($i=1; $i -le 10; $i++) {
    $username = "user$i"
    $password = ConvertTo-SecureString "Password123!" -AsPlainText -Force
    New-ADUser -Name $username `
               -AccountPassword $password `
               -Path "OU=_EMPLOYEES,DC=yourdomain,DC=local" `
               -Enabled $true `
               -SamAccountName $username `
               -UserPrincipalName "$username@yourdomain.local"
}
```

---

#### ✅ Step 11: Test Domain Login on Client-1

- Log out of Client-1
- At the login screen, click "Other User"
- Log in using one of the new accounts:

```
yourdomain\user1
Password123!
```





