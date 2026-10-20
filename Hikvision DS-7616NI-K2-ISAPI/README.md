
# Zabbix Template: Hikvision NVR DS-7616NI-K2 (ISAPI)

Zabbix template for monitoring a Hikvision **DS-7616NI-K2** NVR through its built-in ISAPI web interface. No SNMP and no agent are required on the NVR. Data is collected with a few HTTP agent requests (Digest authentication). All other items are dependent items parsed from those responses, so the load on the NVR stays low.

| | |
|---|---|
| Template name | `Hikvision NVR DS-7616NI-K2 ISAPI` |
| File | `zbx_template_hikvision_nvr_ds7616ni_k2.yaml` |
| Zabbix | 7.0 or newer (export format `7.0`, tested on 7.4.7) |
| Default `{$NVR.HDD.MIN}` | `1` |

## Model notes

- Built from the real Storage, Channel status and System/status responses of a DS-7616NI-K2 (14 cameras, 1 disk).
- On this firmware the XML namespace is `http://www.hikvision.com/ver20/XMLSchema` with `version="1.0"`, and each `<hdd>` and `<InputProxyChannelStatus>` element carries its own `version` and `xmlns` attributes. The XML to JSON conversion handles this, no change is needed.
- The Storage response lists only installed disks, and a disk keeps its bay number. A single disk in the second bay is therefore reported as `hdd2`, which is normal.
- `/ISAPI/System/status` reports CPU utilization, memory and uptime on this model. The `cpuDescription` field contains a number (CPU information) and is not used.
- `/ISAPI/System/deviceInfo` was not checked on this model yet. If the model, serial or firmware items show as *Not supported*, see Troubleshooting.

## What it monitors

**Device**

| Item | Description |
|---|---|
| ICMP ping | Reachability of the NVR |
| Model, Serial number, Firmware version | From `/ISAPI/System/deviceInfo` |
| CPU utilization | In % |
| Memory used | In bytes (the NVR reports MB, converted) |
| Uptime | In seconds |

**Storage** (low-level discovery, one set per installed HDD; empty slots with status `notexist` are not discovered)

| Item | Description |
|---|---|
| HDD status | `ok` is healthy, anything else raises a problem |
| HDD capacity | In bytes |
| HDD free space | In bytes |
| HDD installed count | Number of installed HDDs (empty slots are not counted) |

**Cameras / channels** (low-level discovery, one set per channel)

| Item | Description |
|---|---|
| Channel online | 1 = online, 0 = offline |
| Channel detect result | Raw result from the NVR (for example `connect`, `netUnreachable`) |
| Channel password status | `weak` or `strong` as reported by the NVR |

**Summary items**

| Item | Description |
|---|---|
| Channels: total / online / offline | Counts |
| Channels: offline IP list | Text list such as `CH6=x.x.x.x, CH27=x.x.x.x` |

## Triggers

| Trigger | Severity | Condition |
|---|---|---|
| NVR ping failed | Disaster | No ping response for 3 consecutive checks |
| ISAPI data not received | High | No data from the Storage endpoint for 15 minutes |
| HDD problem (per HDD) | High | HDD status is not `ok` |
| HDD count dropped | High | Installed HDD count is below `{$NVR.HDD.MIN}` |
| Cameras offline | Average | At least one camera offline for 3 consecutive checks (offline IPs shown in operational data) |
| Camera offline (per channel) | Warning | Channel offline for 3 consecutive checks |
| High CPU usage | Warning | CPU above `{$NVR.CPU.MAX}` for 10 minutes |
| Camera password weak (per channel) | Info | Channel password status is `weak`. **Disabled by default**, enable it if you want to track weak passwords. |

## Macros

| Macro | Default | Description |
|---|---|---|
| `{$NVR.IP}` | *(empty)* | NVR IP address. **Required**, set it on the host. |
| `{$NVR.SCHEME}` | `http` | `http` or `https` |
| `{$NVR.PORT}` | `80` | NVR web (HTTP/HTTPS) port |
| `{$NVR.USER}` | `admin` | NVR user. A read-only user is recommended. |
| `{$NVR.PASSWORD}` | *(empty, secret)* | NVR user password. **Required**, set it on the host. |
| `{$NVR.CPU.MAX}` | `90` | CPU % threshold |
| `{$NVR.HDD.MIN}` | `1` | Minimum expected number of installed HDDs. Set it to the number of disks in your NVR. |

## ISAPI endpoints used

| Endpoint | Interval | Used for |
|---|---|---|
| `/ISAPI/ContentMgmt/Storage` | 5m | HDD discovery and items |
| `/ISAPI/ContentMgmt/InputProxy/channels/status` | 1m | Camera channel discovery and items |
| `/ISAPI/System/deviceInfo` | 1h | Model, serial, firmware |
| `/ISAPI/System/status` | 1m | CPU, memory, uptime |

The XML responses are converted with the *XML to JSON* preprocessing step, then parsed with JSONPath. Raw items do not keep history.

## Requirements

- Zabbix 7.0 or newer
- Network access from the Zabbix server (or proxy) to the NVR web port. If a firewall sits in between, allow TCP to the NVR web port (port `8000` alone is the device/SDK port and is **not** enough for ISAPI).
- ISAPI enabled on the NVR (on by default on most firmware)
- An NVR user with Digest authentication and permission to read device parameters and status. A read-only user is enough.

## Installation

1. In Zabbix go to **Data collection -> Templates -> Import** and select `zbx_template_hikvision_nvr_ds7616ni_k2.yaml`.
   When updating an existing template, tick *Update existing* for templates, items, triggers and discovery rules.
2. Create a host for the NVR. **No interface is required**, because the template uses `{$NVR.IP}` instead of the host interface.
3. Link the template **Hikvision NVR DS-7616NI-K2 ISAPI** to the host.
4. On the host, open **Macros** and set `{$NVR.IP}`, `{$NVR.USER}`, `{$NVR.PASSWORD}`. Adjust `{$NVR.PORT}`, `{$NVR.SCHEME}` and `{$NVR.HDD.MIN}` if needed.
5. Open the item *Storage: raw data* on the host, click **Test -> Get value from host** and confirm XML comes back.
6. Enable the host. After 1 to 2 minutes check **Monitoring -> Latest data**. HDD and channel items appear after the first discovery run.

## Verify before enabling the host

Run this on the Zabbix server. It must print `200`:

```bash
curl --connect-timeout 5 --digest -u 'USER:PASSWORD' -s -o /dev/null -w "%{http_code}\n" \
  http://NVR_IP/ISAPI/ContentMgmt/Storage
```

Wrong credentials make Zabbix retry again and again, and Hikvision NVRs may temporarily lock the user after repeated failures. Confirm `200` first. If the password contains special characters, put the whole `USER:PASSWORD` in single quotes.

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| `401 Unauthorized` | Wrong user or password, user locked after failed logins (wait 10 to 30 minutes), missing permission, or the NVR accepts Basic authentication only (change the item authentication to Basic). |
| curl prints `000` or times out | No TCP connection. Check IP, routing/firewall and the NVR web port (`nc -zv NVR_IP PORT`). Try `https` and port 443. Only port `8000` open means ISAPI is blocked by a firewall or the web service is off. |
| `Could not resolve host: *UNKNOWN*` | You are testing the item on the template. Test it on the host instead and make sure `{$NVR.IP}` is set on the host. |
| Item **Not supported** (CPU, memory, uptime) | Field names differ on your firmware. Compare `curl --digest -u 'USER:PASS' http://NVR_IP/ISAPI/System/status` with the JSONPath of the item and adjust. |
| Free space is always `0` | Normal on NVRs that record in overwrite mode. No trigger is created on free space for this reason. |
| All cameras offline at once | Usually the NVR itself is unreachable, check the ping trigger first. |

## Security notes

- Use a dedicated **read-only** NVR user for Zabbix, not `admin`, and keep its password in the *secret text* macro `{$NVR.PASSWORD}`.
- Never commit real IP addresses or passwords to this repository.
- The NVR reports a `weak` / `strong` password status per camera. Enable the *Camera password weak* trigger prototype if you want to track it.

## License

Add a license of your choice (for example MIT) as a `LICENSE` file.
