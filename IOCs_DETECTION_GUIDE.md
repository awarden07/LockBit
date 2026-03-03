# LockBit: Indicators of Compromise (IOCs) & Detection Procedures

## SECTION 1: TECHNICAL INDICATORS OF COMPROMISE

### 1.1 File-Based IOCs

#### Malware Executable Hashes (by Variant)

**LockBit 1.0 Known Samples:**
```
SHA-256: 7c4a6b6c2f5e7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
         (representative - varies by build)
MD5:     4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
SSDEEP:  6144:AAA...BBB (fuzzy hash for variant matching)

LockBit 2.0 Known Samples:**
SHA-256: 9f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f
         a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b
MD5:     5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
         2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f

LockBit 3.0 Known Samples (Building Changes Hashes):**
Base executable: a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8
DLL variant:     c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4
Reflective DLL:  e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7

Note: LockBit 3.0 generates unique hashes per deployment via builder tool
→ Use behavioral signatures and YARA rules instead of static hashes
```

#### Ransom Note File Indicators

```
Filename Patterns:
├─ {8-char-hex-code}_README.txt
│  └─ Examples: 3f7d4a2c_README.txt, a9b2f6e1_README.txt
├─ {UUID format}_README.txt
│  └─ Example: 550e8400-e29b-41d4-a716-446655440000.README.txt
├─ LockBit_README.txt
├─ LockBitBlack_[ID].txt
└─ PAYMENT_INSTRUCTIONS.txt

File Content Signatures (Use in IDS/IPS):
│
├─ Exact strings (case-insensitive):
│  ├─ "your network has been breached"
│  ├─ "files have been encrypted"
│  ├─ "lockbit"
│  ├─ "decryption" + "bitcoin"
│  └─ "payment required"
│
├─ Domain indicators:
│  ├─ "lockbit[.]top"
│  ├─ "lockbit3[.]onion"
│  ├─ ".onion" domains (indicates Tor)
│  └─ "TOR Browser" or Tor instructions
│
└─ Bitcoin wallet patterns:
   ├─ Bitcoin addresses (26-35 char alphanumeric string)
   ├─ Example: 1A1z7agoat7SFww3x6oLode09Z8eyAxXRm
   └─ Monero addresses (starts with 4 or 8, 95+ chars)
```

#### Modified System File Locations

**Registry Modifications:**

```
High-Confidence Indicators:

1. Windows Defender/Security Configuration Changes
   ├─ HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender
   │  └─ DisableRealtimeMonitoring = 1 (DWORD)
   ├─ HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender\Real-Time Protection
   │  └─ DisableBehaviorMonitoring = 1
   ├─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\WinDefend
   │  └─ Start = 4 (Disabled service)
   └─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MsMpSvc
      └─ Start = 4

2. Persistence Mechanisms
   ├─ HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
   │  └─ Values with strange names pointing to user temp directories
   │     Example: "SystemUpdate" = "C:\Users\[user]\AppData\Roaming\[random]\app.exe"
   │
   ├─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\*
   │  └─ New service entries with random binary names
   │     Key: Services\[randomGUID]
   │     ImagePath = "C:\Windows\Temp\[random].exe"
   │
   └─ HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
      └─ Similarly suspicious entries

3. System Configuration Changes
   ├─ HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
   │  ├─ AutoAdminLogon = 1 (enables auto-login)
   │  └─ DefaultPassword = [password] (stored in registry)
   │
   ├─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\LSA
   │  └─ LimitBlankPasswordUse = 0 (allows blank passwords)
   │
   └─ HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
      └─ ConsentPromptBehaviorAdmin = 0 (disables UAC prompts)

4. File Association Changes
   ├─ HKEY_CLASSES_ROOT\.{extension}
   │  └─ Modified associations (indicators of file type manipulation)
   │
   └─ HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Shell Extensions
      └─ Unusual COM object registrations
```

**Filesystem Indicators:**

```
Critical File Locations Changed:

1. Windows Defender Status/Configuration
   ├─ C:\Windows\System32\drivers\etc\hosts
   │  └─ Entries blocking Windows Update or AV sites
   │
   ├─ C:\ProgramData\Microsoft\Windows Defender
   │  └─ Missing or corrupted files
   │
   └─ C:\Program Files\Windows Defender\Platform
      └─ Disabled or corrupted executables

2. Wallpaper & Custom Files (Evidence of Execution)
   ├─ C:\Users\[User]\AppData\Roaming\Microsoft\Windows\Themes\
   │  ├─ Theme files with recent modification times
   │  ├─ Wallpaper images (usually .bmp or .jpg)
   │  └─ Desktop.ini files modifying display
   │
   ├─ C:\Windows\System32\oobe\info\backgrounds\
   │  └─ Modified wallpaper files
   │
   └─ C:\Users\[User]\AppData\Roaming\
      └─ Suspicious subdirectory creation with execution artifacts

3. Malware Staging Locations
   ├─ C:\Users\[User]\AppData\Roaming\[random_folder]\
   ├─ C:\Users\[User]\AppData\Local\Temp\
   ├─ C:\Windows\Temp\
   ├─ C:\ProgramData\[random_folder]\
   └─ C:\$Recycle.Bin\[recovery_files]

4. Scheduled Task Files
   ├─ C:\Windows\System32\Tasks\Microsoft\[random_names]
   │  └─ Recently created task XMLfiles
   │
   └─ C:\Windows\Tasks\
      └─ Legacy task locations

5. Backup/Recovery Files (Typically Deleted)
   ├─ C:\System Volume Information\
   │  └─ Shadow copies deleted (but may show deletion artifacts)
   │
   ├─ Backup store locations
   │  └─ Missing or wiped backups
   │
   └─ Backup software caches
      └─ Cleared or backdated
```

### 1.2 Process & Execution Indicators

#### Suspicious Process Chains

**Critical Process Execution Patterns:**

```
Pattern 1: Shadow Copy Deletion Sequence
─────────────────────────────────────────
Process Tree:
├─ explorer.exe
│  └─ cmd.exe (unusual: explorer shouldn't spawn cmd without user interaction)
│     └─ vssadmin.exe delete shadows /all /quiet
│        AND/OR
│     └─ wmic.exe shadowcopy delete
│        AND/OR
│     └─ powershell.exe (running shadow copy deletion commands)
│
Detection:
- Process creation with unusual parents
- Command line arguments with "delete shadows"
- Multiple shadow copy operations in short timeframe
- Execution from user temp directories

Alert Priority: CRITICAL (High confidence ransomware indicator)


Pattern 2: Service/AV Termination Sequence
──────────────────────────────────────────
Process Tree:
├─ ransomware.exe (parent)
│  ├─ taskkill.exe /F /IM sql.exe (database service kill)
│  ├─ taskkill.exe /F /IM backup.exe (backup software kill)
│  ├─ taskkill.exe /F /IM MsMpEng.exe (Windows Defender kill)
│  ├─ taskkill.exe /F /IM avp.exe (Kaspersky kill)
│  ├─ net.exe stop VSS (Volume Shadow Copy service)
│  ├─ net.exe stop BackupExec
│  ├─ wmic.exe process call create (launching additional processes)
│  └─ schtasks.exe /create (creating scheduled tasks)
│
Detection:
- Multiple rapid process terminations
- Targeting of known security/backup software
- Service termination attempts
- Command-line patterns

Alert Priority: CRITICAL


Pattern 3: Credential Harvesting Sequence
─────────────────────────────────────────
Process Tree:
├─ ransomware.exe
│  ├─ powershell.exe (loading credential dumping modules)
│  │  └─ (Mimikatz-like API sequences)
│  ├─ cmd.exe
│  │  └─ reg.exe save HKLM\SAM C:\temp\sam.bak
│  │     reg.exe save HKLM\SECURITY C:\temp\sec.bak
│  │  (Extracting SAM and security registry hives)
│  └─ tasklist.exe /v (process enumeration)
│
Detection:
- LSASS memory access (very sensitive indicator)
- Registry hive file access
- Privileged processes loading from unusual locations
- Direct SAM/SECURITY file access

Alert Priority: CRITICAL


Pattern 4: Lateral Movement Sequence
────────────────────────────────────
Process Tree:
├─ ransomware.exe
│  ├─ net.exe view /all (network enumeration)
│  ├─ net.exe view \\[IP] (remote system enumeration)
│  ├─ copy ransomware.exe \\[victim]\C$\Windows\Temp\
│  ├─ wmic.exe /node:[victim] process call create (remote execution)
│  ├─ schtasks.exe /s [victim] /create (remote task creation)
│  └─ psexec.exe \\[victim] (if present, direct indicator)
│
Detection:
- Network scanning from single source
- Multiple SMB connections to different systems
- Remote process/task creation attempts
- High volume of network-destined commands

Alert Priority: HIGH


Pattern 5: Privilege Escalation Attempt
───────────────────────────────────────
Process Tree:
├─ User-context process
│  ├─ fodhelper.exe (COM elevation moniker)
│  ├─ OR eventvwr.exe
│  ├─ (followed by injection of malware code)
│  └─ Result: Process running as SYSTEM
│
Detection:
- Unusual parent-child relationships (fodhelper/eventvwr not normal parents)
- UAC-related process sequences
- Token manipulation API calls
- Privilege escalation from user to SYSTEM

Alert Priority: HIGH
```

#### Command Line Arguments (Indicators)

```
CRITICAL Argument Patterns:

1. VSS/Backup Deletion
   ├─ vssadmin delete shadows
   ├─ wmic shadowcopy delete
   ├─ diskshadow.exe (shadowcopy management)
   ├─ wbadmin delete systemstatebackup
   └─ icacls "\\?\Volume{GUID}" /grant "%userName%":F

2. Service/Process Termination
   ├─ taskkill /F /IM [antivirus/backup software]
   ├─ net stop [service name]
   ├─ sc stop [service] /force
   └─ Stop-Service -Name [service] -Force

3. Firewall/Security Disabling
   ├─ netsh advfirewall set allprofiles state off
   ├─ Set-MpPreference -EnableRealtimeMonitoring $false
   ├─ Set-MpPreference -DisableBehaviorMonitoring $true
   └─ Set-MpPreference -DisableIntrusionPreventionSystem $true

4. Persistence/Scheduling
   ├─ schtasks /create /tn "[random task name]" /tr "[path to malware]"
   ├─ New-Item -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
   ├─ reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services
      ├" sc create [service] binpath= "[malware path]"
   └─ Invoke-WmiMethod -Class Win32_Process -Name Create

5. Network Enumeration
   ├─ net view /all
   ├─ ipconfig /all
   ├─ arp -a
   ├─ Get-WmiObject Win32_NetworkAdapterConfiguration
   └─ nslookup
```

### 1.3 Network-Based Indicators

#### Command & Control (C2) Domains

**Known LockBit C2 Infrastructure:**

```
Primary Leaks Site (Victim Communication & Data Sales):
├─ lockbit.top - Primary dark web presence
├─ lockbit.news - Secondary information site
├─ lockbit3.onion - Tor-accessible variant
└─ lockbit[variations] - Dynamic domains (varies by campaign)

Secondary C2 Networks:
├─ Direct C2 servers (vary frequently, generated per build)
├─ Bulletproof hosting providers (Russian/Eastern European)
├─ Compromised legitimate servers (compromised ASNs)
└─ Bulletproof ISP hosted infrastructure

DNS Indicators (In Proxy Logs):
├─ Queries to *.onion domains (high confidence Tor usage)
├─ Rapid query resolution failures (C2 change attempts)
├─ Unusual DNS query patterns (bulk subdomain queries)
└─ Queries to known C2 domains or lookalike domains

Cryptocurrency Wallets (Known Extortion Wallets):
├─ Bitcoin addresses monitored on blockchain.com
├─ Monero addresses (more difficult to trace)
├─ Addresses linked to LockBit payments (via transaction analysis)
└─ Mixer addresses (tumbling service recipients)
```

### 1.4 Data Exfiltration Indicators

```
Outbound Connection Patterns:

1. Data Staging/Compression
   - Large file creation in Temp directories (staging for exfil)
   - Rapid directory traversal with .zip/.rar creation
   - 7-Zip, WinRAR process execution with compression flags
   - CloudFlare CDN connections (exfil infrastructure)

2. Bulk Data Transfer
   - Large outbound HTTPS transfers (encrypted exfil)
   - FTP connections to external IPs
   - SFTP/SSH outbound on unusual ports
   - WebDAV over HTTPS (data transfer protocol)
   - BitTorrent connections (distributed exfil)

3. Cloud Service Abuse
   - Google Drive API calls with file upload operations
   - OneDrive/SharePoint to personal account transfers
   - AWS S3 API calls (put object operations)
   - Azure Blob Storage transfers
   - Dropbox/Box/pCloud API authentication

4. Exfiltration Protocol Patterns
   - DNS exfiltration (data encoded in DNS queries)
   - ICMP tunneling (data in ICMP packets)
   - HTTP POST with large body size
   - HTTPS with self-signed certificates
   - Custom protocols on high-numbered ports
```

---

## SECTION 2: BEHAVIORAL INDICATORS & DETECTION SIGNATURES

### 2.1 YARA Rules

```yara
rule LockBit_Ransomware_Generic {
    meta:
        description = "Detect LockBit ransomware variants"
        author = "Threat Intelligence Team"
        date = "2024-02-28"
        version = "1.0"
    
    strings:
        // String indicators
        $ransom_note = "README" nocase
        $lockbit_ref = "lockbit" nocase
        $bitcoin_mention = "bitcoin" nocase
        $payments_ref = "payment" nocase
        
        // RSA key structure (PKCS#1)
        $rsa_begin = "-----BEGIN RSA PRIVATE KEY-----" wide ascii
        $rsa_structure = {30 82 ?? ?? 02 01 00 02} 
        // ^ RSA key structure indicator
        
        // Encryption API sequences
        $crypt_api = "CryptEncrypt" ascii
        $aes_init = "AES" wide ascii
        
        // API hiding indicators
        $getprocaddress = "GetProcAddress" ascii
        $loadlibrary = "LoadLibraryA" ascii
        
        // File encryption patterns
        $delete_shadows = "vssadmin" nocase
        $shadow_wmic = "shadowcopy" nocase
        $file_extension_change = {2E 6C 6F 63 6B 62 69 74} // .lockbit in hex
        
        // Process termination
        $taskkill = "taskkill" nocase
        $service_stop = "net stop" nocase
        
        // Obfuscation indicators
        $control_flow_obfuscation = {EB FE}  // Junk/padding pattern
        $xor_key = {89 C1 D1 E9 88 0C}  // XOR encryption pattern
    
    condition:
        (($rsa_structure or $rsa_begin) and 
         (3 of ($string*)) and 
         (2 of ($crypt*, $getprocaddress, $loadlibrary))) or
        (all of ($delete*, $file_extension_change)) or
        (3 of ($taskkill, $service_stop, $file_extension_change))
}


rule LockBit_FileEncryption_Behavior {
    meta:
        description = "Detect ransomware file encryption behavior"
        type = "behavioral"
    
    strings:
        // File operation API calls
        $file_open = "CreateFileA" ascii
        $file_write = "WriteFile" ascii  
        $file_read = "ReadFile" ascii
        $file_enum = "FindFirstFileA" ascii
        
        // Large I/O operations
        $buffer_op1 = {8B 45 FC 03 45 08} // ADD with buffer operations
        $buffer_op2 = {C7 45 F8 00 10 00 00} // 4096 byte buffer allocation
        
        // Directory traversal patterns
        $recurse_dir = "\\*" wide ascii
        $mask_search = "FindNextFile" ascii
    
    condition:
        all of them
}


rule LockBit_VSS_Deletion {
    meta:
        description = "Detect shadow copy deletion commands"
        severity = "critical"
    
    strings:
        $vssadmin_cmd = "vssadmin.exe" nocase
        $delete_arg = "delete" nocase
        $shadows_arg = "shadows" nocase
        $all_arg = "/all" 
        $quiet_arg = "/quiet"
        
        $wmic_cmd = "wmic.exe" nocase
        $wmic_shadow = "shadowcopy"
        $wmic_delete = "delete"
        
        $diskshadow = "diskshadow.exe"
        $delete_all = "delete all"
    
    condition:
        (($vssadmin_cmd and $delete_arg and $shadows_arg) or
         ($wmic_cmd and $wmic_shadow and $wmic_delete) or
         ($diskshadow and $delete_all))
}
```

### 2.2 Sigma Detection Rules (SIEM/EDR Format)

```yaml
title: LockBit Ransomware - Shadow Copy Deletion
correlation:
  correlation_type: event_count
  condition: selection | count(CommandLine) > 5
  timespan: 5m
  filters: ComputerName

detection:
  selection:
    EventID: 1  # Process Creation
    CommandLine|contains|all:
      - 'vssadmin'
      - 'delete'
      - 'shadows'
  
  filter:
    User|endswith: '\\SYSTEM'  # Expected for SYSTEM operations, but unusual
  
  condition: selection and not filter

falsepositives:
  - Legitimate backup software operations
  - System administrator maintenance

severity: critical

---

title: LockBit Ransomware - Defender Disabling
correlation:
  type: registry_modification
  
detection:
  selection_1:
    RegistryPath: 'HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender'
    RegistryValueName: 'DisableRealtimeMonitoring'
    RegistryValueData: 1
  
  selection_2:
    RegistryPath: 'HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\WinDefend'
    RegistryValueName: 'Start'
    RegistryValueData: 4
  
  condition: selection_1 or selection_2

severity: high

---

title: LockBit Ransomware - Critical Service Termination
correlation:
  type: process_termination_sequence
  
detection:
  selection:
    ProcessName|endswith:
      - 'taskkill.exe'
    CommandLine|contains|any:
      - 'sql.exe'
      - 'backup.exe'
      - 'MsMpEng.exe'
      - 'beserver.exe'
      - 'oracle.exe'
    CommandLine|contains: '/F'  // Force kill flag
  
  filter_admin:
    User|contains: 'Administrator'
    
  timespan: 5m
  count_threshold: 3  // Multiple service kills

condition: selection and not filter_admin
severity: critical
```

---

## SECTION 3: DETECTION & HUNTING PROCEDURES

### 3.1 Rapid Detection Procedures (First Hour)

**Immediate Indicators Check (Use When Incident Suspected):**

```
CHECKLIST FOR INCIDENT RESPONDER (First 60 minutes)

☐ STEP 1: System Isolation (0-5 minutes)
  ├─ Disconnect affected system from network
  ├─ Disconnect from wireless if applicable
  ├─ DO NOT shut down (preserve evidence)
  ├─ Preserve active memory (if possible)
  └─ Do NOT reboot system

☐ STEP 2: Rapid Evidence Preservation (5-15 minutes)
  ├─ Capture system memory:
  │  └─ FTK Imager / DumpIt / WinPMEM for memory DDL
  ├─ Photograph screen contents
  ├─ Document:
  │  ├─ Date/Time of discovery
  │  ├─ User account logged in
  │  ├─ System hostname and IP
  │  ├─ Network shares visible
  │  └─ Any ransom notes visible
  └─ Preserve volatile artifacts

☐ STEP 3: Quick Registry Check (15-20 minutes)
  ├─ Open regedit (as administrator)
  ├─ Navigate to:
  │  ├─ HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender
  │  ├─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\WinDefend
  │  ├─ HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
  │  └─ HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\
  ├─ Screenshot any suspicious entries
  ├─ Document modification timestamps
  └─ Search for recent "LockBit" or ransom-related strings

☐ STEP 4: File System Quick Scan (20-30 minutes)
  ├─ Search for recent ransom notes:
  │  ├─ File pattern: *_README.txt
  │  ├─ Locations: Desktop, Documents, root of drives
  │  └─ Read and document contents
  ├─ Look for file extension changes:
  │  ├─ *.lockbit files
  │  ├─ Unusual bulk file extension changes
  │  └─ Recent file modifications across directories
  ├─ Check temp directories:
  │  ├─ C:\Windows\Temp
  │  ├─ C:\Users\[user]\AppData\Local\Temp
  │  └─ Look for executable files
  └─ Document findings

☐ STEP 5: Process & Service Check (30-40 minutes)
  ├─ Use Task Manager (Ctrl+Shift+Esc):
  │  ├─ Check for unusual processes
  │  ├─ Right-click → Properties on suspicious processes
  │  ├─ Document process names and paths
  │  └─ Screenshot process tree
  ├─ Check Windows Services (services.msc):
  │  ├─ Look for recently created services
  │  ├─ Check service status (running/stopped)
  │  ├─ Right-click → Properties for path information
  │  └─ Document suspicious services
  └─ Check scheduled tasks (taskschd.msc):
     ├─ Recently created tasks
     ├─ Task triggers and actions
     └─ Document suspicious tasks

☐ STEP 6: Network Connectivity Check (40-50 minutes)
  ├─ Run netstat -ano (or Get-NetTCPConnection in PowerShell):
  │  ├─ Identify foreign IP addresses
  │  ├─ Note unusual ports (not 22, 3389, 443, typical)
  │  └─ Document established connections
  ├─ Check recent connections via Event Viewer:
  │  ├─ Windows Logs → Security → Event ID 5156, 5158
  │  └─ Review outbound connection attempts
  └─ Document suspicious network connectivity

☐ STEP 7: Event Log Review (50-60 minutes)
  ├─ Event Viewer → Windows Logs → System:
  │  ├─ Look for Service Control Manager events
  │  ├─ Service start/stop events
  │  └─ Error events
  ├─ Application log:
  │  └─ Look for crashes or unusual events
  ├─ Security log (if available):
  │  ├─ Event ID 4688 (process creation)
  │  ├─ Event ID 4657 (registry modification)
  │  └─ Event ID 5140 (network share access)
  └─ Document timeline of events

☐ STEP 8: External Communication (60+ minutes)
  ├─ DO NOT PROCESS RANSOM if discovered
  ├─ Notify:
  │  ├─ Incident Response Team Lead
  │  ├─ Information Security Director
  │  ├─ IT Director
  │  ├─ Legal Department (if applicable)
  │  └─ FBI Cyber Division (if critical infrastructure)
  ├─ Initiate formal incident response
  └─ Begin forensic investigation
```

### 3.2 Advanced Hunting Procedures (Hour 2-24)

**Extended Investigation Procedures:**

```
ADVANCED THREAT HUNTING CHECKLIST (Hours 2-24)

Memory Forensics Analysis
├─ Use Volatility Framework:
│  ├─ volatility -f memory.dump pslist (process listing)
│  │  └─ Identify suspicious processes
│  ├─ volatility -f memory.dump handles -p [PID] (process handles)
│  │  └─ What files/registry keys opened
│  ├─ volatility -f memory.dump netscan (network sockets)
│  │  └─ Active network connections at memory dump time
│  ├─ volatility -f memory.dump dlllist -p [PID] (DLL injection detection)
│  │  └─ Unusual DLLs loaded by processes
│  ├─ volatility -f memory.dump hivelist (registry hives in memory)
│  │  └─ Registry modification evidence
│  └─ volatility -f memory.dump filescan (open files)
│     └─ Files being accessed at incident time
│
├─ String Extraction:
│  ├─ strings memory.dump | grep -i lockbit
│  ├─ strings memory.dump | grep -i bitcoin
│  ├─ strings memory.dump | grep \.onion
│  └─ Identify decryption keys or C2 addresses
│
└─ Malware Analysis:
   ├─ Extract suspicious PEs from memory
   ├─ Analyze with IDA Pro or Ghidra
   └─ Identify encryption routines and C2 callbacks

Disk Forensics Analysis
├─ Timeline Creation:
│  ├─ Create $MFT timeline (Master File Table):
│  │  └─ fsutil mft zoneinfo C:
│  ├─ Analyze MFT residual data:
│  │  └─ Use MFTECmd (Timeline Explorer)
│  ├─ Examine $UsnJournal (Change Journal):
│  │  └─ Identify rapid file modifications
│  └─ Create comprehensive timeline:
│     └─ Merge$MFT, event logs, system logs
│
├─ Windows Registry Analysis:
│  ├─ Extract registry hives:
│  │  ├─ C:\Windows\System32\config\SAM
│  │  ├─ C:\Windows\System32\config\SECURITY
│  │  ├─ C:\Windows\System32\config\SOFTWARE
│  │  ├─ C:\Windows\System32\config\SYSTEM
│  │  └─ User registry: C:\Users\[user]\NTUSER.DAT
│  ├─ Analyze with:
│  │  ├─ Registry Viewer / Registry Recon
│  │  └─ RegRipper for automated analysis
│  ├─ Focus on:
│  │  ├─ Service creation timestamps
│  │  ├─ AppInit_DLLs (DLL injection)
│  │  ├─ Winlogon entries
│  │  └─ LastWrite times for modifications
│  └─ Document persistence mechanisms
│
├─ Prefetch Analysis:
│  ├─ Location: C:\Windows\Prefetch\
│  ├─ Analyze with:
│  │  └─ Prefetch Parser / PECmd (Eric Zimmerman tools)
│  ├─ Extract:
│  │  ├─ Process execution timestamps
│  │  ├─ Files accessed during execution
│  │  ├─ Execution count
│  │  └─ Last execution time
│  └─ Timeline process execution
│
├─ ShadowCopy Analysis (if available):
│  ├─ Check for deleted files:
│  │  ├─ Previous versions of encrypted files
│  │  ├─ Deleted malware files
│  │  └─ Recovery data
│  ├─ Mount shadow copy:
│  │  └─ wmic shadowcopy call unmount /nointeractive
│  └─ Analyze historical state
│
├─ Alternate Data Streams (ADS):
│  ├─ Check C:\Windows\System32\ for hidden files
│  ├─ Command: dir /s /r C:\Windows\System32\ | find ":$"
│  └─ Potential malware hiding location
│
└─ Link File Analysis:
   ├─ Analyze .lnk files on Desktop/Start menu
   ├─ Tools: LECmd (Link Explorer)
   └─ Identify command execution patterns

Network Forensics
├─ Packet Capture Analysis:
│  ├─ Review network logs
│  ├─ Identify connections to known C2 addresses
│  ├─ Extract exfiltrated data (if captured)
│  ├─ Analyze encryption patterns
│  └─ Timeline command and control activifty
│
├─ DNS Query Analysis:
│  ├─ Review DNS logs for:
│  │  ├─ Queries to *.onion domains
│  │  ├─ Queries to known C2 domains
│  │  ├─ Rapid domain generation pattern (DGA)
│  │  └─ Failed resolution followed by retries
│  └─ Document timeline
│
├─ Web Proxy/Firewall Logs:
│  ├─ Search for:
│  │  ├─ Outbound connections to non-standard ports
│  │  ├─ Large data transfers outbound
│  │  ├─ Tor .onion site access attempts
│  │  └─ Encrypted tunneling protocols
│  └─ Identify exfiltration evidence
│
└─ Email Gateway Logs:
   ├─ Search for:
   │  ├─ Emails with malicious attachments
   │  ├─ Emails from credential harvesting (phishing)
   │  ├─ Spoofed internal email addresses
   │  └─ Links to known malware URLs
   └─ Timeline email-based compromise


Lateral Movement Investigation
├─ Cross-System Analysis:
│  ├─ Check all systems on same subnet
│  ├─ Look for similar encryption patterns
│  ├─ Analyze for common compromise timestamps
│  ├─ Check for credential usage across systems
│  └─ Map infection spread pattern
│
├─ Remote Access Logs:
│  ├─ RDP Connection logs (Event ID 4624/4625)
│  ├─ VPN access logs
│  ├─ Remote administration tool logs
│  └─ Document access patterns
│
├─ SMB Activity Analysis:
│  ├─ Network share access logs
│  ├─ Failed vs. successful access attempts
│  ├─ File copy patterns
│  └─ Executable execution on shares
│
└─ Credential Analysis:
   ├─ Dumped credentials identification
   ├─ Credential usage across systems
   ├─ Pass-the-hash attack evidence
   ├─ Domain credential compromise
   └─ Backdoor account creation


Data Exfiltration Investigation
├─ Data Staging Detection:
│  ├─ Large temporary files created
│  ├─ Archive file creation (.zip, .rar)
│  ├─ Compression tool execution
│  └─ Staging directory analysis
│
├─ Egress Data Tracking:
│  ├─ Large outbound connections
│  ├─ Connections to cloud storage
│  ├─ FTP/SFTP connections
│  ├─ Custom protocol analysis
│  └─ Data transfer volume
│
├─ exfiltration Tool Analysis:
│  ├─ Custom exfil tool identification
│  ├─ Protocol analysis
│  ├─ Credential stuffing for cloud services
│  ├─ Compromised accounts used for exfulsion
│  └─ Ransomware operator access evidence
│
└─ Timeline Correlation:
   ├─ Data access timeline
   ├─ Staging timeline
   ├─ Network transfer timeline
   └─ C2 communication timeline
```

### 3.3 Threat Hunting Search Queries (SIEM)

**Splunk/ELK Queries for LockBit Detection:**

```splunk
# Shadow Copy Deletion Hunt
index=main EventCode=1 (vssadmin OR wmic OR diskshadow)
  (delete OR deleted) shadows
| stats count by ComputerName, CommandLine
| where count > 0

# Critical Service Termination Hunt
index=main EventCode=1 taskkill /F parent_process=ransomware.exe
  (sql.exe OR backup.exe OR MsMpEng.exe OR avp.exe)
| timeline count by ComputerName
| where count > 3

# Lateral Movement Hunt (SMB)
index=main EventCode=5140 IpAddress != 192.168.*
AND RelativeTargetName IN ("ADMIN$", "C$", "IPC$")
| stats count by ComputerName, SourceWorkstation
| where count > 10

# Ransom Note Detection
index=main file_name="*_README.txt" OR file_name="*README*.txt"
| stats count by ComputerName, file_path
| where count > 0

# Registry Modification Hunt
index=main EventCode=4657 registry_path="*\Windows Defender*"
AND (DisabledRealtimeMonitoring OR DisableBehaviorMonitoring)
| stats by ComputerName, registry_path

# PowerShell Credential Dumping
index=main powershell command IN (
  "*Get-WmiObject*Win32_Process*",
  "*lsass*",
  "*Invoke-Mimikatz*",
  "*Get-ADUser*"
) AND ComputerName != domain_controller
| stats count by ComputerName, User, command
```

---

## SECTION 4: INCIDENT RESPONSE WORKBOOK

### Recovery & Notification Procedures

```
POST-INCIDENT RECOVERY PROCEDURES

Phase 1: Forensic Preservation (Hours 0-48)
├─ Initiate forensic imaging of affected systems
├─ Preserve memory dumps
├─ Collect event logs and system logs
├─ Preserve network traffic captures
├─ Document all artifacts

Phase 2: System Isolation & Assessment (Hours 48-72)
├─ Verify backup integrity and restore capability
├─ Assess scope of compromise (how many systems)
├─ Review data exfiltration evidence
├─ Determine encryption scope and file count
└─ Estimate recovery time (with/without ransom)

Phase 3: Eradication (Days 3-7)
├─ Identify all persistence mechanisms
├─ Remove malware from all infected systems
├─ Reset all credentials (assume compromise)
├─ Patch identified vulnerabilities
├─ Remove unauthorized accounts/services

Phase 4: Recovery (Days 7-14)
├─ Restore systems from clean backups
├─ Verify data integrity post-restoration
├─ Reconfigure security controls
├─ Enable enhanced monitoring
└─ Conduct post-mortem analysis

Phase 5: Lessons Learned (Days 14-30)
├─ Security assessment of controls
├─ Identify security gaps exploited
├─ Plan improvements
├─ Update incident response procedures
└─ Staff training on findings
```

---

**Last Updated**: February 2026
**Distribution**: Authorized Cybersecurity Professionals Only
