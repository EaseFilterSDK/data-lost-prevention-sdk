# EaseFilter SDK - Data Loss Prevention (DLP) & DRM

[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)]()
[![Language](https://img.shields.io/badge/Language-C%23%20%7C%20C++%20%7C%20Python%20%7C%20Rust%20%7C%20Java-success.svg)]()
[![Architecture](https://img.shields.io/badge/architecture-Kernel--Mode-brightgreen.svg)]()
[![License](https://img.shields.io/badge/license-Commercial-red.svg)]()

> A robust, kernel-level Data Loss Prevention (DLP) and Digital Rights Management (DRM) solution utilizing the EaseFilter Windows File System Minifilter Driver.

Traditional perimeter security is no longer enough to protect against decentralized workforces and advanced data exfiltration. The **EaseFilter DLP SDK** empowers organizations to implement true **Data-Centric Protection**. By tying security policies directly to the data itself through **Transparent File Encryption (TFE)** and persistent **DRM Tagging**, you can secure files at the kernel level without disrupting user workflows.

![DLP Architecture](https://www.easefilter.com/Images/dlp-architecture.jpg)
---

## 🚀 Core Capabilities

* **Kernel-Level Immutable Interception:** Operates via a Windows Minifilter Driver. Unlike user-mode hooks, kernel drivers cannot be bypassed or "unhooked" by high-privilege malware.
* **Process-Based Isolation (Zero-Trust):** Implement a Least Privilege model. Dynamically allow trusted apps (e.g., CAD or enterprise software) to read files while blocking potential leak endpoints (e.g., `chrome.exe`, `outlook.exe`).
* **Transparent File Encryption (TFE):** Files remain encrypted at rest using AES-256. The filter driver decrypts data on-the-fly *only* in-memory for authorized processes. 
* **USB & Device Control:** Prevent files from being moved to external hardware by dynamically toggling the `ALLOW_COPY_PROTECTED_FILES_TO_USB` flag.
* **Persistent DRM Metadata:** Embed encrypted DRM headers directly into the file structure. Access policies persist even when a file is moved to a USB drive, a cloud repository (OneDrive/S3), or sent via email.
* **Compliance Ready:** By capturing the "Who, What, When, and Where" of every I/O request (including Process PID and User SID), the SDK generates granular audit logs required for **HIPAA**, **GDPR**, and **CMMC** compliance.

## 🛡️ Leak Prevention Matrix

| Leak Vector | Prevention Technique | EaseFilter Feature |
| :--- | :--- | :--- |
| **USB Flash Drives** | Block write-out or device attachment. | `ALLOW_COPY_PROTECTED_FILES_TO_USB` |
| **Web Browser Upload** | Deny file-read permissions to browser processes. | `ProcessNameAccessRightList` |
| **Email Attachments** | Block email clients (`outlook.exe`) from protected paths. | `ProcessNameAccessRightList` |
| **Print-to-PDF** | Block print-spooler access to the original file. | Pre-I/O Callback Monitoring |
| **Physical Theft** | File remains encrypted; decryption key stays in-memory. | Transparent File Encryption (TFE) |

---

## 🏗️ Under the Hood: The Windows Minifilter

Implementing a robust security layer involves registering a **Pre-Operation Callback** within the Windows `FLT_REGISTRATION` structure. 

This enables the driver to inspect every `IRP_MJ_CREATE` and `IRP_MJ_READ` request before it ever reaches the physical storage stack. By sitting beneath user-mode applications, the EaseFilter driver gains universal visibility across all entry and exit points, including physical drives, SMB network shares, and local application buffers.


## 💻 Quick Start: Implementing a DRM Security Engine (C#)

This example demonstrates how to set up an isolation mini-filter, enforce AES-256 encryption, block global decryption, and explicitly grant decryption rights *only* to a trusted process using DRM tags.

```csharp
using System;
using EaseFilter.FilterControl;

public class DrmSecurityEngine 
{
    public void Initialize() 
    {
        // 1. Set up the isolation mini-filter for a secure directory
        FileFilter securityFilter = new FileFilter(@"C:\SecureVault\*");

        // 2. Enable military-grade AES-256 encryption at the driver level
        securityFilter.EnableEncryption = true;
        securityFilter.EncryptionKey = Utils.GetKeyByPassPhrase("Your-Enterprise-Key", 32);

        // 3. --- DECRYPTION ACCESS CONTROL ---
        // Disable decryption globally by default. Malware/untrusted apps will only read cipher-text.
        securityFilter.EnableReadEncryptedData = false;

        // 4. Explicitly grant decryption rights ONLY to your trusted processes
        // Note: Ensure the certificate name matches your signed application
        securityFilter.AddTrustedProcessRight(
            "SecureViewer.exe", 
            FilterAPI.ALLOW_MAX_ACCESS_RIGHT, 
            "EaseFilter Technologies", 
            null
        );

        // 5. Handle the DRM Tag request dynamically from the user-mode service
        securityFilter.EnableEncryptionKeyFromService = true;
        securityFilter.OnFilterRequestEncryptKey += (sender, e) => 
        {
            // Attach unique DRM metadata for remote policy tracking
            e.CustomTagData = "Global_Policy_ID_XYZ";
            
            // Retrieve per-file keys from your central Key Management Server (KMS)
            e.EncryptionKey = KeyServer.GetPerFileKey(e.FileName);
        };

        // 6. Apply the filter rule and start the engine
        string lastError = "";
        FilterControl.AddFilter(securityFilter);
        
        bool isStarted = FilterControl.StartFilter(
            FilterAPI.FilterType.FILE_SYSTEM_CONTROL, 
            Environment.ProcessorCount, 
            30, 
            "YOUR-LICENSE-KEY", 
            ref lastError
        );

        if (!isStarted)
        {
            Console.WriteLine($"Failed to start DLP engine: {lastError}");
        }
        else
        {
            Console.WriteLine("DLP Security Engine is actively protecting files.");
        }
    }
}
```

---
## EaseFilter File System Filter Driver SDK Reference
| Product Name | Description |
| --- | --- |
| [Cloud File System SDK](https://www.easefilter.com/cloud/cloud-file-system-sdk.htm) | EaseFilter Cloud File System SDK Introduction. |
| [CloudTier Storage Tiering SDK](https://www.easefilter.com/cloud/storage-tiering-sdk.htm) | EaseFilter Storage Tiering Filter Driver SDK Introduction. |
| [File Monitor SDK](https://www.easefilter.com/kb/file-monitor-filter-driver-sdk.htm) | EaseFilter File Monitor Filter Driver SDK Introduction. |
| [File Control SDK](https://www.easefilter.com/kb/file-control-file-security-sdk.htm) | EaseFilter File Control Filter Driver SDK Introduction. |
| [File Encryption SDK](https://www.easefilter.com/kb/transparent-file-encryption-filter-driver-sdk.htm) | EaseFilter Transparent File Encryption Filter Driver SDK Introduction. |
| [Registry Filter SDK](https://www.easefilter.com/kb/registry-filter-drive-sdk.htm) | EaseFilter Registry Filter Driver SDK Introduction. |
| [Process Filter SDK](https://www.easefilter.com/kb/process-filter-driver-sdk.htm) | EaseFilter Process Filter Driver SDK Introduction. |
| [EaseFilter SDK Programming](https://www.easefilter.com/kb/programming.htm) | EaseFilter Filter Driver SDK Programming. |

## EaseFilter SDK Sample Projects
| Sample Project | Description |
| --- | --- |
| [CloudTier Storage Tiering Demo](https://www.easefilter.com/cloud/cloudtier-storage-tiering-demo.htm) | A HSM File System Filter Driver Demo. |
| [CloudTier S3 Tiering Demo](https://www.easefilter.com/cloud/cloudtier-s3-intelligent-tiering-demo.htm) | CloudTier S3 Intelligent Tiering Demo. |
| [Cloud File DR S3 Demo](https://www.easefilter.com/cloud/cloud-file-dr-demo.htm) | Cloud File DR S3 Demo. |
| [Amazon S3 File Explorer Demo](https://www.easefilter.com/cloud/s3-browser-demo.htm) | Amazon S3 File Explorer Demo. |
| [Auto File DRM Encryption](https://www.easefilter.com/kb/auto-file-drm-encryption-tool.htm) | Auto file encryption with DRM data embedded. |
| [Transparent File Encrypt](https://www.easefilter.com/kb/AutoFileEncryption.htm) | Transparent on access file encryption. |
| [Secure File Sharing with DRM](https://www.easefilter.com/kb/DRM_Secure_File_Sharing.htm) | Secure encrypted file sharing with digital rights management. |
| [File Monitor Example](https://www.easefilter.com/kb/file-monitor-demo.htm) | Monitor file system I/O in real time, tracking file changes. |
| [File Protector Example](https://www.easefilter.com/kb/file-protector-demo.htm) | Prevent sensitive files from being accessed by unauthorized users or processes. |
| [FolderLocker Example](https://www.easefilter.com/kb/FolderLocker.htm) | Lock file automatically in a FolderLocker. |
| [Process Monitor](https://www.easefilter.com/kb/Process-Monitor.htm) | Monitor the process creation and termination, block unauthorized process running. |
| [Registry Monitor](https://www.easefilter.com/kb/RegMon.htm) | Monitor the Registry activities, block the modification of the Registry keys. |
| [Secure Sandbox Example](https://www.easefilter.com/kb/Secure-Sandbox.htm) |A secure sandbox example, block the processes accessing the files out of the box. |
| [FileSystemWatcher Example](https://www.easefilter.com/kb/FileSystemWatcher.htm) | File system watcher, logging the file I/O events. |
| [ZeroTrust Example](https://www.easefilter.com/kb/zero-trust-file-access-control-demo.htm) | Zero trust file access control with encryption feature. |

## Filter Driver Reference

* [Understand MiniFilter Driver](https://www.easefilter.com/kb/understand-minifilter.htm)
* [Understand File I/O](https://www.easefilter.com/kb/File_IO.htm)
* [Understand I/O Request Packets(IRPs)](https://www.easefilter.com/kb/understand-irps.htm)
* [Filter Driver Developer Guide](https://www.easefilter.com/kb/DeveloperGuide.htm)
* [MiniFilter Filter Driver Framework](https://www.easefilter.com/kb/minifilter-framework.htm)
* [Isolation Filter Driver](https://www.easefilter.com/kb/Isolation_Filter_Driver.htm)

## License

This repository contains two distinct licensing categories:

1. **Wrapper/Implementation Code (MIT License):** 
   The source code, scripts, and configuration files provided directly in this repository are licensed under the [MIT License](LICENSE). You are free to modify, distribute, and use this specific integration code.

2. **EaseFilter SDK (Commercial License):**
   This project requires the EaseFilter SDK (including its binaries, filter drivers, and core libraries) to function. The EaseFilter SDK is **proprietary commercial software** and is strictly excluded from the MIT License.   
   To use the EaseFilter SDK for commercial purposes or in a production environment, you must obtain a valid commercial license directly from [EaseFilter](https://www.easefilter.com/license.pdf).

## Support
If you have questions or need help, please contact support@easefilter.com 

[Home](https://www.easefilter.com/) | [Solution](https://www.easefilter.com/solutions.htm) | [Download](https://www.easefilter.com/download.htm) | [Demos](https://www.easefilter.com/online-fileio-test.aspx) | [Blog](https://blog.easefilter.com/) | [Programming](https://www.easefilter.com/kb/programming.htm)
