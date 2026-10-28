# Splunk-Enterprise-MDR-Lab

```
splunk-enterprise-mdr-lab/
│
├── README.md
├── architecture/
|   └──
├── data-sources/
|   ├──
|   └──
├── detections/
|   ├──
|   └──
├── investigations/
│   ├── 
│   └──
├── dashboards/
|   ├──
|   └──
├── mitre/
|   └──
└── screenshots
```

### Objective

Build a practical enterprise-style Splunk environment demonstrating:
- Security telemetry ingestion
- SPL-based detection engineering
- Alert triage
- Security investigation
- Event correlation
- MITRE ATT&CK mapping
- Detection tuning
- SOC dashboards
- MDR incident response workflows

### Architecture

Initial architecture:

Splunk Enterprise
        |
        |
Windows endpoint
Windows Event Logs
Sysmon

The environment will progressively incorporate additional telemetry sources and simulated attack activity.
---
---

## Troubleshooting VMware Networking for a Splunk Lab

### Objective

Establish connectivity between a Windows endpoint running Splunk Universal Forwarder and an Ubuntu VM running Splunk Enterprise, using TCP port 9997.

### Initial symptoms

- Ubuntu did not obtain an IP address as expected during the networking investigation.
- DHCP troubleshooting did not resolve the issue.
- Attempted VMware bridged networking did not establish the required connectivity.
- VMware Bridge Protocol was enabled on the relevant Windows network adapter.
- VMware Virtual Network Editor showed the NAT network configured on VMnet8.

### Troubleshooting performed

1. Investigated Ubuntu network configuration and DHCP behaviour.
2. Attempted manual IPv4 configuration, then reverted the changes.
3. Investigated VMware Virtual Network Editor settings, including VMnet0 bridging and VMnet8 NAT.
4. Used Restore Defaults in Virtual Network Editor to reset the VMware virtual networking configuration.
5. Verified that VMware Bridge Protocol was checked in Windows Network Connections ("ncpa.cpl").
6. Investigated the physical network environment and identified that the Windows endpoint had been connected to guest Wi-Fi, which may restrict communication with devices on the main network.
7. Reset and reconfigured the Wi-Fi network, then retested bridged networking.
8. Bridged networking still did not work as required.
9. Abandoned bridging for this lab and returned to NAT.
10. Configured a VMware NAT port-forwarding rule for the Splunk receiving port, TCP 9997.

### Resolution approach

Rather than continuing to troubleshoot bridging without sufficient evidence, the lab was returned to NAT with port forwarding.

This provides a controlled path for forwarding endpoint telemetry into the Ubuntu Splunk receiver without depending on successful bridged networking.

### Validation status

- VMware NAT selected for the Ubuntu VM.
- NAT port forwarding configured for TCP 9997.
- Splunk Universal Forwarder installed on the intended Windows endpoint.
- Checks confirmed that the forwarder service was running and that TCP connectivity to port 9997 succeeded.
- End-to-end ingestion from the newly configured physical endpoint has been verified.
- Windows event-log inputs ingestion is confirmed in Local Configuration, and Sysmon ingestion is yet to be configured.

### Lessons learned

- Verify the actual virtual network configuration before pursuing guest-level DHCP troubleshooting.
- Use NetworkManager tools such as "nmcli" when appropriate for modern Ubuntu installations.
- Guest Wi-Fi client isolation is a possible explanation for cross-device connectivity failures, not proof of the cause.
- A successful TCP connection does not prove that events are being collected or indexed.
- Validate the pipeline in stages: network connectivity, receiver configuration, forwarder destination, enabled inputs, and searchable events.

### Next steps

1. Search Splunk for existing endpoint events.
2. Inspect the Universal Forwarder destination and effective input configuration.
3. Establish a baseline of Windows Application, System and Security logs.
4. Install and configure Sysmon.
5. Verify Sysmon events in Splunk before building detections.
