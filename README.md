# Zabbix Custom Device Monitoring Templates

A collection of model-specific Zabbix monitoring templates for printers, photocopiers, and network video recorders (NVRs).

These templates were developed with AI assistance and validated through hands-on testing on their corresponding device models.

## Supported Devices

| Device Model           | Protocol | Device Type                 |
| ---------------------- | -------- | --------------------------- |
| Canon LBP243dw II      | SNMP     | Laser Printer               |
| Canon imageRUNNER 2520 | SNMP     | Photocopier                 |
| Hikvision DS-9664NI-I8 | ISAPI    | Network Video Recorder      |
| Brother DCP-B7640DW    | SNMP     | Multifunction Laser Printer |

## Repository Structure

* `Canon-LBP243dw-II-SNMP/`
* `Canon-imageRUNNER-2520-SNMP/`
* `Hikvision-DS-9664NI-I8-ISAPI/`
* `Brother-DCP-B7640DW-SNMP/`

Each device directory contains its template export, documentation, and available monitoring screenshots.

## Key Objectives

* Centralized device monitoring through Zabbix
* Model-specific monitoring configurations
* Improved device status visibility
* Reusable configurations for compatible devices
* Practical infrastructure monitoring and troubleshooting

## Requirements

* A compatible Zabbix server
* Network connectivity between Zabbix and the target device
* SNMP enabled and correctly configured for SNMP-based templates
* Appropriate ISAPI access and permissions for the Hikvision NVR template
* Required macros, interfaces, and credentials configured according to each template's documentation

## Installation

1. Download the XML file from the relevant device directory.
2. Sign in to the Zabbix web interface.
3. Navigate to **Data collection → Templates**.
4. Click **Import** and select the XML file.
5. Review the import options and complete the import.
6. Configure the required interfaces, macros, and authentication settings.
7. Link the template to the appropriate host.
8. Verify collected data and trigger behavior.

Menu names may vary slightly by Zabbix version.

## Testing

All four templates have been tested against their corresponding device models.

Refer to each device's README for the protocol, configuration requirements, and available monitoring details.

## Security

Never publish passwords, SNMP community strings, API credentials, authentication headers, or other secrets. Review screenshots and XML exports before publishing.

## Contributions

Feedback, issue reports, and suggestions for improvement are welcome.

## License

A license has not yet been selected. Until one is added, do not assume that others have permission to redistribute or modify this project.

