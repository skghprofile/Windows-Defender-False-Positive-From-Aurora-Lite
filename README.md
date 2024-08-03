# Windows Defender False Positive From Aurora Lite

## Table of Contents
- [Windows Defender False Positive From Aurora Lite](#windows-defender-false-positive-from-aurora-lite)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Windows Defender Results](#windows-defender-results)
  - [Quarantined File Analysis](#quarantined-file-analysis)
  - [False Positive Analysis](#false-positive-analysis)
  - [Windows Defender Exclusion](#windows-defender-exclusion)

---

## Introduction

Wanting to gain more experience with enterprise EDR solutions, I came across Nextron Systems AURORA Agent Lite.

From Nextron Systems's [Aurora Agent User Manual GitHub repository](https://github.com/NextronSystems/aurora-agent-manual) they describe the Aurora Agent as:

The Aurora Agent is a lightweight and customizable EDR agent based on Sigma. It uses Event Tracing for Windows (ETW) to recreate events that are very similar to the events generated by Microsoft's Sysmon and applies Sigma rules and IOCs to them. AURORA complements the open Sigma standard with "response actions" that allow users to react to a Sigma match. It is everything that other EDRs aren't.

- It is completely transparent and fully customizable due to the open Sigma rule set and configuration files
- It saves 99% of the network bandwidth and storage
- It works completely on-premises, no data leaves your network
- It can be configured to use only a limited amount of resources

We offer an enterprise and a "Lite" version, which is free of charge. The free version uses only the open source rule set, lacks comfort features and a central management.

Since early 2024, I have been using Aurora Agent Lite on my personal computer to deepen my understanding of Sigma rules and the events that trigger these alerts. By examining the dashboard, I can determine whether alerts are malicious and require action or if they are false positives.

## Windows Defender Results

In the past few weeks, I have intermittently received notifications from Microsoft Windows Defender about quarantined files. Reviewing the Windows Defender protection history reveals the following results:

![Windows Defender Protection History](/Images/WDFP-img01.PNG)
![Windows Defender Protection History](/Images/WDFP-img02.PNG)

Looking at the results from the images above, each one is triggered by the same file "C:\Program Files\Aurora-Agent\signatures\sigma-rules\public\windows\powershell\powershell_classic\posh_pc_tamper_windows_defender_set_mp.yml" which is a part of the Aurora Agent program and the file that triggers the quarantine is a .yml configuration file for a rule.

## Quarantined File Analysis

Opening the .yml file in question yields this configuration rule text:
```yaml
title: Tamper Windows Defender - PSClassic
id: ec19ebab-72dc-40e1-9728-4c0b805d722c
related:
 - id: 14c71865-6cd3-44ae-adaa-1db923fae5f2
      type: similar
status: experimental
description: Attempting to disable scheduled scanning and other parts of Windows Defender ATP or set default actions to allow.
references:
 - https://github.com/redcanaryco/atomic-red-team/blob/f339e7da7d05f6057fdfcdd3742bfcf365fee2a9/atomics/T1562.001/T1562.001.md
author: frack113, Nasreddine Bencherchali (Nextron Systems)
date: 2021/06/07
modified: 2024/01/02
tags:
 - attack.defense_evasion
 - attack.t1562.001
logsource:
    product: windows
    category: ps_classic_provider_start
detection:
    selection_set_mppreference:
        Data|contains: 'Set-MpPreference'
    selection_options_bool_allow:
        Data|contains:
 - '-dbaf $true'
 - '-dbaf 1'
 - '-dbm $true'
 - '-dbm 1'
 - '-dips $true'
 - '-dips 1'
 - '-DisableArchiveScanning $true'
 - '-DisableArchiveScanning 1'
 - '-DisableBehaviorMonitoring $true'
 - '-DisableBehaviorMonitoring 1'
 - '-DisableBlockAtFirstSeen $true'
 - '-DisableBlockAtFirstSeen 1'
 - '-DisableCatchupFullScan $true'
 - '-DisableCatchupFullScan 1'
 - '-DisableCatchupQuickScan $true'
 - '-DisableCatchupQuickScan 1'
 - '-DisableIntrusionPreventionSystem $true'
 - '-DisableIntrusionPreventionSystem 1'
 - '-DisableIOAVProtection $true'
 - '-DisableIOAVProtection 1'
 - '-DisableRealtimeMonitoring $true'
 - '-DisableRealtimeMonitoring 1'
 - '-DisableRemovableDriveScanning $true'
 - '-DisableRemovableDriveScanning 1'
 - '-DisableScanningMappedNetworkDrivesForFullScan $true'
 - '-DisableScanningMappedNetworkDrivesForFullScan 1'
 - '-DisableScanningNetworkFiles $true'
 - '-DisableScanningNetworkFiles 1'
 - '-DisableScriptScanning $true'
 - '-DisableScriptScanning 1'
 - '-MAPSReporting $false'
 - '-MAPSReporting 0'
 - '-drdsc $true'
 - '-drdsc 1'
 - '-drtm $true'
 - '-drtm 1'
 - '-dscrptsc $true'
 - '-dscrptsc 1'
 - '-dsmndf $true'
 - '-dsmndf 1'
 - '-dsnf $true'
 - '-dsnf 1'
 - '-dss $true'
 - '-dss 1'
    selection_options_actions_func:
        Data|contains:
 - 'HighThreatDefaultAction Allow'
 - 'htdefac Allow'
 - 'LowThreatDefaultAction Allow'
 - 'ltdefac Allow'
 - 'ModerateThreatDefaultAction Allow'
 - 'mtdefac Allow'
 - 'SevereThreatDefaultAction Allow'
 - 'stdefac Allow'
    condition: selection_set_mppreference and 1 of selection_options_*
falsepositives:
 - Legitimate PowerShell scripts that disable Windows Defender for troubleshooting purposes. Must be investigated.
level: high
```

Looking through the file, I can conclude that this file is not malicious and that Windows Defender has identified this file as a false positive.

## False Positive Analysis

Curious about why this recently triggered Microsoft's Windows Defender, I searched online to see if any others are experiencing the same issue. 

Fortunately, I was not the only person experiencing this issue and found a [GitHub issue thread](https://github.com/SigmaHQ/sigma/issues/4925) outlining this problem.

![GitHub Issue Thread](/Images/WDFP-img03.PNG)

The user nasbench on GitHub explains the issue and solution:

![GitHub Issue Thread](/Images/WDFP-img04.PNG)

Because I only have the Aurora Agent Lite and not the enterprise version, there is little I can do since the rules aren't encrypted, and Windows Defender flags the plaintext strings and keywords as malicious.

In the meantime, I can submit the file to Microsoft and report the file as a false positive in hopes that in future Windows Defender database updates, this file will not trigger Windows Defender to flag the file as malicious and quarantine it.

## Windows Defender Exclusion

I already tried to add the file as an exclusion within Windows Defender, but Windows Defender still automatically quarantines the file. My thought is that the exclusion only works for manual scans and not the automatic portion of Windows Defender.

I doublechecked my exclusion settings to ensure that the file in question is the same one that keeps getting flagged automatically:

![Windows Defender Window](/Images/WDFP-img05.PNG)
![Windows Defender Window](/Images/WDFP-img06.PNG)
![Windows Defender Window](/Images/WDFP-img07.PNG)