# SOC Automation Project
Built By: Ben Holomah

## Homelab Enviornment Baseline
- **Hypervisor:** VMware
- **SIEM:** Wazuh
- **Endpoints:** Windows 10, 2 Ubuntu Linux Servers (One for Wazuh and Another for TheHive)
- **Tools:** Wazuh, TheHive, Shuffle, Sysmon, VirusTotal, Mimikatz, Ubuntu, VMware

## Overview
- I started this lab by creating a network diagram to visualize the architecture and understand the flow of data between components before beginning implementation.
<img width="935" height="862" alt="Screenshot 2026-03-06 182616" src="https://github.com/user-attachments/assets/a575c5e2-7534-435a-a7ea-e8e2805739ca" />

---
## Windows Configuration — Sysmon
- I downloaded Sysmon along with a Sysmon configuration file to define what events to monitor.

- Ran PowerShell as Administrator
- Navigated to the Sysmon download directory using cd
- Executed the Sysmon installer with the configuration file

[SCREENSHOT: Sysmon installation command in PowerShell]
<img width="971" height="492" alt="Screenshot 2026-03-06 184809" src="https://github.com/user-attachments/assets/c39e806c-cc6f-4201-b6f9-1492ad56f0d4" />

[SCREENSHOT: Sysmon service running in PowerShell]
<img width="807" height="589" alt="Screenshot 2026-03-06 185223" src="https://github.com/user-attachments/assets/3bc81651-6795-4e98-88ab-841c6e11c909" />

---
## TheHive Server Setup — Ubuntu VM
- I created a dedicated Ubuntu server for TheHive and managed it remotely via SSH from my host machine. I chose SSH because it is a cleaner, more realistic method of server management and a skill I wanted to develop.
SSH Setup:

- Retrieved the Ubuntu VM's IP address using ip a
- Attempted to SSH from PowerShell but received a "connection refused" error
- Through research I learned SSH needed to be installed and enabled on the VM first
- Installed SSH using
```
sudo apt install openssh-server -y
```
- Enabled and started the service using
```
sudo systemctl enable ssh && sudo systemctl start ssh
```
- Verified it was running using
```
  sudo systemctl status ssh
```

[SCREENSHOT: SSH status showing active/running]
<img width="820" height="437" alt="Screenshot 2026-03-09 193212" src="https://github.com/user-attachments/assets/a1362b66-3082-49c2-b5d9-1757570c6941" />
[SCREENSHOT: Successful SSH connection from PowerShell]
<img width="1109" height="572" alt="Screenshot 2026-03-09 194005" src="https://github.com/user-attachments/assets/8cacbc14-6141-44e9-af2b-b5cd511f6490" />

## Dependency Installation:

- Updated and upgraded the system packages
- Installed Java and verified the installation using
```
  java -version
```
- Installed and configured Apache Cassandra
- Installed Elasticsearch
- Installed TheHive

[SCREENSHOT: Java version output]
<img width="1101" height="194" alt="Screenshot 2026-03-09 204144" src="https://github.com/user-attachments/assets/9930ed0e-8bdc-4ca7-a8dd-d49db78223fc" />

[SCREENSHOT: Cassandra/Elasticsearch/TheHive installation completion]
<img width="1108" height="608" alt="Screenshot 2026-03-09 195610" src="https://github.com/user-attachments/assets/7d5abc3a-44d3-4a97-9bef-fabae7a911ca" />

---

## TheHive & Cassandra Configuration

- Opened the Cassandra configuration file at /etc/cassandra/cassandra.yaml
- Changed the cluster name
- Updated the listen_address, rpc_address, and seed_provider to the TheHive server's IP address
- Stopped and restarted the Cassandra service to apply changes
- Verified the service status

[SCREENSHOT: cassandra.yaml with updated settings]
<img width="1021" height="561" alt="Screenshot 2026-03-09 200434" src="https://github.com/user-attachments/assets/158c429e-c2ca-4697-a29c-caa37934c140" />

<img width="1116" height="630" alt="Screenshot 2026-03-09 200850" src="https://github.com/user-attachments/assets/f38b06ea-8238-4a89-be93-a3e055c46c66" />

<img width="1099" height="577" alt="Screenshot 2026-03-09 201030" src="https://github.com/user-attachments/assets/251ea7df-e70d-4302-b678-6a738bb79ba1" />

[SCREENSHOT: Cassandra service running]
<img width="1100" height="609" alt="Screenshot 2026-03-09 202105" src="https://github.com/user-attachments/assets/a7187b92-9200-41be-acf1-9f9230c92477" />

Note: A recurring mishap during this phase was forgetting to prepend sudo to commands when working as a non-root user over SSH. Unlike cloud-hosted servers which default to root access, my local VM required explicit privilege escalation — a good real-world security practice.
---
## Elasticsearch Configuration & Troubleshooting
- Uncommented and configured cluster.name, node.name, network.host, http.port, and cluster.initial_master_nodes
- Removed the secondary node from cluster.initial_master_nodes

## Troubleshooting:
When attempting to start Elasticsearch I received an error. I checked the journal log using
```
sudo journalctl -xeu elasticsearch.service
```
which returned status=70/SOFTWARE, indicating a crash at the software level.

After investigation I identified two issues:
1. xpack.security.enabled: true was requiring SSL certificates that didn't exist
2. The auto-generated security configuration block was conflicting with my settings

- I resolved this by removing the entire security auto-configuration block from elasticsearch.yml and adding xpack.security.enabled: false.
- After approximately 45 minutes of troubleshooting, I discovered the root cause — the lab walkthrough was built on Elasticsearch v7, while my system had installed v8, which enables security by default and has breaking configuration changes.
- I removed v8, pinned the installation to v7.17, and Elasticsearch started successfully.
[SCREENSHOT: Elasticsearch service running after fix]
<img width="1110" height="514" alt="Screenshot 2026-03-09 212906" src="https://github.com/user-attachments/assets/96c5dbbd-9933-4981-9fce-10b27a4557e7" />

[SCREENSHOT: elasticsearch.yml final configuration]
<img width="1105" height="1315" alt="Screenshot 2026-03-09 204233" src="https://github.com/user-attachments/assets/43fd3a96-fd87-48d5-9ed0-2fbeef05351e" />

---
## TheHive Configuration & Access
- Modified ownership of the /opt/thp directory using
```
chown -R thehive:thehive /opt/thp
```
- Successfully accessed TheHive via browser at http://<TheHive-IP>:9000

[SCREENSHOT: TheHive login page]
<img width="1252" height="943" alt="image" src="https://github.com/user-attachments/assets/58aa3cff-130c-4347-84b5-bcbb1a523e7d" />

[SCREENSHOT: TheHive dashboard after login]
<img width="1279" height="911" alt="Screenshot 2026-03-09 215442" src="https://github.com/user-attachments/assets/5f10f935-0b9b-4283-b858-7a5b72c7cf17" />

---
## Wazuh — Windows Agent Configuration

- Configured the Windows 11 VM as a Wazuh agent
- Modified the ossec.conf file to monitor only Sysmon data by replacing default log sources with Microsoft-Windows-Sysmon/Operational
- Restarted the Wazuh service to apply changes
- Verified Sysmon telemetry was appearing in the Wazuh Discover view

[SCREENSHOT: ossec.conf Sysmon configuration]
<img width="871" height="596" alt="Screenshot 2026-03-09 223942" src="https://github.com/user-attachments/assets/c06b91cc-acb4-4240-b32b-cde6c4c90fce" />

[SCREENSHOT: Sysmon telemetry in Wazuh Discover]
<img width="1366" height="1297" alt="Screenshot 2026-03-09 224255" src="https://github.com/user-attachments/assets/edcc7d5a-aac5-4a6f-9086-400218935ec2" />

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
<img width="1370" height="959" alt="Screenshot 2026-03-09 233257" src="https://github.com/user-attachments/assets/830cf815-01c8-4084-9ac5-ad2ea42beccb" />

[SCREENSHOT: Mimikatz alert appearing in Wazuh dashboard]
<img width="1058" height="855" alt="Screenshot 2026-03-09 235356" src="https://github.com/user-attachments/assets/23b98482-3454-4ef2-abba-d5c047851f7e" />

[SCREENSHOT: Alert details showing MITRE ATT&CK mapping]
<img width="1053" height="1151" alt="Screenshot 2026-03-09 235555" src="https://github.com/user-attachments/assets/64cf2e1c-8a0d-4f9a-8297-07e0ab0bfc25" />

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
<img width="900" height="858" alt="image" src="https://github.com/user-attachments/assets/44f9fdda-f725-4b63-b5af-075945cd7d47" />

[SCREENSHOT: Successful workflow execution run details]
<img width="1675" height="1105" alt="Screenshot 2026-03-10 024136" src="https://github.com/user-attachments/assets/ed3154a2-52ab-45ce-9534-0d762c520008" />
<img width="1644" height="1024" alt="Screenshot 2026-03-10 024152" src="https://github.com/user-attachments/assets/471c9a77-eae3-4cc2-840b-8dfc210b1c96" />

[SCREENSHOT: TheHive case automatically created from Mimikatz alert]
<img width="1246" height="938" alt="Screenshot 2026-03-10 022455" src="https://github.com/user-attachments/assets/5d94573c-f305-4a45-a353-aff20db19d61" />

[SCREENSHOT: Email notification received]
<img width="1689" height="1110" alt="Screenshot 2026-03-10 024211" src="https://github.com/user-attachments/assets/8e8905d2-123c-4c95-b3c1-0b2bbe4e9751" />
## Note: Shuffle's built-in email node confirmed successful dispatch ("success": true) however the email was never received. This is a known issue with Shuffler.io's shared mail server, which has poor sender reputation and is frequently blocked by email providers. In a production environment this would be replaced with a dedicated SMTP integration for reliable delivery.
---
Summary
This project demonstrated the full pipeline of a SOC automation workflow:
Mimikatz executed on Windows → Sysmon detects it → Wazuh agent forwards log → Wazuh manager triggers custom rule → Shuffle SOAR receives webhook → Hash extracted and submitted to VirusTotal → Alert created in TheHive → Analyst notified via email
