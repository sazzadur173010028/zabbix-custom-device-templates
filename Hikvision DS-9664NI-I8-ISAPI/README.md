# Hikvision NVR DS-9664NI-I8 — Zabbix ISAPI Template

A custom Zabbix template for monitoring the Hikvision DS-9664NI-I8 Network Video Recorder (NVR) through its built-in ISAPI web interface.

The template uses HTTP agent requests with HTTP Digest authentication to collect device information, system status, HDD information, and camera channel status. Dependent items parse the collected responses, reducing the number of direct requests sent to the NVR.

**Tested device:** Hikvision DS-9664NI-I8
**Tested firmware:** V4.61.030
**Tested Zabbix version:** 7.4.7
**Minimum Zabbix version:** 7.0+

## Table of Contents

* [Features](#features)
* [Monitoring Details](#monitoring-details)
* [Triggers and Alerts](#triggers-and-alerts)
* [User Macros](#user-macros)
* [ISAPI Endpoints](#isapi-endpoints)
* [Requirements](#requirements)
* [Installation](#installation)
* [Verify ISAPI Connectivity](#verify-isapi-connectivity)
* [Troubleshooting](#troubleshooting)
* [Different NVR Models or Firmware](#different-nvr-models-or-firmware)
* [Security Recommendations](#security-recommendations)
* [Repository Contents](#repository-contents)
* [Contributing](#contributing)
* [License](#license)

## Features

* NVR reachability monitoring
* Device model, serial number, and firmware information
* CPU utilization, memory usage, and uptime
* HDD discovery and per-disk monitoring
* Camera channel discovery and status monitoring
* Online and offline channel summaries
* Camera offline IP list in operational data
* Configurable CPU utilization and minimum HDD count thresholds
* Optional camera password strength alerts

No Zabbix agent or SNMP configuration is required on the NVR. The template uses ISAPI over HTTP or HTTPS.

## Monitoring Details

### Device Information and System Status

| Item             | Description                     |
| ---------------- | ------------------------------- |
| ICMP ping        | NVR network reachability        |
| Model            | Device model reported by ISAPI  |
| Serial number    | NVR serial number               |
| Firmware version | Installed firmware version      |
| CPU utilization  | CPU usage percentage            |
| Memory used      | Memory usage converted to bytes |
| Uptime           | System uptime in seconds        |

### Storage Monitoring — Low-Level Discovery

The template discovers HDDs reported by the NVR and creates monitoring items for each discovered disk.

| Item            | Description                                               |
| --------------- | --------------------------------------------------------- |
| HDD status      | Reports disk status; a non-`ok` value indicates a problem |
| HDD capacity    | Total capacity in bytes                                   |
| HDD free space  | Available space in bytes                                  |
| HDD total count | Number of HDDs reported by the NVR                        |

### Camera and Channel Monitoring — Low-Level Discovery

The template discovers camera channels reported by the NVR.

| Item                     | Description                                                              |
| ------------------------ | ------------------------------------------------------------------------ |
| Channel online           | `1` = online, `0` = offline                                              |
| Channel detection result | Raw status reported by the NVR, such as `connect` or `netUnreachable`    |
| Channel password status  | Password strength status reported by the NVR, such as `weak` or `strong` |

### Summary Items

| Item                   | Description                                                                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------- |
| Total channels         | Total discovered or reported channels                                                          |
| Online channels        | Number of online channels                                                                      |
| Offline channels       | Number of offline channels                                                                     |
| Offline camera IP list | Text list of offline channel identifiers and associated IP addresses, when provided by the NVR |

## Triggers and Alerts

The template includes the following triggers according to its configuration.

| Trigger                 | Severity    | Condition                                                   |
| ----------------------- | ----------- | ----------------------------------------------------------- |
| NVR ping failed         | Disaster    | No ping response for three consecutive checks               |
| ISAPI data not received | High        | No data from the storage endpoint for 15 minutes            |
| HDD problem             | High        | Discovered HDD status is not `ok`                           |
| HDD count dropped       | High        | Reported HDD count is below `{$NVR.HDD.MIN}`                |
| Cameras offline         | Average     | At least one camera is offline for three consecutive checks |
| Camera offline          | Warning     | Individual channel is offline for three consecutive checks  |
| High CPU usage          | Warning     | CPU exceeds `{$NVR.CPU.MAX}` for 10 minutes                 |
| Camera password weak    | Information | Channel password status is `weak`; disabled by default      |

Trigger expressions and severity levels are defined in the imported template. Confirm the actual expressions in Zabbix before making production changes.

## User Macros

Configure the following macros on the NVR host.

| Macro             | Default | Description                                                           |
| ----------------- | ------- | --------------------------------------------------------------------- |
| `{$NVR.IP}`       | Empty   | NVR IP address; required                                              |
| `{$NVR.SCHEME}`   | `http`  | Web protocol: `http` or `https`                                       |
| `{$NVR.PORT}`     | `80`    | NVR web service port                                                  |
| `{$NVR.USER}`     | `admin` | NVR username; a dedicated read-only account is recommended            |
| `{$NVR.PASSWORD}` | Empty   | NVR password; required                                                |
| `{$NVR.CPU.MAX}`  | `90`    | CPU utilization threshold in percent                                  |
| `{$NVR.HDD.MIN}`  | `5`     | Minimum expected HDD count; set this to the number of installed disks |

The defaults above describe the documented template configuration. Confirm them against the exported file if you modify the template.

## ISAPI Endpoints

The template uses the following ISAPI endpoints.

| Endpoint                                        | Check interval | Purpose                                   |
| ----------------------------------------------- | -------------- | ----------------------------------------- |
| `/ISAPI/ContentMgmt/Storage`                    | 5 minutes      | HDD discovery and storage monitoring      |
| `/ISAPI/ContentMgmt/InputProxy/channels/status` | 1 minute       | Camera channel discovery and status       |
| `/ISAPI/System/deviceInfo`                      | 1 hour         | Device model, serial number, and firmware |
| `/ISAPI/System/status`                          | 1 minute       | CPU, memory, and uptime                   |

The XML responses are converted using the XML-to-JSON preprocessing step and parsed with JSONPath. Raw response items are configured without history storage to avoid retaining unnecessary raw data.

## Requirements

* Zabbix 7.0 or newer; tested on Zabbix 7.4.7
* A compatible Hikvision NVR with the required ISAPI endpoints
* Network connectivity from the Zabbix server or proxy to the NVR web service
* ISAPI access enabled on the NVR
* A user account with permission to read device and system status
* HTTP Digest authentication support and correct credentials

Firmware implementations can differ between NVR models and releases. Compatibility with other models has not been established solely by testing this device.

## Installation

### 1. Import the Template

1. Download `zbx_template_hikvision_nvr_ds9664ni_i8.yaml` from this repository.
2. Sign in to the Zabbix web interface.
3. Navigate to **Data collection → Templates**.
4. Click **Import** and select the downloaded file.
5. Review the import options and complete the import.

When updating an existing template, select the appropriate **Update existing** options for templates, items, triggers, and discovery rules, as applicable.

### 2. Create the NVR Host

1. Navigate to **Data collection → Hosts**.
2. Create a host for the Hikvision NVR.
3. Link the **Hikvision NVR DS-9664NI-I8 ISAPI** template.
4. Configure the host interface required by the ICMP ping item. If your actual template uses a different ping implementation, follow that item's configuration.

A Zabbix agent installed on the NVR is not required.

### 3. Configure Host Macros

Open the host's **Macros** section and configure:

* `{$NVR.IP}` — NVR IP address
* `{$NVR.USER}` — NVR username
* `{$NVR.PASSWORD}` — NVR password

Adjust `{$NVR.SCHEME}`, `{$NVR.PORT}`, `{$NVR.CPU.MAX}`, and `{$NVR.HDD.MIN}` as necessary.

Store the password as a **Secret text** macro where supported. Use a dedicated read-only NVR account instead of the administrator account.

### 4. Verify Monitoring

After enabling the host:

1. Navigate to **Monitoring → Latest data**.
2. Check device information and system status.
3. Confirm HDD discovery and storage values.
4. Confirm camera channel discovery and online/offline status.
5. Review the Problems section for trigger activity.

Some discovered items may take one or two discovery cycles to appear.

## Verify ISAPI Connectivity

Before enabling monitoring, verify that the Zabbix server can reach the NVR and authenticate successfully.

Run the following command on the Zabbix server, replacing `NVR_IP` and `USER` with the actual NVR IP address and username.

```bash
curl --digest -u USER \
  -s -o /dev/null -w "%{http_code}\n" \
  http://NVR_IP/ISAPI/ContentMgmt/Storage
```

Curl prompts for the password when only the username is supplied. An HTTP status of `200` indicates that the endpoint returned a successful response.

For HTTPS, replace `http://` with `https://` and specify the appropriate port if required. Use a valid certificate configuration for production environments.

Do not put passwords directly into commands that may be saved in shell history.

Repeated authentication failures can cause account lockouts on some NVR configurations. Verify the account and permissions before repeatedly retrying.

## Troubleshooting

| Symptom                                    | Possible cause and recommended action                                                                                                                      |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `401 Unauthorized`                         | Check the username, password, account permissions, lockout status, and authentication method. The template uses Digest authentication.                     |
| Curl reports HTTP code `000`               | No HTTP response was received. Check the NVR IP, routing, firewall rules, web port, and HTTP/HTTPS configuration.                                          |
| `Could not resolve host: *UNKNOWN*`        | A required macro may be unset, or the item is being tested outside the correct host context. Check `{$NVR.IP}` and test the item on the configured host.   |
| CPU, memory, or uptime item is unsupported | Compare the actual `/ISAPI/System/status` response with the item's JSONPath and preprocessing configuration. Firmware field names may differ.              |
| HDD free space always reports `0`          | Some NVR recording configurations may report zero free space during overwrite recording. Verify the raw endpoint response before treating this as a fault. |
| All cameras appear offline                 | First check NVR reachability, ICMP monitoring, and ISAPI response availability. Then verify the channel-status response.                                   |
| HDD count trigger reports a problem        | Verify the actual installed disk count and configure `{$NVR.HDD.MIN}` accordingly.                                                                         |

## Different NVR Models or Firmware

Other Hikvision models may expose similar ISAPI endpoints, but compatibility must be verified on each target model and firmware version.

To investigate compatibility, compare the endpoint responses from the target NVR.

```bash
curl --digest -u USER \
  http://NVR_IP/ISAPI/System/status
```

```bash
curl --digest -u USER \
  http://NVR_IP/ISAPI/ContentMgmt/Storage
```

Also verify the camera channel-status endpoint:

```bash
curl --digest -u USER \
  http://NVR_IP/ISAPI/ContentMgmt/InputProxy/channels/status
```

Review the returned fields and compare them with the template's preprocessing and JSONPath expressions.

If you clone an existing Zabbix host, verify the IP, port, scheme, username, HDD threshold, and other macros. Secret macro values may need to be entered again.

## Security Recommendations

* Use a dedicated read-only NVR account for monitoring.
* Store passwords using Zabbix secret text macros where supported.
* Prefer HTTPS with proper certificate validation when available.
* Restrict network access to the NVR management interface.
* Do not publish credentials, authentication headers, private IP details that should remain confidential, or real SNMP communities.
* Review the camera password-strength trigger before enabling it; it is disabled by default.
* Avoid repeatedly testing invalid credentials because the NVR may temporarily lock the account.

## Repository Contents

```text
.
├── README.md
└── zbx_template_hikvision_nvr_ds9664ni_i8.yaml
```

## Contributing

Issues, feedback, and pull requests are welcome.

When reporting a problem, include:

* NVR model
* Firmware version
* Zabbix version
* A description of the issue
* Relevant sanitized endpoint output or screenshots

Never include passwords, API credentials, or other sensitive information in issue reports.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

