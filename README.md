# 📡 NMS DWDM HT6000 — Optical Transport Network Telemetry

Sistem **Network Management & Optical Telemetry** berbasis Docker untuk memantau perangkat transmisi serat optik **DWDM HT6000** secara *real-time* menggunakan **SNMP Exporter, Prometheus, dan Grafana**.

Proyek ini dirancang untuk kebutuhan operasional **24/7 Network Operations Center (NOC)** dan **Core/Transport Network Engineering**, dilengkapi dengan *SNMP Agent Simulator* bawaan untuk pengujian laboratorium.

---

## 📊 Tampilan Dashboard Grafana

![Dashboard NMS DWDM HT6000](dashboard.png)

---

## 🏗️ Arsitektur Sistem

    [DWDM HT6000 Chassis / Simulator: 400G OTU, EDFA, OLP, NMS, FAN]
                                  |
                                  | SNMP v2c (UDP 161) - OID: 1.3.6.1.4.1.83667
                                  v
                  [Prometheus SNMP Exporter (:9116)]
                                  |
                                  | HTTP Scrape (15s) + Optical Scaling
                                  v
                       [Prometheus TSDB (:9090)]
                                  |
                                  | PromQL
                                  v
                      [Grafana Dashboard (:3000)]

---

## 🔍 Modul & Parameter Optik yang Dipantau

| Modul Kartu | Base OID | Parameter Kritis yang Dipantau | Konversi Skala |
| :--- | :--- | :--- | :--- |
| **400G OTU** *(Slot 1)* | `...83667.1.1.4.16.1.*` | CFP2 Tx/Rx Power, OSNR, ESNR, Chromatic Dispersion (CD), DGD, FEC Errors, QSFP28 Client Power & Temp | Power/Temp: `x 0.01`, OSNR/ESNR: `x 0.1` |
| **EDFA Amplifier** *(Slot 2)* | `...83667.1.1.4.6.2.*` | Input Power (`In1`, `In2`), Output Power (`Out1`, `Out2`), Optical Gain (`Gain1`, `Gain2`), Pump Current | Power/Gain: `x 0.01` (`dBm` / `dB`) |
| **OLP Protection** *(Slot 3)* | `...83667.1.1.4.3.3.*` | Primary (`R1`/`T1`) & Secondary (`R2`/`T2`) Optical Power, Common `RX`/`TX`, Active Working Route | Power/Temp: `x 0.01` (`dBm` / `°C`) |
| **NMS & Power** | `...83667.1.1.3.2.*` | Redundansi Power Supply (`PSU 1` & `PSU 2`), Suhu Board NMS, Status LAN, SFP Management, System Uptime | Power/Temp: `x 0.01` (`dBm` / `°C`) |
| **FAN Tray** *(Slot 32)* | `...83667.1.1.4.23.32.*` | Status Kipas (`Fan 1` – `Fan 8`), Kecepatan Putaran Aktual (`RPM`), Duty Cycle (`%`) | Kecepatan: `1:1` (`RPM`) |

---

## 📂 Struktur Direktori

- `docker-compose.yml` — Orkestrasi container Prometheus, SNMP Exporter, Simulator HT6000, dan Grafana
- `prometheus/prometheus.yml` — Konfigurasi target scraping dan relabeling SNMP
- `snmp-exporter/snmp.yml` — Pemetaan OID Enterprise HT6000 dan konversi skala metrik optik
- `simulator/data/public.snmprec` — Data simulasi OID perangkat DWDM HT6000
- `grafana/dashboards/ht6000-overview.json` — Dashboard NOC Optical Telemetry siap pakai
- `grafana/provisioning/` — Konfigurasi auto-provisioning Data Source dan Dashboard Grafana

---

## 🚀 Cara Menjalankan Proyek

1. Jalankan seluruh layanan dengan Docker Compose:
   `sudo docker compose up -d`
2. Verifikasi metrik SNMP Exporter:
   `curl -s "http://localhost:9116/snmp?target=ht6000-simulator:161&module=ht6000&auth=public_v2" | grep ht6000_`
3. Buka Grafana di browser: `http://localhost:3000` (Menu **Dashboards -> DWDM Monitoring -> NMS DWDM HT6000 - NOC Optical Telemetry**)
4. Buka Prometheus di browser: `http://localhost:9090`
