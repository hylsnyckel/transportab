## Skämt & Sido Transport AB
Windows Server Active Directory & File Sharing Setup
## 1. Project Overview

This repository documents the implementation of a Windows Server infrastructure for Skämt & Sido Transport AB.

The project focuses on centralized identity management, access control, DNS, and department-based file sharing.

## 2. Environment Overview
Domain name: ad.skamt.se
Organizational Structure
text
GBG
├── Staff
│   └── User accounts
└── Groups
    └── Security groups

## Infrastructure Components
Windows Server with Active Directory Domain Services (AD DS)
DNS for domain name resolution
Linux server and client systems on the same network
Shared folders with access controlled by security groups
## 3. Active Directory Users and Security Groups

User accounts are organized under GBG/Staff. Security groups are organized under GBG/Groups.

User	Role	Security Groups
Anna Ekonomi	Finance	Ekonomi, Gemensamt
Bob Boss	Management	Ledning, Gemensamt
Carl Logistik	Logistics / Warehouse	Logistik, Gemensamt
Lars IT	IT Administrator	IT, Gemensamt


These groups provide department-based access to shared resources. Administrative privileges should be restricted to authorized personnel.

## 4. Shared Folders and Permissions

The main shared directory is named Shared. It contains separate folders for departments and common resources.

Shared/
├── Ekonomi/
├── Gemensamt/
├── IT/
├── Ledning/
└── Logistik/

User	Role	Security Groups
Anna Ekonomi	Finance	Ekonomi, Gemensamt
Bob Boss	Management	Ledning, Gemensamt
Carl Logistik	Logistics / Warehouse	Logistik, Gemensamt
Lars IT	IT Administrator	IT, Gemensamt

Access is managed using Active Directory security groups. 

## 5. Services and Technologies
Windows Server
Active Directory Domain Services (AD DS)
DNS (Domain Name System)
Organizational Units (OUs)
Active Directory security groups
File sharing
PowerShell administration
Group Policy Objects (GPOs)

## 6. Implementation Checklist
Install and configure Windows Server.
Configure network settings and a static IP address.
Install Active Directory Domain Services (AD DS).
Promote the server to a domain controller.
Configure DNS and verify domain name resolution.
Create the domain structure and organizational units.
Create user accounts and security groups.
Configure shared folders and access permissions.
Test access with different user accounts.
Configure and test relevant Group Policy Objects.
Document configuration decisions and troubleshooting steps.

## 7. Testing and Troubleshooting

Verify that domain users can authenticate successfully, DNS resolution works correctly, and shared folders enforce the intended access restrictions.

Test both authorized and unauthorized access to confirm that permissions are applied correctly.

Document installation, domain, or network issues and the steps taken to resolve them.


## 8. Security Considerations
Apply the principle of least privilege.
Assign folder permissions through security groups wherever practical.
Restrict administrative privileges to authorized personnel.
Regularly review user memberships and folder permissions.
Test access restrictions using different user accounts.

## 9. Expected Outcome

The project aims to provide centralized identity management, reliable DNS resolution, structured user and group administration, and secure department-based file sharing for Skämt & Sido Transport AB.