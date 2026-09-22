# Enterprise SOC Lab - Infrastructure Setup

## 1. Overview

This document describes the initial infrastructure deployed for the Enterprise SOC Lab.

The objective of this phase was to build a small Windows enterprise environment that can later generate realistic authentication, endpoint, and Active Directory telemetry for security monitoring and investigation.

The environment currently consists of:

- `DC01` - Windows Server Domain Controller
- `WKS01` - Windows 11 domain workstation
- Active Directory Domain Services
- Internal DNS
- Organizational Units (OUs)
- Domain users and security groups
- Delegated administrative permissions
- RSAT administration from WKS01

Domain:

`ad.softelia.lab`

Internal network:

`10.10.10.0/24`

---

## 2. Network Architecture

The internal laboratory network uses VMware VMnet2.

VMnet2 is configured as a host-only network:

- Network: `10.10.10.0/24`
- DHCP: Disabled
- Internet access: Not provided directly
- Purpose: Isolated enterprise network

### DC01

- Hostname: `DC01`
- IP address: `10.10.10.10`
- Role: Domain Controller
- Services: Active Directory Domain Services and DNS

### WKS01

- Hostname: `WKS01`
- Internal IP address: `10.10.10.20`
- Domain: `ad.softelia.lab`
- Primary internal DNS: `10.10.10.10`

A temporary second VMware NAT interface was added to WKS01 to provide Internet connectivity for the installation of RSAT while maintaining connectivity with the isolated Active Directory network.

---

## 3. Active Directory Structure

The Active Directory environment was organized under the `Softelia` OU.

The structure includes OUs for:

- Admin
- Groups
- Servers
- Users
- Workstations

Departmental OUs were created under Users, including:

- Development
- Finance
- HR
- IT
- Sales

This structure separates directory objects according to their administrative purpose and allows policies and delegated permissions to be applied at appropriate scopes.

---

## 4. Users and Security Groups

Several test users were created to represent employees from different departments.

Security groups were used to assign permissions based on roles instead of assigning permissions directly to individual users.

Examples include:

- `GG-IT-Users`
- `GG-Finance-Users`
- `GG-Helpdesk-PasswordReset`

This approach simplifies permission management and follows role-based access principles.

---

## 5. Separation of Administrative Accounts

The lab separates normal user activity from privileged administrative activity.

Example:

`dtorres`

Standard account used for normal workstation activity.

`adm-dtorres`

Separate administrative account used only when privileged operations are required.

The administrative account was not made a Domain Administrator.

Instead, specific permissions were delegated through the security group:

`GG-Helpdesk-PasswordReset`

This reduces the exposure of privileged credentials and follows the principle of least privilege.

---

## 6. Delegated Active Directory Administration

Password reset permissions were delegated to the help desk administrative group.

The objective was to allow an administrator such as `adm-dtorres` to perform a specific support task without granting unrestricted administrative access to Active Directory.

Two tests were performed.

### Authorized operation

Using RSAT from WKS01 with the `adm-dtorres` account, the password of the test user Laura Mendez was successfully reset.

Result:

`SUCCESS`

This confirmed that the delegated password reset permission was functioning.

### Unauthorized operation

From a PowerShell session running as:

`SOFTELIA\adm-dtorres`

an attempt was made to create a new Active Directory user using `New-ADUser`.

Active Directory returned:

`Access denied`

and:

`UnauthorizedAccessException`

Result:

`DENIED`

This confirmed that the account could perform its delegated function but could not perform unrelated administrative operations.

---

## 7. RSAT Administration

Remote Server Administration Tools (RSAT) were installed on WKS01.

Active Directory Users and Computers could then be executed using the dedicated administrative account:

`SOFTELIA\adm-dtorres`

This allows Active Directory administration from an administrative workstation without requiring routine interactive access to the Domain Controller.

---

## 8. Security Principles Demonstrated

This phase of the lab demonstrates several enterprise security concepts:

- Active Directory domain administration
- Organizational Unit design
- Role-based security groups
- Separation of standard and privileged accounts
- Principle of least privilege
- Delegated administration
- Administrative access using RSAT
- DNS dependency of Active Directory
- Network segmentation
- Positive and negative authorization testing

The environment will serve as the identity and endpoint foundation for later SOC monitoring, detection engineering, and incident investigation exercises.