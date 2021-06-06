---
title: "CAR-2021-05-006: CertUtil Download With URLCache and Split Arguments"
layout: analytic
submission_date: 2021/05/11
information_domain: Analytic
subtypes: Process
analytic_type: TTP
contributors: Splunk Threat Research <research@splunk.com>
applicable_platforms: Windows
---

Certutil.exe may download a file from a remote destination using `-urlcache`. This behavior does require a URL to be passed on the command-line. In addition, `-f` (force) and `-split` (Split embedded ASN.1 elements, and save to files) will be used. It is not entirely common for `certutil.exe` to contact public IP space. However, it is uncommon for `certutil.exe` to write files to world writeable paths.\ During triage, capture any files on disk and review. Review the reputation of the remote IP or domain in question.


### ATT&CK Detection

|Technique|Subtechnique(s)|Tactic(s)|Level of Coverage|
|---|---|---|---|
|[Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)|N/A|[Command and Control](https://attack.mitre.org/tactics/TA0011/)|Moderate|


### Implementations

#### Pseudocode – CertUtil download (Pseudocode, CAR native)


Pseudocode implementation of the Splunk search below


```
processes = search Process:Create
certutil_downloads = filter processes where (
  exe ="C:\Windows\System32\certutil.exe" AND command_line = *urlcache* AND command_line = *split*)
output certutil_downloads
```


#### Splunk code (Splunk, Endpoint)


To successfully implement this search you need to be ingesting information on process that include the name of the process responsible for the changes from your endpoints into the `Endpoint` datamodel in the `Processes` node.


```
| tstats count min(_time) as firstTime max(_time) as lastTime from datamodel=Endpoint.Processes where Processes.process_name=certutil.exe Processes.process=*urlcache* Processes.process=*split* by Processes.dest Processes.user Processes.parent_process Processes.process_name Processes.process Processes.process_id Processes.parent_process_id
```



### Unit Tests

#### Test Case 1

**Configurations:** Using Splunk [Attack Range](https://github.com/splunk/attack_range)

Replay the detection [dataset](https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1105/atomic_red_team/windows-sysmon.log)  using the Splunk attack range with the commands below

```
python attack_range.py replay -dn data_dump [--dump NAME_OF_DUMP]
```

#### Test Case 2

**Configurations:** Using [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam)

execute the atomic test [T1105](https://github.com/redcanaryco/atomic-red-team/tree/master/atomics/T1105) against a Windows target.

```
Invoke-AtomicTest T1105
```


