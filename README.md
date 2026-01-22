# SOC-Automation-Project
Created by: Ben Holomah
Goal: Land a Junior SOC analyst role by May 2026
This is a "journal entry" of my process creating my first SOC automation lab. My purpose in creating it is to help me gain and apply technical skills that I'd be using as a SOC analyst. I want to document the journey, displaying my my logic, desire to sharpen my skills, and desire to acquire experience using tools used in the field.

---

## Homelab Enviornment Baseline
- **Hypervisor:** VMware
- **SIEM:** Wazuh + Splunk
- **Endpoints:** Windows 10, Kali Linux Server

---

## Baseline Setup
**Date** January 21, 2026
- I installed win10 onto my VM. During the installation process I opted for a Local Account via the Offline Account/ Limited Experience path.

**Logic:** I figured it would provide a cleaner lab enviroment for me. It'll reduce any Microsoft cloud sync services logs or anything related.

- I wanted to confirm that my offline account was already an admin for my own peace of mind by entering the command 'net user %username%'. (It was).

---
