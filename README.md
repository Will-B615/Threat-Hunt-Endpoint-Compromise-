# Threat-Hunt-Endpoint-Compromise-

## Overview 

This project documents a confirmed endpoint compromise investigated through Microsoft Defender Advanced Hunting in a cyber range environment. The case began with Help Desk Ticket #4451 after a Finance user reported repeated overnight login prompts on host `npt-ws01`, and the investigation focused on activity from 22 April 2026, 04:30 to 06:00 UTC. 

  

This portfolio entry demonstrates practical skills in threat hunting, incident scoping, timeline reconstruction, indicator extraction, persistence analysis, containment planning, and incident reporting using Microsoft security telemetry. 

  

## Skills Demonstrated 

- Microsoft Defender Advanced Hunting for endpoint and identity investigation. 

- Detection and analysis of suspicious authentication, remote execution, and command-and-control activity. 

- Reconstruction of attacker behavior using process, network, file, registry, and account telemetry. 

- Mapping findings to the NIST SP 800-61 incident response lifecycle. 

- Development of evidence-based containment, eradication, and recovery actions. 

  

## Environment 

| Category | Details | 

| Platform | Microsoft Defender portal with Advanced Hunting. | 

| Primary host | `npt-ws01`.| 

| Secondary host in scope | `npt-srv01`.| 

| Investigation window | 22 April 2026, 04:30 to 06:00 UTC.| 

| Initial trigger | Help Desk Ticket #4451 reporting repeated login prompts.| 

---
<img width="770" height="295" alt="Screenshot 2026-07-16 110601" src="https://github.com/user-attachments/assets/4b0ea420-00a5-448f-99d8-8d80c2a3fb11" />


 


 ---
## Investigation Objective 

The goal was to determine whether the reported login prompts were benign or evidence of a real compromise, then identify what happened, what artifacts were created, and whether the attacker moved beyond the initial workstation. 

  

## Detection Summary 

The strongest early detection point was unauthorized successful use of the `helpdesk` account from external IP `20.110.92.50`, because that event tied the intrusion to suspicious access before later persistence activity appeared. A second high-confidence signal was the execution chain showing `cmd.exe /Q /c start "" "C:\Windows\Temp\WindowsUpdate.exe"` with `wmiprvse.exe` as the parent process, which indicated likely remote WMI-based execution. 

 --- 

<img width="787" height="157" alt="Screenshot 2026-07-14 142713" src="https://github.com/user-attachments/assets/02f90f86-7483-467b-b879-fe40cb0b372b" />


 
---
## Key Findings 

| Finding | Evidence | 

| Unauthorized access | Successful logon using account `helpdesk` from `20.110.92.50`.| 

| Malicious execution | Payload launched with `cmd.exe /Q /c start "" "C:\Windows\Temp\WindowsUpdate.exe"`.| 

| Execution method | Parent process `wmiprvse.exe`, consistent with remote WMI execution.| 

| Command and control | Outbound communication to `updates.abordasync.website`.| 

| Malware hash | `20cef6a013953890f9d38605d25d60dd63b42b09946bbb18ddb4a456da306e77`.| 

| Persistence 1 | Run key `WindowsHealthCheck`.| 

| Persistence 2 | Scheduled task `GoogleUpdaterTask`.| 

| Persistence 3 | Service `WindowsHealthSvc`.| 

| Backdoor account | Local account `nexus_admin` / `nexusadmin` created and added to Administrators.| 

| Scope expansion | Activity extended toward `npt-srv01`.| 

  
---

<img width="855" height="201" alt="Screenshot 2026-07-14 134505" src="https://github.com/user-attachments/assets/dcfc5cf2-9f0e-4b17-9cf5-1d7ba0734605" />
 

 

<img width="1072" height="176" alt="Screenshot 2026-07-12 131050" src="https://github.com/user-attachments/assets/eb549cb8-ddbd-4c5c-9a5a-5ef55f0fcb24" />

 

--- 

## Reconstructed Attack Chain 

1. A suspicious successful logon occurred on `npt-ws01` using the `helpdesk` account from external IP `20.110.92.50`. 

2. The attacker launched `C:\Windows\Temp\WindowsUpdate.exe` through `cmd.exe`, with `wmiprvse.exe` confirming likely remote WMI execution. 

3. The compromised host communicated with `updates.abordasync.website`, indicating command-and-control activity. 

4. The attacker established persistence through a Run key, scheduled task, and service named `WindowsHealthCheck`, `GoogleUpdaterTask`, and `WindowsHealthSvc`. 

5. A privileged local backdoor account named `nexus_admin` / `nexusadmin` was created and added to the local Administrators group. 

6. Evidence suggested movement toward a second host, `npt-srv01`, which expanded the investigation scope beyond a single endpoint. 

  


 

The timeline visualization above shows Defender Advanced Hunting results for npt-ws01, grouped by ActionType. The sequence of ProcessCreated, ConnectionSuccess, and RegistryValueSet events illustrates the attack chain across execution, command‑and‑control, and persistence phases during the incident window. 

 

## MITRE ATT&CK-Aligned Observations 

The observed activity is consistent with valid account abuse, remote service or WMI-style execution, command-and-control traffic, registry/task/service persistence, local account creation, privilege escalation, and probable lateral movement. Even though the exercise focused on reconstruction rather than formal ATT&CK mapping, the telemetry clearly supports documenting credential abuse, execution, persistence, and scope expansion as distinct phases of the intrusion. 

  

## Containment Actions 

- Isolate `npt-ws01` to stop outbound command-and-control and prevent further lateral movement. 

- Investigate and isolate `npt-srv01` if matching activity is confirmed or suspected. 

- Disable the compromised `helpdesk` account, reset credentials, invalidate active sessions, and review recent authentication activity. 

- Disable and remove the attacker-created local account `nexus_admin` / `nexusadmin` and review local administrator group membership. 

- Block `updates.abordasync.website`, `20.110.92.50`, the implant hash, and execution of similarly named binaries from Temp paths. 

- Remove persistence artifacts including `WindowsHealthCheck`, `GoogleUpdaterTask`, `WindowsHealthSvc`, and `C:\Windows\Temp\WindowsUpdate.exe` after evidence preservation. 

  

## Lessons Learned 

This case shows why suspicious successful logons should be treated as a high-value signal, especially when a support or privileged account authenticates from an unexpected external IP. It also highlights the need to monitor or restrict remote administrative methods such as WMI, improve detections for Temp-folder payload execution and persistence creation, and perform rapid enterprise-wide hunts for related indicators. 

  

## What This Project Proves 

This project shows the ability to move from a vague user complaint to a defensible incident conclusion using endpoint telemetry, structured analysis, and incident response methodology. It also demonstrates practical experience documenting findings in a way that supports SOC triage, containment decision-making, and follow-on threat hunting across additional assets. 
