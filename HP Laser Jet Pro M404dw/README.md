# Zabbix Template: HP LaserJet Pro M404dw (SNMP)

Zabbix template for the **HP LaserJet Pro M404dw** over SNMP v1/v2c. Besides the usual status and network items it **tracks toner replacement and shows how many pages each toner cartridge printed**.

| | |
|---|---|
| Template name | `HP LaserJet Pro M404dw SNMP` |
| File | `zbx_template_hp_laserjet_pro_m404dw_snmp.yaml` |
| Zabbix | 7.0 or newer (export format `7.0`) |
| Monitoring method | SNMP v1/v2c agent items plus ICMP ping |

All OIDs come from a real `snmpwalk` of an M404dw (Printer-MIB, Host-Resources-MIB, IF-MIB and a few HP private OIDs).

## What it monitors

| Group | Items |
|---|---|
| Availability | ICMP ping, packet loss, response time, SNMP data availability |
| General | Device name, model, serial number, device uptime, SNMP agent uptime |
| Status | Printer status (idle, printing, warmup), HP status text, display message, top cover status |
| Toner | Description, cartridge model and ID, max capacity, **toner level (%)**, toner used (%) |
| Pages | **Total pages printed**, **pages printed on current toner**, pages printed in the last 24 hours, pages printed in the last 30 days |
| Memory | Total, used, utilization % |
| Network (eth0) | Operational status, speed, bits received/sent, inbound/outbound errors, inbound discards |

## Triggers

| Trigger | Severity | Condition |
|---|---|---|
| Ping failed | Average | No ping response for 3 checks |
| Ping packet loss | Warning | Loss above `{$HP.ICMP.LOSS.WARN}` for 5 minutes |
| SNMP data not received | Average | No printer status data for 15 minutes |
| Toner low | Warning | Toner above 0% and at or below `{$HP.TONER.LOW}` |
| Toner empty | High | Toner level is 0% |
| **Toner cartridge replaced** | Info | Toner level jumped up by more than `{$HP.TONER.JUMP}` points. The problem name shows the pages the previous cartridge printed. |
| Top cover open | Warning | Cover status is door open or interlock open |
| eth0 down | Average | eth0 operational status is not `up` |
| Printer rebooted | Info | Device uptime below 10 minutes |
| High memory usage | Warning | Memory utilization above `{$HP.MEM.MAX}` for 15 minutes |

## Toner tracking: pages printed per cartridge

The printer only reports a lifetime page counter (`Total pages printed`) and the toner level. The template combines them:

- **Pages printed on current toner** = `Total pages printed` minus the macro `{$HP.TONER.BASE.PAGES}`.
- When a new cartridge is installed, the toner level jumps up and the trigger **Toner cartridge replaced** fires. Its name contains the final page count of the cartridge that was just removed, for example:
  `Toner cartridge replaced (previous cartridge printed 2874 pages)`.
  The event stays in the problem history, so you keep a permanent record for every cartridge.

### One-time setup

1. Link the template and open **Latest data** for the printer.
2. If the cartridge installed right now is new, set the host macro `{$HP.TONER.BASE.PAGES}` to the current value of **Total pages printed**. If you leave it at `0`, the counter shows pages since the printer was new.

### Every time you change the toner

1. After the next poll (within about 5 minutes) a **Toner cartridge replaced** problem appears. Read the pages printed by the old cartridge in the problem name.
2. Open **Latest data** and note the current **Total pages printed**.
3. Set the host macro `{$HP.TONER.BASE.PAGES}` to that value. **Pages printed on current toner** starts again from 0.
4. Close the problem manually (*Update problem -> Close problem*).

Notes:
- The detection needs the toner level to rise by more than `{$HP.TONER.JUMP}` points (default 20). Re-seating a nearly full cartridge may not be detected. In that case use **Pages printed on current toner** directly.
- The toner level is reported in whole percent steps, so it is a coarse indicator. Use **Pages printed on current toner** to compare cartridges.

## Macros

| Macro | Default | Description |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `public` | SNMP v1/v2c community. Use it in the host SNMP interface. |
| `{$HP.TONER.LOW}` | `10` | Toner low threshold, in percent |
| `{$HP.TONER.JUMP}` | `20` | Toner level increase treated as a cartridge replacement |
| `{$HP.TONER.BASE.PAGES}` | `0` | Total pages counter value when the current cartridge was installed |
| `{$HP.MEM.MAX}` | `95` | Memory utilization threshold, in percent |
| `{$HP.ICMP.LOSS.WARN}` | `20` | Packet loss threshold, in percent |

## Installation

1. In Zabbix go to **Data collection -> Templates -> Import** and import `zbx_template_hp_laserjet_pro_m404dw_snmp.yaml`.
2. Create a host for the printer and add an **SNMP interface** (IP address of the printer, port 161, version v1 or v2c, community `{$SNMP_COMMUNITY}`).
3. Link the template **HP LaserJet Pro M404dw SNMP** to the host.
4. Set the macros above (at least `{$HP.TONER.BASE.PAGES}`) and check **Latest data** after a few minutes.

Check SNMP from the Zabbix server first:

```bash
snmpget -v1 -c COMMUNITY PRINTER_IP 1.3.6.1.2.1.43.10.2.1.4.1.1
```

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| All SNMP items *Not supported* | No SNMP interface on the host, wrong community, or SNMP disabled in the printer web page. |
| Toner level shows `-2` or `-3` | The printer cannot report an exact level (`-2` unknown, `-3` some remaining). Percent values are reported normally by this model. |
| Pages on current toner is negative | `{$HP.TONER.BASE.PAGES}` is larger than the Total pages counter. Set it to the value at the time the cartridge was installed. |
| Pages last 24h or 30d looks wrong right after import | The calculation needs history. Values become correct after 24 hours and 30 days of data. |
| Cartridge model or ID item *Not supported* | These two items use HP private OIDs and may not exist on every firmware. They are informational, disable them if needed. |

## Security notes

- Change the default SNMP community `public` on the printer and keep the real one only in the host macro. Do not commit communities, IP addresses or serial numbers to this repository.

## License

Add a license of your choice (for example MIT) as a `LICENSE` file.

