---
name: enterprise-windows-infrastructure-security
description: Obsidian-based knowledge base for Windows Server administration, Active Directory, Group Policy, and defensive hardening with hands-on labs.
triggers:
  - how do I harden Active Directory
  - show me Windows Server security best practices
  - help me set up a Windows enterprise lab
  - how do I configure Group Policy for security
  - what are the steps to secure DNS and DHCP
  - guide me through Windows Server defensive hardening
  - how do I build a purple team Windows lab
  - show me enterprise Windows security configurations
---

# Enterprise Windows Infrastructure Security Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Enterprise Windows Infrastructure & Security is a comprehensive, lab-driven curriculum delivered as an Obsidian knowledge base for Windows Server administration and defensive hardening. It covers the full enterprise stack: Windows Server OS, PowerShell, Active Directory Domain Services, Group Policy, DNS, DHCP, file services, IIS, remote access, and purple-team security validation.

**Key Features:**
- 20 modules progressing from fundamentals to advanced hardening
- Hands-on labs using a `corp.local` domain topology
- Security-first approach with hardening integrated into every service
- PowerShell automation and command-line administration
- Purple-team practice with attack/defense scenarios
- Multi-VM lab environment (DC, member server, client, attacker)

## Installation & Setup

### Clone the Repository

```bash
# Clone the knowledge base
git clone https://github.com/armourinfosec/Enterprise-Windows-Infrastructure-Security.git
cd Enterprise-Windows-Infrastructure-Security
```

### Open in Obsidian

1. Download and install [Obsidian](https://obsidian.md/)
2. Open Obsidian and select "Open folder as vault"
3. Navigate to the cloned repository directory
4. Start with the main README and follow the learning path sequentially

### Lab Environment Setup

The curriculum requires a virtualization environment. Minimum setup:

```plaintext
Hardware Requirements:
- CPU: 4+ cores with VT-x/AMD-V enabled
- RAM: 16 GB recommended
- Disk: 250 GB+ SSD
- Hypervisor: VirtualBox 7.x, Hyper-V, or VMware Workstation

VM Topology (corp.local domain):
- DC01: Windows Server 2022 (AD DS, DNS, DHCP, GPO)
- SRV01: Windows Server 2022 (IIS, File Services, DFS, FTP)
- WKS01: Windows 10/11 (domain-joined client)
- Kali: Linux (attack/defense validation)

Network: 10.10.10.0/24 isolated internal network
```

## Learning Path Structure

The knowledge base is organized into 6 stages with 20 modules:

### Stage 1: Foundations
- Module 1: Fundamentals of the Operating System
- Module 2: Windows OS Administration
- Module 3: Windows Commands
- Module 4: Windows PowerShell

### Stage 2: Lab & Networking
- Module 5: Lab Setup & Virtualization
- Module 6: Networking Fundamentals

### Stage 3: Directory & Core Services
- Module 7: Active Directory Domain Services (AD DS)
- Module 8: Group Policy Objects (GPO)
- Module 9: Domain Name System (DNS)
- Module 10: Dynamic Host Configuration Protocol (DHCP)

### Stage 4: Infrastructure Services
- Module 11: File Services & DFS
- Module 12: Web Server (IIS)
- Module 13: FTP Server Administration
- Module 14: Proxy Server Administration
- Module 15: Remote Access & VPN

### Stage 5: Operate & Resilience
- Module 16: Server Management
- Module 17: Backup/Restore/Recovery
- Module 18: Monitoring & Logging

### Stage 6: Security & Practice
- Module 19: Enterprise Security (purple team)
- Module 20: Software Development Life Cycle

## Key Commands & PowerShell Patterns

### Active Directory Administration

```powershell
# Import Active Directory module
Import-Module ActiveDirectory

# Create organizational unit
New-ADOrganizationalUnit -Name "IT" -Path "DC=corp,DC=local" `
    -ProtectedFromAccidentalDeletion $true

# Create user with secure password
$Password = Read-Host -AsSecureString "Enter password"
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" `
    -SamAccountName "jsmith" -UserPrincipalName "jsmith@corp.local" `
    -Path "OU=IT,DC=corp,DC=local" -AccountPassword $Password `
    -Enabled $true -ChangePasswordAtLogon $true

# Create security group
New-ADGroup -Name "IT-Admins" -GroupScope Global `
    -GroupCategory Security -Path "OU=IT,DC=corp,DC=local"

# Add user to group
Add-ADGroupMember -Identity "IT-Admins" -Members "jsmith"

# Query domain controllers
Get-ADDomainController -Filter * | Select-Object Name, IPv4Address, Site

# Check replication status
Get-ADReplicationPartnerMetadata -Target "DC01.corp.local" `
    | Select-Object Partner, LastReplicationSuccess
```

### Group Policy Management

```powershell
# Import Group Policy module
Import-Module GroupPolicy

# Create new GPO
New-GPO -Name "Workstation-Hardening" -Comment "Security baseline for workstations"

# Link GPO to OU
New-GPLink -Name "Workstation-Hardening" `
    -Target "OU=Workstations,DC=corp,DC=local" -LinkEnabled Yes

# Set registry-based policy
Set-GPRegistryValue -Name "Workstation-Hardening" `
    -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
    -ValueName "EnableLUA" -Type DWord -Value 1

# Configure password policy
Set-ADDefaultDomainPasswordPolicy -Identity corp.local `
    -MinPasswordLength 14 -PasswordHistoryCount 24 `
    -MaxPasswordAge (New-TimeSpan -Days 90) `
    -MinPasswordAge (New-TimeSpan -Days 1) `
    -ComplexityEnabled $true

# Generate GPO report
Get-GPOReport -Name "Workstation-Hardening" -ReportType Html `
    -Path "C:\Reports\Workstation-Hardening.html"

# Force Group Policy update on remote computer
Invoke-GPUpdate -Computer "WKS01" -Force -RandomDelayInMinutes 0
```

### DNS Configuration

```powershell
# Create forward lookup zone
Add-DnsServerPrimaryZone -Name "corp.local" -ReplicationScope Domain

# Create reverse lookup zone
Add-DnsServerPrimaryZone -NetworkId "10.10.10.0/24" -ReplicationScope Domain

# Add DNS A record
Add-DnsServerResourceRecordA -Name "srv01" -ZoneName "corp.local" `
    -IPv4Address "10.10.10.11" -CreatePtr

# Add DNS CNAME record
Add-DnsServerResourceRecordCName -Name "intranet" -ZoneName "corp.local" `
    -HostNameAlias "srv01.corp.local"

# Configure conditional forwarder
Add-DnsServerConditionalForwarderZone -Name "partner.com" `
    -MasterServers "192.168.1.10" -ReplicationScope Forest

# Enable DNS query logging
Set-DnsServerDiagnostics -Queries $true -QueryErrors $true `
    -LogFilePath "C:\Windows\System32\dns\dns.log"

# Clear DNS cache
Clear-DnsServerCache -Force
```

### DHCP Server Management

```powershell
# Install DHCP role
Install-WindowsFeature DHCP -IncludeManagementTools

# Authorize DHCP server in AD
Add-DhcpServerInDC -DnsName "DC01.corp.local" -IPAddress "10.10.10.10"

# Create DHCP scope
Add-DhcpServerv4Scope -Name "Corporate-LAN" `
    -StartRange "10.10.10.100" -EndRange "10.10.10.200" `
    -SubnetMask "255.255.255.0" -State Active

# Set scope options
Set-DhcpServerv4OptionValue -ScopeId "10.10.10.0" `
    -Router "10.10.10.1" -DnsServer "10.10.10.10" `
    -DnsDomain "corp.local"

# Create DHCP reservation
Add-DhcpServerv4Reservation -ScopeId "10.10.10.0" `
    -IPAddress "10.10.10.50" -ClientId "00-15-5D-01-02-03" `
    -Description "Print-Server"

# Configure DHCP failover
Add-DhcpServerv4Failover -Name "DC01-DC02-Failover" `
    -PartnerServer "DC02.corp.local" -ScopeId "10.10.10.0" `
    -LoadBalancePercent 50 -MaxClientLeadTime 1:00:00
```

### File Services & Share Management

```powershell
# Create shared folder
New-Item -Path "C:\Shares\IT-Data" -ItemType Directory
New-SmbShare -Name "IT-Data" -Path "C:\Shares\IT-Data" `
    -FullAccess "CORP\IT-Admins" -ReadAccess "CORP\Domain Users"

# Set NTFS permissions
$Acl = Get-Acl "C:\Shares\IT-Data"
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "CORP\IT-Admins", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow"
)
$Acl.SetAccessRule($AccessRule)
Set-Acl "C:\Shares\IT-Data" $Acl

# Install DFS Namespace role
Install-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication `
    -IncludeManagementTools

# Create DFS namespace
New-DfsnRoot -Path "\\corp.local\Files" -TargetPath "\\SRV01\DFSRoot" `
    -Type DomainV2

# Add DFS folder target
New-DfsnFolder -Path "\\corp.local\Files\IT-Data" `
    -TargetPath "\\SRV01\IT-Data"

# Configure DFS replication
New-DfsReplicationGroup -GroupName "IT-Data-Replication"
Add-DfsrMember -GroupName "IT-Data-Replication" `
    -ComputerName "SRV01", "SRV02"
```

### IIS Web Server Configuration

```powershell
# Install IIS with common features
Install-WindowsFeature Web-Server, Web-Mgmt-Tools, Web-Scripting-Tools `
    -IncludeManagementTools

# Import WebAdministration module
Import-Module WebAdministration

# Create new website
New-Website -Name "IntranetSite" -Port 443 -Protocol https `
    -PhysicalPath "C:\inetpub\intranet" `
    -ApplicationPool "IntranetAppPool" -Force

# Create application pool with specific identity
New-WebAppPool -Name "IntranetAppPool"
Set-ItemProperty "IIS:\AppPools\IntranetAppPool" -Name "processModel.identityType" `
    -Value "SpecificUser"
Set-ItemProperty "IIS:\AppPools\IntranetAppPool" -Name "processModel.userName" `
    -Value "CORP\IIS-AppPool"

# Bind SSL certificate
$Cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {
    $_.Subject -like "*intranet.corp.local*"
}
New-WebBinding -Name "IntranetSite" -Protocol https -Port 443 -SslFlags 0
$Binding = Get-WebBinding -Name "IntranetSite" -Protocol https
$Binding.AddSslCertificate($Cert.Thumbprint, "my")

# Configure security headers
Set-WebConfigurationProperty -Filter "system.webServer/httpProtocol/customHeaders" `
    -PSPath "IIS:\Sites\IntranetSite" -Name "." -Value @{
        name='X-Content-Type-Options'; value='nosniff'
    }
```

### Security Hardening Patterns

```powershell
# Enable Windows Firewall on all profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Configure firewall rule for RDP with source restriction
New-NetFirewallRule -DisplayName "RDP-AdminSubnet" -Direction Inbound `
    -Protocol TCP -LocalPort 3389 -Action Allow `
    -RemoteAddress "10.10.10.0/24" -Profile Domain

# Disable SMBv1 protocol
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Enable PowerShell script block logging
$RegPath = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $RegPath -Force
Set-ItemProperty -Path $RegPath -Name "EnableScriptBlockLogging" -Value 1

# Configure audit policy for account logon
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Kerberos Authentication Service" /success:enable /failure:enable

# Enable LSA Protection (RunAsPPL)
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "RunAsPPL" -Value 1 -PropertyType DWORD -Force

# Disable LLMNR and NetBIOS
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" `
    -Name "EnableMulticast" -Value 0 -PropertyType DWORD -Force
Get-NetAdapter | ForEach-Object {
    Set-NetAdapterBinding -Name $_.Name -ComponentID ms_tcpip -Enabled $false
}

# Configure LAPS (Local Administrator Password Solution)
Import-Module AdmPwd.PS
Update-AdmPwdADSchema
Set-AdmPwdComputerSelfPermission -Identity "Workstations"
```

### Security Monitoring & Logging

```powershell
# Configure advanced audit policy
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
auditpol /set /category:"Object Access" /success:enable /failure:enable
auditpol /set /category:"Policy Change" /success:enable /failure:enable
auditpol /set /category:"Privilege Use" /success:enable /failure:enable
auditpol /set /category:"System" /success:enable /failure:enable

# Increase Security event log size
wevtutil sl Security /ms:1073741824  # 1 GB

# Query security events for failed logons
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625  # Failed logon
    StartTime=(Get-Date).AddHours(-24)
} | Select-Object TimeCreated, Message | Format-Table -AutoSize

# Export security logs
wevtutil epl Security "C:\Logs\Security-$(Get-Date -Format 'yyyyMMdd').evtx"

# Configure Sysmon for enhanced logging
# Download Sysmon from Sysinternals and use SwiftOnSecurity config
sysmon64.exe -accepteula -i sysmonconfig-export.xml

# Query Sysmon events
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1  # Process creation
    StartTime=(Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Message
```

## Purple Team Attack & Defense Scenarios

### Reconnaissance Detection

```powershell
# Monitor for AD enumeration attempts
# Check for unusual LDAP queries from non-admin accounts
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=1644  # LDAP query
    StartTime=(Get-Date).AddHours(-24)
} | Where-Object {
    $_.Message -notmatch "Domain Admins|Enterprise Admins"
} | Format-Table TimeCreated, Message

# Detect BloodHound/SharpHound activity
# Look for rapid LDAP queries from a single source
$Events = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4662  # Directory Service Access
    StartTime=(Get-Date).AddMinutes(-10)
}
$Events | Group-Object {$_.Properties[1].Value} | 
    Where-Object {$_.Count -gt 100} | 
    Select-Object Name, Count
```

### Credential Theft Defense

```powershell
# Enable Credential Guard
# Requires UEFI, Secure Boot, and TPM 2.0
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" `
    -Name "EnableVirtualizationBasedSecurity" -Value 1 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LsaCfgFlags" -Value 1 -PropertyType DWORD -Force

# Restrict NTLM authentication
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "RestrictSendingNTLMTraffic" -Value 2  # Deny all

# Detect Mimikatz execution
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
    StartTime=(Get-Date).AddHours(-1)
} | Where-Object {
    $_.Message -match "mimikatz|sekurlsa|lsadump"
}
```

### Lateral Movement Detection

```powershell
# Monitor for PsExec usage
Get-WinEvent -FilterHashtable @{
    LogName='System'
    ProviderName='Service Control Manager'
    Id=7045  # Service installed
    StartTime=(Get-Date).AddHours(-24)
} | Where-Object {
    $_.Message -match "PSEXESVC"
}

# Detect WMI lateral movement
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-WMI-Activity/Operational'
    Id=5857,5858,5859,5860,5861
    StartTime=(Get-Date).AddHours(-24)
} | Format-Table TimeCreated, Id, Message

# Monitor for remote PowerShell sessions
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-PowerShell/Operational'
    Id=4103,4104  # Script block logging
    StartTime=(Get-Date).AddHours(-1)
} | Where-Object {
    $_.Message -match "Enter-PSSession|Invoke-Command"
}
```

## Common Workflows

### Deploy a Secure AD Domain

```powershell
# 1. Install AD DS role on DC01
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 2. Promote to domain controller
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "corp.local" `
    -DomainNetbiosName "CORP" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (Read-Host -AsSecureString "Enter DSRM password") `
    -Force:$true

# 3. Configure DNS forwarders
Add-DnsServerForwarder -IPAddress "8.8.8.8", "8.8.4.4"

# 4. Create OU structure
New-ADOrganizationalUnit -Name "Corp" -Path "DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Users" -Path "OU=Corp,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "OU=Corp,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Servers" -Path "OU=Corp,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Groups" -Path "OU=Corp,DC=corp,DC=local"

# 5. Apply security baseline GPO
New-GPO -Name "Domain-Security-Baseline"
New-GPLink -Name "Domain-Security-Baseline" -Target "DC=corp,DC=local"

# 6. Configure default domain password policy
Set-ADDefaultDomainPasswordPolicy -Identity corp.local `
    -MinPasswordLength 14 `
    -ComplexityEnabled $true `
    -PasswordHistoryCount 24 `
    -LockoutThreshold 5 `
    -LockoutDuration (New-TimeSpan -Minutes 30) `
    -MaxPasswordAge (New-TimeSpan -Days 90)
```

### Join a Member Server to Domain

```powershell
# On SRV01 - Configure network settings
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "10.10.10.11" `
    -PrefixLength 24 -DefaultGateway "10.10.10.1"
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
    -ServerAddresses "10.10.10.10"

# Join domain
Add-Computer -DomainName "corp.local" `
    -Credential (Get-Credential CORP\Administrator) `
    -Restart -Force
```

### Backup and Restore Active Directory

```powershell
# Install Windows Server Backup
Install-WindowsFeature Windows-Server-Backup

# Perform system state backup (includes AD)
wbadmin start systemstatebackup -backupTarget:E: -quiet

# Schedule daily AD backup
$Action = New-ScheduledTaskAction -Execute "wbadmin.exe" `
    -Argument "start systemstatebackup -backupTarget:E: -quiet"
$Trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName "AD-Daily-Backup" `
    -Action $Action -Trigger $Trigger `
    -User "SYSTEM" -RunLevel Highest

# Restore from system state backup
wbadmin start systemstaterecovery -version:01/15/2024-02:00 -backupTarget:E: -quiet

# Perform authoritative restore of deleted OU
Restart-Computer -Force  # Boot into Directory Services Restore Mode
ntdsutil "activate instance ntds" "authoritative restore" `
    "restore subtree OU=IT,DC=corp,DC=local" quit quit
```

## Troubleshooting

### Active Directory Replication Issues

```powershell
# Check replication status
repadmin /replsummary
repadmin /showrepl

# Force replication between DCs
repadmin /syncall /AdeP

# Test replication health
dcdiag /test:replications /v

# Check DNS configuration
dcdiag /test:dns /v
```

### Group Policy Not Applying

```powershell
# Check GPO links and inheritance
Get-GPInheritance -Target "OU=Workstations,DC=corp,DC=local"

# View resultant set of policy on remote computer
gpresult /S WKS01 /H C:\Reports\gpresult-WKS01.html /F

# Force immediate policy refresh
Invoke-GPUpdate -Computer "WKS01" -Force -RandomDelayInMinutes 0

# Check for GPO processing errors in event log
Get-WinEvent -ComputerName WKS01 -FilterHashtable @{
    LogName='System'
    ProviderName='Microsoft-Windows-GroupPolicy'
    Level=2  # Error
} | Select-Object TimeCreated, Message
```

### DNS Resolution Problems

```powershell
# Test DNS resolution
Resolve-DnsName -Name "srv01.corp.local" -Server "10.10.10.10"

# Check DNS server event log
Get-WinEvent -FilterHashtable @{
    LogName='DNS Server'
    Level=2,3  # Error and Warning
    StartTime=(Get-Date).AddHours(-24)
}

# Verify zone replication
Get-DnsServerZone -Name "corp.local" | Select-Object ZoneName, ReplicationScope, IsDsIntegrated

# Clear and refresh DNS cache
Clear-DnsServerCache -Force
ipconfig /flushdns
```

### Authentication Failures

```powershell
# Check Kerberos tickets on client
klist

# Purge and refresh Kerberos tickets
klist purge
gpupdate /force

# Verify time synchronization (critical for Kerberos)
w32tm /query /status
w32tm /resync /force

# Check for account lockouts
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut

# View failed logon attempts for specific user
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} | Where-Object {
    $_.Properties[5].Value -eq "jsmith"
} | Select-Object TimeCreated, @{N='SourceIP';E={$_.Properties[19].Value}}
```

## Knowledge Base Navigation

Each module folder contains:
- `Readme.md` - Module hub with learning objectives and lab overview
- Concept notes - Deep-dive explanations with tagged commands
- Lab guides - Step-by-step reproducible configurations
- Security sections - Hardening steps and attack/defense scenarios

**Best practices for using this knowledge base:**

1. **Follow the sequential learning path** - Later modules assume knowledge from earlier stages
2. **Build the lab progressively** - Take snapshots before major configuration changes
3. **Practice both administration and hardening** - Don't skip the security sections
4. **Document your lab** - Keep notes on IP addresses, credentials, and custom configurations
5. **Validate with attacks** - Use the purple-team modules to test your hardening
6. **Cross-reference modules** - Use Obsidian's graph view to explore related concepts

## Reference Configuration

Standard `corp.local` lab topology:

| Host | IP | Roles | OS |
|------|-----------|-------|------------|
| DC01 | 10.10.10.10 | AD DS, DNS, DHCP, GPO | Windows Server 2022 |
| SRV01 | 10.10.10.11 | File, DFS, IIS, FTP | Windows Server 2022 |
| WKS01 | DHCP | Domain client | Windows 11 Pro |
| Kali | 10.10.10.50 | Attack/validation | Kali Linux |

Network: `10.10.10.0/24` isolated internal network  
Domain: `corp.local` (forest functional level Windows Server 2016+)

## Additional Resources

- **Practical Labs** - Collection of scenario-based exercises (see `Practical-Labs/` folder)
- **Enterprise Projects** - Multi-service capstone projects (see `Enterprise-Projects/` folder)
- **Module-specific labs** - Each module folder contains hands-on guides
- **PowerShell scripts** - Automation examples throughout the knowledge base
- **Attack scenarios** - Purple-team validation in Module 19

For questions and updates, visit [armourinfosec.com](https://www.armourinfosec.com) or the GitHub repository.
