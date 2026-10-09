# Brother DCP-B7640DW— Zabbix SNMP Template

## Overview

A custom Zabbix template for monitoring the Brother DCP-B7640DW printer through SNMP.

## Device

* **Manufacturer:** Brother
* **Model:** DCP-B7640DW
* **Monitoring Protocol:** SNMP
* **Validation:** Tested on the corresponding device model

## Requirements

* Network connectivity between Zabbix and the printer
* SNMP enabled on the printer
* Compatible SNMP configuration and permissions
* A compatible Zabbix server

## Installation

1. Download `template.xml` from this directory.
2. Open Zabbix → Data collection → Templates.
3. Select Import and upload the XML file.
4. Review and complete the import.
5. Configure the required host interface and macros.
6. Link the template to the target printer host.
7. Verify the collected data in Latest data.

## Monitoring Details

See the imported template's actual items, triggers, and discovery rules for the supported metrics.

## Screenshots

Screenshots demonstrating the template and its collected data can be added to the `screenshots/` directory.

## Security

Do not share SNMP community strings, credentials, or sensitive network information.

## Notes

This template was developed with AI assistance and validated through hands-on testing on the corresponding device model.

