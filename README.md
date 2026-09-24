# Configuring-Storage-Account-Blobs-and-Azure-Files-.
Hands-on Azure lab creating a storage account, uploading a blob to a private container, provisioning an Azure Files SMB share, and mounting it on a Windows Server VM via PowerShell using the storage account key.

# Azure Storage Account Lab (Blob Containers, Azure Files, and SMB Mounting)

`Microsoft Azure` `Azure Storage` `Blob Storage` `Azure Files` `PowerShell` `Windows Server` `SMB`

## Overview
This lab was hands on practice standing up an Azure Storage account and working with two of its core services, Blob Storage and Azure Files. I created a storage account, uploaded a blob into a private container, provisioned an SMB file share, and then connected that file share to a Windows Server VM using the storage account key and a PowerShell script generated directly from the Azure Portal.

The goal wasn't just to click through the portal wizard once. It was to understand what each configuration choice actually does, why the defaults are set the way they are, and what it looks like when a file share gets mounted as a real network drive on a server rather than just sitting in Azure.

## Objective
Get comfortable creating and configuring a Storage account in Azure, uploading and managing data in Blob Storage, provisioning an Azure Files SMB share, and connecting to that share from a Windows Server host using credentials and a mapped drive.

## Environment
- **Storage account:** `storage1files24`, deployed to the `1storageaccount` resource group under Azure subscription 1
- **Region:** West US, with geo redundant storage (GRS) providing a secondary copy in East US
- **Blob container:** `company-container`, private access level
- **File share:** `companydrive24`, SMB protocol, 100 TiB max storage quota, transaction optimized access tier
- **Target VM:** Windows Server, connected to the share over port 445 using PowerShell
- **Access method:** Storage account key based authentication (not Microsoft Entra)

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|---------------|----------------|
| **Azure Portal** | Web based management console | Created the storage account, containers, and file share, and configured access settings |
| **Azure Blob Storage** | Object storage service | Held an uploaded file (`36.avif`) inside a private container to test upload and container management |
| **Azure Files** | Managed SMB/NFS file share service | Provisioned a network file share that can be mounted like a traditional file server |
| **PowerShell** | Scripting and remote administration | Ran the Azure generated connection script to test port 445 reachability, cache credentials, and mount the file share as a drive |
| **Test-NetConnection** | Network connectivity test | Verified the storage account was reachable over port 445 before attempting to mount the share |
| **cmdkey** | Windows credential manager utility | Saved the storage account key as a cached credential so the mapped drive would persist across reboots |

## What I Did

### Creating the Storage Account
1. Deployed a new storage account, `storage1files24`, into the `1storageaccount` resource group in the West US region.
2. Confirmed the deployment completed successfully from the deployment overview blade, which also gave me the correlation ID and deployment timestamp for reference.
3. Reviewed the account's default configuration on the Properties page, including blob soft delete (7 days), container soft delete (7 days), hierarchical namespace (disabled), minimum TLS version (1.2), and public network access (enabled from all networks). This was a useful way to see what Azure turns on by default versus what needs to be explicitly configured for a hardened environment.

### Working with Blob Storage
1. Created a container named `company-container` and set its anonymous access level to private, alongside the account's other default container, `$logs`.
2. Uploaded a file (`36.avif`) into the container and confirmed the upload with the portal's success notification.
3. Checked the blob's properties in the container view, including last modified date, access tier (Hot, inferred), blob type (block blob), size (30.21 KiB), and lease state (available).

### Provisioning the Azure Files Share
1. Created a new SMB file share named `companydrive24` inside the same storage account.
2. Reviewed the share's properties, including maximum storage size (102400 GiB), current used capacity (0 GiB at creation), access tier (transaction optimized), redundancy (GRS), and soft delete (7 days).
3. Noted that identity based access was not configured, meaning the share was set up for key based authentication rather than Microsoft Entra integration.

### Connecting the File Share to a Windows Server
1. Opened the Connect blade for the file share and selected storage account key as the authentication method, rather than Active Directory or Microsoft Entra, since this environment wasn't domain joined for file access.
2. Copied the auto generated PowerShell script, which does three things: tests connectivity to the storage account over port 445 using `Test-NetConnection`, saves the storage account key as a cached credential with `cmdkey`, and mounts the share as a persistent PSDrive if the connection test succeeds.
3. Ran the script from an elevated PowerShell prompt on the Windows Server VM. The connection test succeeded, the credential was added successfully, and the share was mounted as drive `Z`, showing 102400.00 GB free and 0.00 GB used, matching what the portal reported.
4. Confirmed the mapped drive appeared correctly in the PowerShell session output under `Name`, `Provider`, and `Root`, pointing to `\\storage1files24.file.core.windows.net\companydrive24`.

## What's in This Repo

```
azure-storage-lab/
├── README.md                        # This file
└── screenshots/
    ├── 01-blob-container-upload.png
    ├── 02-storage-account-properties.png
    ├── 03-containers-list.png
    ├── 04-deployment-complete.png
    ├── 05-file-share-properties.png
    ├── 06-connect-blade-script.png
    └── 07-powershell-mount-confirmation.png
```

## Skills I Picked Up
- **Understanding the difference between Blob Storage and Azure Files,** and when you'd reach for object storage versus a traditional network file share that legacy applications can map as a drive letter.
- **Reading storage account defaults critically,** noticing settings like public network access being enabled from all networks by default, which is something I'd want to lock down in a production environment rather than leave untouched.
- **Working with storage account key authentication,** including what the Connect blade's generated script is actually doing line by line rather than just running it blindly.
- **Verifying connectivity before troubleshooting,** using `Test-NetConnection` against port 445 to confirm the share was reachable before assuming a mount failure was a credentials problem.
- **Persisting credentials safely with cmdkey,** so a mapped network drive survives a server reboot instead of needing to be reconnected manually every time.

## How This Applies in the Real World
Azure Files with SMB is commonly used as a drop in replacement for an on premises file server, especially for organizations migrating legacy applications that expect a mapped drive letter rather than an API based storage call. Understanding both the portal side configuration and what the underlying PowerShell is actually doing matters because in a real environment you're often the one troubleshooting a broken mount on a server that can't reach the storage endpoint, and knowing to check port 445 reachability first, before touching credentials, saves a lot of wasted time.

The choice between storage account key auth and Microsoft Entra based identity access is also a real world decision point. Key based auth is simpler to stand up, which is why it shows up in a lot of quick deployments and labs, but it also means anyone with that key has full access to the share. In a production setting, moving to identity based access control would be the next logical hardening step.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in **healthcare**. It's a different field on paper, but a lot of the muscle memory carries over: following procedures carefully, protecting sensitive information, and staying methodical when a configuration doesn't behave the way the documentation says it should. I'm currently studying for **CompTIA Security+** and building labs like this one to get real hands on reps with cloud infrastructure, since that's the part of my background I'm actively filling in.

## What I Want to Learn Next
- Reconfiguring this share to use Microsoft Entra identity based access instead of the storage account key, and comparing the setup process
- Locking down public network access and testing the share through a private endpoint instead
- Enabling diagnostic logging on the storage account and reviewing access patterns in Azure Monitor
- Practicing blob lifecycle management policies to automatically move older blobs to cooler access tiers

## Limitations & What I'd Do Differently in Production
- **Public network access was left enabled from all networks.** In production I would restrict this to specific virtual networks or use a private endpoint instead.
- **Storage account key authentication grants full access to anyone holding the key.** A production deployment would move toward Microsoft Entra based identity access for better auditability and least privilege control.
- **No diagnostic logging or monitoring alerts were configured** on the storage account during this lab. A real deployment would want activity logging enabled from the start.
- **This was a single container and single share setup.** A production environment would likely involve access policies, network rules, and lifecycle management layered on top of what's shown here.

## References
- [Azure Blob Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Azure Files Documentation](https://learn.microsoft.com/en-us/azure/storage/files/)
- [Azure Storage Security Guide](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
