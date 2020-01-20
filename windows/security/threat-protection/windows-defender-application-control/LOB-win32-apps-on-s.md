---
title: Allow LOB Win32 Apps on Intune-Managed S Mode Devices (Windows 10)
description: Using WDAC supplemental policies, you can expand the S mode base policy on your Intune-managed devices.
ms.prod: w10
ms.mktglfcycl: deploy
ms.sitesec: library
ms.pagetype: security
ms.localizationpriority: medium
author: isbrahm
ms.date: 09/18/2019
---

# Allow Line-of-Business Win32 Apps on Intune-Managed S Mode Devices

**Applies to:**

-   Windows 10

Beginning in Windows 10 (build 18363), Microsoft Intune enables customers to deploy and run business critical Win32 applications as well as Windows components that are normally blocked in S mode (ex. PowerShell.exe) on their Intune-managed Windows 10 in S mode (S mode) devices. 

With Intune, IT Pros can now configure their managed S mode devices using a Windows Defender Application Control (WDAC) supplemental policy that expands the S mode base policy to authorize the apps their business uses. This feature changes the S mode security posture from “every app is Microsoft-verified" to “every app is verified by Microsoft or your organization”. 

# Policy Authorization Process
![Policy Authorization](images/wdac-intune-policy-authorization.png)
The general steps for expanding the S mode base policy on your devices are to generate a supplemental policy, sign that policy, and then upload the signed policy to Intune and assign it to user or device groups.
1. Generate a supplemental policy with WDAC tooling

    This policy will expand the S mode base policy to authorize additional applications. Anything authorized by either the S mode base policy or your supplemental policy will be allowed to run. Your supplemental policies can specify filepath rules, trusted publishers, and more. 

    Refer to [Deploy multiple Windows Defender Application Control Policies](deploy-multiple-windows-defender-application-control-policies.md) for guidance on creating supplemental policies and [Deploy Windows Defender Application Control policy rules and file rules](select-types-of-rules-to-create.md) to choose the right type of rules to create for your policy.

    > [!Note] Policies which are supplementing the S mode base policy must use **-SupplementsBasePolicyID 5951A96A-E0B5-4D3D-8FB8-3E5B61030784**, as this is the S mode policy ID.
2. Sign policy
    
    Supplemental S mode policies must be digitally signed. To sign your policy, you can choose to use the Device Guard Signing Service or your organization's custom Public Key Infrastructure (PKI). Refer to [Use the Device Guard Signing Portal in the Microsoft Store for Business](use-device-guard-signing-portal-in-microsoft-store-for-business.md) for guidance on using DGSS and [Create a code signing cert for WDAC](create-code-signing-cert-for-windows-defender-application-control.md) for guidance on signing using an internal CA.

    Once your policy is signed, you must authorize the signing certificate you used to sign the policy and optionally one or more additional signers that can be used to sign updates to the policy in the future. Use Add-SignerRule to add the signing certificate to the WDAC policy, filling in the correct path and filenames for `<policypath>` and `<certpath>`: 
       
    `Add-SignerRule -FilePath <policypath> -CertificatePath <certpath> -User -Update`
3. Deploy the signed supplemental policy using Microsoft Intune

    Upload the signed policy to Intune and assign it to user or device groups. Intune will generate tenant- and device- specific authorization tokens. Intune then deploys the corresponding authorization token and supplemental policy to each device in the assigned group. Together, these expand the S mode base policy on the device. 
    <!-- Intune link?-->

> [!Note] When updating your supplemental policy, ensure that the new version number is strictly greater than the previous one. Using the same version number is not allowed by Intune. Refer to [Set-CIPolicyVersion](https://docs.microsoft.com/powershell/module/configci/set-cipolicyversion?view=win10-ps) for information on setting the version number.

# Standard Process for Deploying Apps through Intune
![Deploying Apps through Intune](images/wdac-intune-app-deployment.png)
Refer to [Intune Standalone - Win32 app management](https://docs.microsoft.com/intune/apps-win32-app-management)  for guidance on the existing procedure of packaging signed catalogs and app deployment.

# Optional: Process for Deploying Apps using Catalogs
![Deploying Apps using Catalogs](images/wdac-intune-app-catalogs.png)
Your supplemental policy can be used to significantly relax the S mode base policy, but there are security trade-offs you must consider in doing so. For example, you can use a signer rule to trust an external signer, but that will authorize all apps signed by that certificate, which may include apps you don’t want to allow as well.

Instead of authorizing signers external to your organization, Intune has added new functionality to make it easier to authorize existing applications (without requiring repackaging or access to the source code) through the use of signed catalogs. This works for apps which may be unsigned or even signed apps when you don’t want to trust all apps that may share the same signing certificate.

The basic process is to generate a catalog file for each app using Package Inspector, then sign the catalog files using the DGSS or a custom PKI. After that, IT Pros can use the standard Intune app deployment process outlined above. Refer to [Deploy catalog files to support Windows Defender Application Control](deploy-catalog-files-to-support-windows-defender-application-control.md) for more in-depth guidance on generating catalogs. 

> [!Note] Every time an app updates, you will need to deploy an updated catalog. Because of this, IT Pros should try to avoid using catalog files for applications that auto-update and direct users not to update applications on their own.


