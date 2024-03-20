# prox-exporter-telegraph

## Install pve-exporter on proxmox

```bash
pveum user add pve-exporter@pve -password <password>
pveum acl modify / -user pve-exporter@pve -role PVEAuditor
useradd -s /bin/false pve-exporter
apt update
apt install -y python3-venv
python3 -m venv /opt/prometheus-pve-exporter
source /opt/prometheus-pve-exporter/bin/activate
pip install prometheus-pve-exporter
deactivate
mkdir /etc/prometheus
```

`vi /etc/prometheus/pve.yml`

```bash
default:
    user: pve-exporter@pve
    password: <password>
    verify_ssl: false
```
```bash
chown -v root:pve-exporter /etc/prometheus/pve.yml
chmod -v 640 /etc/prometheus/pve.yml
```
`vi /etc/systemd/system/prometheus-pve-exporter.service`


```bash
[Unit]
Description=Prometheus Proxmox VE Exporter
Documentation=https://github.com/prometheus-pve/prometheus-pve-exporter

[Service]
Restart=always
User=pve-exporter
ExecStart=/opt/prometheus-pve-exporter/bin/pve_exporter --config.file /etc/prometheus/pve.yml

[Install]
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload
systemctl enable prometheus-pve-exporter.service
systemctl start prometheus-pve-exporter.service
ss -lntp | grep 9221
curl --silent http://127.0.0.1:9221/pve | grep pve_version_info
```

## Install Telegraf on proxmox
```bash
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.15.2-1_amd64.deb
dpkg -i telegraf_1.15.2-1_amd64.deb
rm telegraf_1.15.2-1_amd64.deb
telegraf config > telegraf.conf
cp telegraf.conf /etc/telegraf/telegraf.conf
service telegraf restart
systemctl status telegraf
```

