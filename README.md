# ELK_SIEM_lab

## Introduction
This SOC Lab Challenge is designed to simulate real-world Security Operations Center (SOC) activities. The goal is to strengthen my hands-on skills in monitoring, detecting, and responding to cybersecurity incidents. Over four weeks, I’ll build a complete SOC environment by configuring SIEM tools, analyzing attacks, and managing incidents to better understand how SOC teams operate in real scenarios.

## Objective
The objective of this challenge is to broaden my practical knowledge of SOC through step-by-step labs that cover:
* Setting up and using the ELK Stack (Elasticsearch, Logstash, Kibana) for log analysis.
* Detecting and investigating brute-force attacks, malware infections, and C2 traffic.
* Creating dashboards, alerts, and automated responses.
* Managing incidents using a ticketing system to simulate SOC workflows.
By completing this challenge, I aim to build real-world SOC experience, strengthen my analytical skills, and demonstrate readiness for an entry-level SOC analyst role.

## Requirment
To complete this Lab, the following virtual machines are used:

| VM                      | RAM   | Processors| Purpose        |
|-------------------------|-------|-----------|----------------|
| ELK Stack               | 8 GB  | 4    | Centralized log collection, analysis, and visualization |
| Mythic C2 Server        | 3 GB  | 2    | Simulate command & control attacks               |
| Fleet Server            | 3 GB  | 2    | Endpoint management and log forwarding           |
| Ubuntu Endpoint         | 3 GB  | 2    | Linux endpoint for log generation and testing    |
| Windows Server          | 3 GB  | 2    | Windows environment for brute-force/malware analysis |
| OS Ticket Server        | 2 GB  | 2    | Ticketing system for incident management         |
| Kali Linux              | 2 GB  | 2    | Attack simulation and penetration testing        |



