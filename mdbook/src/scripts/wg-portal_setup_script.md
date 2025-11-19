Edited script to have portal not running as root TEST

```
#!/bin/bash
# Complete setup script for WireGuard, NAT, and wg-portal with systemd autostart and config
# For Debian/Ubuntu-based systems

set -e  # Exit if any command fails

echo "=== Installing required packages ==="
sudo apt update
sudo apt install -y wireguard curl netfilter-persistent iptables

echo "=== Enabling IPv4 forwarding ==="
if ! grep -q "^net.ipv4.ip_forward=1" /etc/sysctl.conf; then
    echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
else
    echo "IPv4 forwarding already enabled in /etc/sysctl.conf"
fi

sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -p

# Detect main network interface automatically
NET_IFACE=$(ip route show default | awk '/default/ {print $5}')
echo "=== Detected main interface: $NET_IFACE ==="

echo "=== Setting up NAT (iptables) ==="
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o "$NET_IFACE" -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 10.11.12.1/24 -o "$NET_IFACE" -j MASQUERADE

echo "=== Saving iptables rules ==="
sudo netfilter-persistent save

echo "=== Creating wg-portal user if not present ==="
if ! id "wg-portal" &>/dev/null; then
    sudo useradd --system --create-home --shell /usr/sbin/nologin wg-portal
    echo "User wg-portal created."
else
    echo "User wg-portal already exists."
fi

echo "=== Downloading wg-portal ==="
curl -L -o wg-portal https://github.com/h44z/wg-portal/releases/download/v2.1.0/wg-portal_linux_amd64

echo "=== Installing wg-portal ==="
sudo mkdir -p /opt/wg-portal
sudo install wg-portal /opt/wg-portal/

echo "=== Creating wg-portal config directory and config.yaml ==="
sudo mkdir -p /opt/wg-portal/config

sudo tee /opt/wg-portal/config/config.yaml >/dev/null <<'EOF'
core:
  admin_user: admin@wgportal.local
  admin_password: wgportal-default
  admin_api_token: ""
  disable_admin_user: false
  editable_keys: true
  create_default_peer: false
  create_default_peer_on_creation: false
  re_enable_peer_after_user_enable: true
  delete_peer_after_user_deleted: false
  self_provisioning_allowed: false
  import_existing: true
  restore_state: true

backend:
  default: local
  local_resolvconf_prefix: tun.

advanced:
  log_level: info
  log_pretty: false
  log_json: false
  start_listen_port: 51820
  start_cidr_v4: 10.11.12.0/24
  start_cidr_v6: fdfd:d3ad:c0de:1234::0/64
  use_ip_v6: true
  config_storage_path: ""
  expiry_check_interval: 15m
  rule_prio_offset: 20000
  route_table_offset: 20000
  api_admin_only: true
  limit_additional_user_peers: 0

database:
  debug: false
  slow_query_threshold: "0"
  type: sqlite
  dsn: data/sqlite.db
  encryption_passphrase: ""

statistics:
  use_ping_checks: true
  ping_check_workers: 10
  ping_unprivileged: false
  ping_check_interval: 1m
  data_collection_interval: 1m
  collect_interface_data: true
  collect_peer_data: true
  collect_audit_data: true
  listening_address: :8787

mail:
  host: 127.0.0.1
  port: 25
  encryption: none
  cert_validation: true
  username: ""
  password: ""
  auth_type: plain
  from: Wireguard Portal <noreply@wireguard.local>
  link_only: false
  allow_peer_email: false

auth:
  oidc: []
  oauth: []
  ldap: []
  webauthn:
    enabled: true
  min_password_length: 16
  hide_login_form: false

web:
  listening_address: :8888
  external_url: http://localhost:8888
  site_company_name: WireGuard Portal
  site_title: WireGuard Portal
  session_identifier: wgPortalSession
  session_secret: very_secret
  csrf_secret: extremely_secret
  request_logging: false
  expose_host_info: false
  cert_file: ""
  key_File: ""

webhook:
  url: ""
  authentication: ""
  timeout: 10s
EOF

echo "=== Setting ownership of /opt/wg-portal to wg-portal user ==="
sudo chown -R wg-portal:wg-portal /opt/wg-portal

echo "=== Creating systemd service for wg-portal (non-root user) ==="
cat <<EOF | sudo tee /etc/systemd/system/wg-portal.service >/dev/null
[Unit]
Description=WireGuard Portal Service
After=network.target

[Service]
ExecStart=/opt/wg-portal/wg-portal
WorkingDirectory=/opt/wg-portal
Restart=always
User=wg-portal
Group=wg-portal
Environment="WG_PORTAL_CONFIG=/opt/wg-portal/config/config.yaml"
StandardOutput=append:/var/log/wg-portal.log
StandardError=append:/var/log/wg-portal.log

[Install]
WantedBy=multi-user.target
EOF

echo "=== Creating log file and setting proper permissions ==="
sudo touch /var/log/wg-portal.log
sudo chown wg-portal:wg-portal /var/log/wg-portal.log

echo "=== Reloading systemd and enabling wg-portal service ==="
sudo systemctl daemon-reload
sudo systemctl enable wg-portal.service
sudo systemctl start wg-portal.service

echo "=== Setup Complete ==="
echo " WireGuard installed"
echo " IPv4 forwarding enabled"
echo " NAT configured and persistent"
echo " wg-portal user created"
echo " wg-portal installed with config"
echo " wg-portal running as NON-ROOT user and auto-starts on boot"
echo
echo "You can check wg-portal status with:"
echo "  sudo systemctl status wg-portal"
echo
echo "Access wg-portal web interface at: http://<your-server-ip>:8888"
echo "Login with user: admin@wgportal.local  password: wgportal-default"
echo
echo "Logs: /var/log/wg-portal.log"
