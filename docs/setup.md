# Setup Overview

This page describes the components and configuration approach at a high level. Actual config files are maintained privately.

## Prerequisites

| Component | Version |
|---|---|
| Grafana | 12.x |
| VictoriaMetrics | 1.x single-node |
| Telegraf | 1.30+ |
| Python | 3.10+ |
| MySQL | 8.x |

Python dependencies: `netmiko`, `pymysql`

## VictoriaMetrics

Single-node deployment, retention set to 12 months. Telegraf writes via the InfluxDB line protocol endpoint (`/write`) on port 8428. Grafana queries via the Prometheus-compatible endpoint on the same port.

## Telegraf

Seven separate `systemd` service instances, each with its own config directory. Instances 1–4 have their agent lists regenerated nightly; instances 5–7 use static configs.

SNMPv3 with `authPriv` security level (SHA auth, AES priv) is used for all device polling.

## Grafana datasource

The datasource uid used across all dashboards is `DS_VICTORIAMETRICS`. Set this uid when provisioning the datasource to avoid having to edit every dashboard JSON.

## Automation cron schedule

The pipeline runs 4 times a day to keep the device inventory in sync with live ISIS topology.

```
01:03  13:03  collect ISIS LSDB from AGG/CORE routers → parse → update MySQL
07:03  19:03

02:03  14:03  regenerate CSR batch configs → restart instances 1–4
08:03  20:03
```

## MySQL schema

```sql
CREATE TABLE isisips (
  id       INT AUTO_INCREMENT PRIMARY KEY,
  ip       VARCHAR(15)  NOT NULL,
  hostname VARCHAR(100) NOT NULL UNIQUE,
  category ENUM('AGG','CORE','BE','CSR') NOT NULL,
  updated  TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
