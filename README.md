# SOC-Automation-Project
Created by: Ben Holomah
Goal: Land a Junior SOC analyst role by May 2026
This is a "journal entry" of my process creating my first SOC automation lab. My purpose in creating it is to help me gain and apply technical skills that I'd be using as a SOC analyst. I want to document the journey, displaying my my logic, desire to sharpen my skills, and desire to acquire experience using tools used in the field.

---

## Homelab Enviornment Baseline
- **Hypervisor:** VMware
- **SIEM:** Wazuh
- **Endpoints:** Windows 10, Kali Linux Server

---

## Baseline Setup
**Date** January 21, 2026
- I installed win10 onto my VM. During the installation process I opted for a Local Account via the Offline Account/ Limited Experience path.

**Logic:** I figured it would provide a cleaner lab enviroment for me. It'll reduce any Microsoft cloud sync services logs or anything related.

- I wanted to confirm that my offline account was already an admin for my own peace of mind by entering the command 'net user %username%'. (It was).

---

SOC Automation Project
## Overview
- I started this lab by creating a network diagram to visualize the architecture and understand the flow of data between components before beginning implementation.
[SCREENSHOT: Network/architecture diagram]
---
## Windows Configuration — Sysmon
- I downloaded Sysmon along with a Sysmon configuration file to define what events to monitor.

- Ran PowerShell as Administrator
- Navigated to the Sysmon download directory using cd
- Executed the Sysmon installer with the configuration file

[SCREENSHOT: Sysmon installation command in PowerShell]
[SCREENSHOT: Sysmon service running in PowerShell]
---
## TheHive Server Setup — Ubuntu VM
- I created a dedicated Ubuntu server for TheHive and managed it remotely via SSH from my host machine. I chose SSH because it is a cleaner, more realistic method of server management and a skill I wanted to develop.
SSH Setup:

- Retrieved the Ubuntu VM's IP address using ip a
- Attempted to SSH from PowerShell but received a "connection refused" error
- Through research I learned SSH needed to be installed and enabled on the VM first
- Installed SSH using sudo apt install openssh-server -y
- Enabled and started the service using sudo systemctl enable ssh && sudo systemctl start ssh
- Verified it was running using sudo systemctl status ssh

[SCREENSHOT: SSH status showing active/running]
[SCREENSHOT: Successful SSH connection from PowerShell]
## Dependency Installation:

- Updated and upgraded the system packages
- Installed Java and verified the installation using java -version
- Installed and configured Apache Cassandra
- Installed Elasticsearch
- Installed TheHive

[SCREENSHOT: Java version output]
[SCREENSHOT: Cassandra/Elasticsearch/TheHive installation completion]
---

## TheHive & Cassandra Configuration

- Opened the Cassandra configuration file at /etc/cassandra/cassandra.yaml
- Changed the cluster name
- Updated the listen_address, rpc_address, and seed_provider to the TheHive server's IP address
- Stopped and restarted the Cassandra service to apply changes
- Verified the service status

[SCREENSHOT: cassandra.yaml with updated settings]
[SCREENSHOT: Cassandra service running]
Note: A recurring mishap during this phase was forgetting to prepend sudo to commands when working as a non-root user over SSH. Unlike cloud-hosted servers which default to root access, my local VM required explicit privilege escalation — a good real-world security practice.
---
## Elasticsearch Configuration & Troubleshooting
- Uncommented and configured cluster.name, node.name, network.host, http.port, and cluster.initial_master_nodes
- Removed the secondary node from cluster.initial_master_nodes

## Troubleshooting:
When attempting to start Elasticsearch I received an error. I checked the journal log using sudo journalctl -xeu elasticsearch.service which returned status=70/SOFTWARE, indicating a crash at the software level.

After investigation I identified two issues:
1. xpack.security.enabled: true was requiring SSL certificates that didn't exist
2. The auto-generated security configuration block was conflicting with my settings

I resolved this by removing the entire security auto-configuration block from elasticsearch.yml and adding xpack.security.enabled: false.
After approximately 45 minutes of troubleshooting, I discovered the root cause — the lab walkthrough was built on Elasticsearch v7, while my system had installed v8, which enables security by default and has breaking configuration changes. I removed v8, pinned the installation to v7.17, and Elasticsearch started successfully.
[SCREENSHOT: Elasticsearch service running after fix]
[SCREENSHOT: elasticsearch.yml final configuration]
---
## TheHive Configuration & Access
- Modified ownership of the /opt/thp directory using chown -R thehive:thehive /opt/thp
- Successfully accessed TheHive via browser at http://<TheHive-IP>:9000

[SCREENSHOT: TheHive login page]
[SCREENSHOT: TheHive dashboard after login]
---
## Wazuh — Windows Agent Configuration

- Configured the Windows 11 VM as a Wazuh agent
- Modified the ossec.conf file to monitor only Sysmon data by replacing default log sources with Microsoft-Windows-Sysmon/Operational
- Restarted the Wazuh service to apply changes
- Verified Sysmon telemetry was appearing in the Wazuh Discover view

[SCREENSHOT: ossec.conf Sysmon configuration]
[SCREENSHOT: Sysmon telemetry in Wazuh Discover]
---
## Mimikatz Detection

- Disabled Windows Defender on the Windows 11 VM to allow testing
- Downloaded Mimikatz — a credential dumping tool commonly used in real-world attacks
- Modified ossec.conf on the Wazuh manager — changed logall and logall_json from no to yes
- Created a new index in Wazuh to view wazuh-archives data
- Created a custom detection rule to trigger on Mimikatz execution:

  - Monitors for originalFileName matching mimikatz.exe
  - Triggers on Sysmon Event ID 1 (Process Creation)
  - Rule level set to 15 (critical)
  - Mapped to MITRE ATT&CK technique T1003 (Credential Dumping)


Successfully received an alert in the wazuh-alerts index upon execution

[SCREENSHOT: Custom rule in local_rules.xml]
[SCREENSHOT: Mimikatz alert appearing in Wazuh dashboard]
[SCREENSHOT: Alert details showing MITRE ATT&CK mapping]
---

## SOAR Automation — Shuffle

- Created an automation workflow on Shuffler.io
- Configured a webhook trigger to receive alerts from Wazuh
- Added a SHA256 regex capture node to extract the file hash from the Sysmon event
- Integrated VirusTotal API to automatically look up the extracted hash
- Resolved a connectivity issue — Shuffler.io is cloud-based and could not reach my local TheHive VM directly. I resolved this by using ngrok to create a secure tunnel, exposing TheHive publicly so Shuffle could communicate with it
- Integrated TheHive into the workflow to automatically create an alert/case for each detection
- Configured automated email notification to socautomation@protonmail.com upon detection

[SCREENSHOT: Full Shuffle workflow diagram]
[SCREENSHOT: Successful workflow execution run details]
[SCREENSHOT: TheHive case automatically created from Mimikatz alert]
[SCREENSHOT: Email notification received]
---
Summary
This project demonstrated the full pipeline of a SOC automation workflow:
Mimikatz executed on Windows → Sysmon detects it → Wazuh agent forwards log → Wazuh manager triggers custom rule → Shuffle SOAR receives webhook → Hash extracted and submitted to VirusTotal → Alert created in TheHive → Analyst notified via email
