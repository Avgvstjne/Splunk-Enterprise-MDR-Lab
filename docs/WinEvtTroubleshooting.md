## Splunk Lab: Windows Event Forwarding and Troubleshooting

## Objective

Build a small SIEM lab using a Windows endpoint, Splunk Universal Forwarder, and Splunk Enterprise running on an Ubuntu VM. Validate endpoint event collection, forwarding, receiving, and indexing before adding Sysmon-based telemetry and detections.

## Lab architecture

- Endpoint: Windows PC running Splunk Universal Forwarder.
- SIEM: Ubuntu VM running Splunk Enterprise.
- Transport: Splunk TCP receiving on port 9997.
- VM networking: VMware NAT (VMnet8) with a port-forwarding rule for TCP 9997.

## Troubleshooting: Windows event ingestion

The Universal Forwarder was installed on the intended Windows endpoint.

## Validation performed:

- "splunk.exe list forward-server" reported active forwarding to the Ubuntu Splunk receiver.
- Forwarder "splunkd.log" showed a connection to the intended receiver IP address and port.
- Splunk internal TCP input metrics ("group=tcpin_connections") showed the endpoint hostname, confirming that the receiver registered a forwarding connection.
- Searches for Windows event logs initially returned no results.

## The Windows Universal Forwarder's local "inputs.conf" was inspected. It contained enabled inputs for the Application, System, and Security event channels:

- "[WinEventLog://Application]"
- "[WinEventLog://System]"
- "[WinEventLog://Security]"

Each input had "disabled = 0", "start_from = oldest", and "current_only = 0".

The Ubuntu Splunk Enterprise instance was then inspected. Its receiving "inputs.conf" was empty. Adding the following stanza enabled the required TCP receiver:

[splunktcp://9997]
disabled = 0

After this change, Windows logs began arriving in Splunk.

Finding: The receiver input configuration was a confirmed part of the ingestion problem. The subsequent disk-space issue became the next recovery priority.

## Troubleshooting: Disk-space exhaustion

After Windows logs started arriving, the Splunk instance reached a disk-space limit and searches became unavailable.

The "minFreeSpace" setting was reduced to 500 MiB, but this did not restore search functionality.

The cause of the remaining disk-space problem has not yet been confirmed.

Planned diagnostic steps:

1. Check filesystem capacity using "df -h".
2. Check inode availability using "df -i".
3. Compare disk usage across Splunk indexed data, dispatch artifacts, and internal logs.
4. Increase the virtual disk capacity of Ubuntu VM.
5. Boot Ubuntu and check whether it sees the additional space
6. Expand the root partition to use the additional disk space.
7. Expand the filesystem, commonly with resize2fs for ext4.
6. Verify that searches and indexing resume.
7. Reconfirm that new Windows events are searchable.

The dispatch directory will not be cleared indiscriminately. Its contents will be inspected first, and index bucket files will not be manually deleted.

Current status

Component| Status
Windows Universal Forwarder installed| Confirmed
Forwarder connection to receiver| Confirmed
Splunk receiver configured on TCP 9997| Configured
Windows event-log inputs enabled| Confirmed in local configuration
Windows logs arriving in Splunk| Observed after receiver configuration change
Search and indexing recovery after disk exhaustion| Confirmed
Reconfirm that new Windows events are searchable | Pending Verification
Sysmon installation and ingestion| Not yet completed
Detection development| Pending baseline validation

Next milestones

1. Verify searchable Windows Application and System events.
2. Document the working ingestion path and capture evidence.
3. Install Sysmon on the Windows endpoint.
4. Forward and validate Sysmon events.
5. Develop and test an initial endpoint detection.
