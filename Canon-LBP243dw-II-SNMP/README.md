# Canon i-SENSYS LBP242 II / LBP243 II — Zabbix Template

A custom Zabbix 7.4 template for monitoring Canon i-SENSYS LBP242 II and LBP243 II printers using SNMPv1.

The template covers printer identity, availability, device status, toner information, page counters, Ethernet statistics, and firmware information.

## Template Information

| Property         | Value                                |
| ---------------- | ------------------------------------ |
| Template file    | `canon-lbp242ii-243ii-snmp.xml`      |
| Supported models | Canon i-SENSYS LBP242 II / LBP243 II |
| Protocol         | SNMPv1                               |
| Zabbix version   | 7.4                                  |
| Validation       | Tested template                      |

## Monitoring Features

* Printer model, serial number, and location.
* Device uptime.
* Printer status and error state.
* Ethernet operational status and speed.
* Ethernet traffic, errors, and discards.
* Black toner description and reported level.
* Lifetime page counter.
* Canon private counters.
* Firmware version information.
* ICMP ping availability.

## Triggers

* SNMP unavailable.
* ICMP ping failure.
* Toner warning between 10% and 20%.
* High toner alert below 10%.

Toner alerts depend on the values reported by the printer and the configured preprocessing.

## Requirements

* Zabbix Server or Proxy 7.4.
* SNMPv1 enabled on the printer.
* Network connectivity between the printer and Zabbix.
* `snmpwalk`, `snmpget`, and `fping` available on the monitoring server where required.

## Installation

1. Open **Data collection → Templates → Import**.
2. Import `canon-lbp242ii-243ii-snmp.xml`.
3. Create a host for the printer.
4. Add an SNMP interface using the printer's IP address.
5. Select SNMPv1 and configure the appropriate community.
6. Link this template to the host.
7. Check **Monitoring → Latest data** to review the collected values.

Configure the host interface community consistently with the `{$SNMP_COMMUNITY}` macro.

## Technical Notes

### Printer-MIB consumables

The toner and lifetime-counter items use Printer-MIB table indexing. The following command can be used to inspect raw consumable values when troubleshooting:

```bash
snmpwalk -v1 -c 'YOUR_SNMP_COMMUNITY' PRINTER_IP 1.3.6.1.2.1.43.11.1
```

Replace the example community and IP with your actual values. Confirm the returned table indices if an item reports an unexpected value.

### Ethernet monitoring

Ethernet items depend on the interface information exposed by the printer. If an item returns no data on a particular firmware version, inspect the relevant OID and item error before adjusting the configuration.

### Toner thresholds

The configured thresholds are intended to warn between 10% and 20% and raise a higher-severity alert below 10%. Confirm the returned units and preprocessing when reviewing toner values.

## Security

* Avoid using the default SNMP community `public` in production.
* Restrict SNMP access to trusted monitoring systems.
* Never publish credentials or sensitive network information.

## License

This project is licensed under the MIT License. See the repository's [LICENSE](../../LICENSE) file for details.

## Disclaimer

Provided as-is. Monitoring results depend on the OIDs and values exposed by the printer firmware.
