# Brother DCP-B7640DW — Zabbix Template

A custom Zabbix 7.4 template for monitoring the Brother DCP-B7640DW multifunction printer using SNMPv1.

The template was built using a full Printer-MIB walk. Printer status, error-state, and uptime OIDs were confirmed using `snmpget`.

## Template Information

| Property       | Value                                    |
| -------------- | ---------------------------------------- |
| Template file  | `brother-dcp-b7640dw-snmp.xml`           |
| Device model   | Brother DCP-B7640DW                      |
| Protocol       | SNMPv1                                   |
| Zabbix version | 7.4                                      |
| Validation     | Tested template; selected OIDs confirmed |

## Monitoring Features

* Device description, name, serial number, and location.
* Device uptime.
* Printer status.
* Error state decoded into readable flags.
* LCD message.
* Cover status.
* Lifetime page counter.
* Black toner description and reported level.
* Drum remaining value and capacity.
* Calculated drum remaining percentage.
* ICMP ping availability.

Decoded error-state flags can include values such as `doorOpen`, `jammed`, and `lowPaper`, depending on the printer's reported status.

## Triggers

* SNMP unavailable.
* ICMP ping failure.
* Cover open.
* Printer error flags detected.
* Toner empty.
* Drum remaining below 10%.
* Drum remaining below 5%.
* Lifetime page counter decreased.
* Printer reboot detected.

## Toner Monitoring Limitation

The tested printer reports the following standard Printer-MIB values:

* Toner level: `-3` — some toner remains, but the amount is unknown.
* Toner capacity: `-2` — a normal numeric capacity is unavailable.

Consequently, a reliable toner percentage cannot be calculated from these standard SNMP readings.

The toner-empty trigger activates when the reported level reaches `0`. The template also checks the `lowToner` and `noToner` error-state flags for warnings.

Brother-specific OIDs under `1.3.6.1.4.1.2435` may provide more detailed toner information. Those OIDs must be identified and verified before adding them to the template.

## Drum Monitoring

The template monitors the reported drum remaining value and capacity and calculates a percentage when usable values are available.

Alerts are configured for drum remaining below 10% and below 5%.

## Requirements

* Zabbix Server or Proxy 7.4.
* SNMPv1 enabled on the printer.
* Network connectivity between Zabbix and the printer.
* `snmpwalk`, `snmpget`, and `fping` available on the monitoring server where required.

## Installation

1. Navigate to **Data collection → Templates → Import**.
2. Import `brother-dcp-b7640dw-snmp.xml`.
3. Create a host for the Brother DCP-B7640DW.
4. Add an SNMP interface with the printer's IP address.
5. Select SNMPv1 and configure the correct community.
6. Link the template to the host.
7. Open **Monitoring → Latest data** and review status, error flags, consumables, counters, and uptime.

Configure the host interface community consistently with the `{$SNMP_COMMUNITY}` macro.

## Troubleshooting

### SNMP items show Not supported

Test the affected OID directly:

```bash
snmpget -v1 -c 'YOUR_SNMP_COMMUNITY' PRINTER_IP OID
```

Replace `OID` with the numeric OID from the affected item. Review the returned value, item configuration, and preprocessing.

### Toner percentage is unavailable

This is an expected limitation when the printer reports `-3` for toner level and `-2` for toner capacity. These readings do not provide a valid toner percentage.

To investigate Brother-specific consumable information, collect a sanitized walk:

```bash
snmpwalk -v1 -c 'YOUR_SNMP_COMMUNITY' PRINTER_IP 1.3.6.1.4.1.2435
```

### Drum percentage is missing

Inspect the raw drum remaining and capacity values and confirm that the calculated item's preprocessing handles unavailable readings correctly.

## Security

* Avoid the default SNMP community `public` in production.
* Restrict SNMP access to trusted monitoring systems.
* Redact credentials, serial numbers, and sensitive network details from shared diagnostics.

## License

This project is licensed under the MIT License. See the repository's [LICENSE](../../LICENSE) file for details.

## Disclaimer

Provided as-is. Monitoring results depend on the device's SNMP implementation and firmware.
