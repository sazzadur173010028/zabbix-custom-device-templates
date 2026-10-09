# Canon imageRUNNER iR2520 — Zabbix Template

A custom Zabbix 7.4 template for monitoring the Canon imageRUNNER iR2520 multifunction photocopier using SNMPv1.

The template monitors device information, uptime, printer status, copy and print counters, scan totals, paper level, and reachability.

## Template Information

| Property       | Value                    |
| -------------- | ------------------------ |
| Template file  | `canon-ir2520-snmp.xml`  |
| Device model   | Canon imageRUNNER iR2520 |
| Protocol       | SNMPv1                   |
| Zabbix version | 7.4                      |
| Validation     | Tested template          |

## Monitoring Features

* Canon model and serial number.
* Contact and location.
* Device uptime.
* Printer status.
* Decoded error and door state.
* Canon enterprise status.
* Ethernet speed.
* Black-and-white copy counters: large and small.
* Calculated total copy count.
* Print total and scan total.
* Lifetime page counter.
* Tray 1 paper level.
* ICMP ping availability.

## Copy Counter Calculation

The template calculates the total copy count from the large and small black-and-white copy counters because the dedicated copy-total OID was not available on the tested model.

`Copy Total = Large Copy Counter + Small Copy Counter`

## Validation Notes

* The expected IF-MIB interface status and traffic counters did not return data on the tested iR2520. The unsupported interface items and link-down trigger were removed.
* The copy-total OID `1.3.6.1.4.1.1602.1.11.1.3.1.4.101` was unavailable on this model; the total is calculated from the supported counters.
* Ethernet speed and Tray 1 paper-level readings should be interpreted according to the values returned by the device.

ICMP ping is used for basic reachability monitoring.

## Requirements

* Zabbix Server or Proxy 7.4.
* SNMPv1 enabled on the printer.
* Network connectivity between Zabbix and the printer.
* `snmpwalk`, `snmpget`, and `fping` available on the monitoring server where required.

## Installation

1. Navigate to **Data collection → Templates → Import**.
2. Import `canon-ir2520-snmp.xml`.
3. Create a host for the Canon iR2520.
4. Add an SNMP interface with the printer's IP address.
5. Select SNMPv1 and configure the correct community.
6. Link the template to the host.
7. Open **Monitoring → Latest data** to review status, counters, and availability.

Configure the host interface community consistently with `{$SNMP_COMMUNITY}`.

## Troubleshooting

### SNMP item shows Not supported

Test the relevant OID:

```bash
snmpget -v1 -c 'YOUR_SNMP_COMMUNITY' PRINTER_IP OID
```

Replace `OID` with the numeric OID configured in the affected item. Compare the raw response with the item's configuration and preprocessing.

### Copy Total is incorrect

Verify that both large and small copy counters return valid numeric values and that the calculated item adds them correctly.

### Interface counters return no data

The tested iR2520 did not expose the expected IF-MIB interface table. Use ICMP ping for reachability instead of relying on unavailable interface counters.

## Security

* Avoid the default SNMP community `public` in production.
* Restrict SNMP access to authorized monitoring systems.
* Redact credentials and sensitive network information from diagnostic output.

## License

This project is licensed under the MIT License. See the repository's [LICENSE](../../LICENSE) file for details.

## Disclaimer

Provided as-is. Monitoring coverage depends on the device's SNMP implementation and firmware.


