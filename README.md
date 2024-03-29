# Multi-Engine Golang Applications with Nginx Proxy

This README provides a streamlined guide for setting up dual Golang applications on an Ubuntu server. It focuses on running multiple engines from a singular server, each accessible via unique endpoints through Nginx reverse proxy.

## Prerequisites

- Ubuntu Server
- Domain Names (e.g., api1.yourdomain.com, api2.yourdomain.com) linked to your server's IP

## 1. Install Golang

```bash
sudo apt update
sudo apt install golang-go
```

## 2. Develop Two Golang Applications

For app1/main.go:

```golang
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from App 1")
    })

    http.ListenAndServe(":8080", nil)
}
```

For app2/main.go:

```golang
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from App 2")
    })

    http.ListenAndServe(":8081", nil)
}
```

## 3. Systemd Service Configuration

For app1, create /etc/systemd/system/app1.service:

```ini
[Unit]
Description=Golang App 1

[Service]
ExecStart=/usr/bin/go run /root/app1/main.go
Restart=always
User=root
Group=root
WorkingDirectory=/root/app1

[Install]
WantedBy=multi-user.target
```

Activate and initiate the service:

```bash
sudo systemctl enable app1.service
sudo systemctl start app1.service
```

Repeat the process for app2.

After modifying a systemd service file, you need to reload the systemd daemon to apply the changes.:

```bash
sudo systemctl daemon-reload
sudo systemctl restart app1
```

## 4. Nginx Installation

```bash
sudo apt install nginx
```

## 5. Nginx Configuration

Modify /etc/nginx/sites-available/default:

```nginx
server {
    listen 80;
    server_name api1.yourdomain.com;
    location / {
        proxy_pass http://localhost:8080;
    }
}

server {
    listen 80;
    server_name api2.yourdomain.com;
    location / {
        proxy_pass http://localhost:8081;
    }
}
```

Validate and refresh Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## 6. Setting up DNS Records on Cloudflare

Add a records, as shown in the table below

| Type | Name | Content     |
|------|------|-------------|
| A    | api1 | ip_address  |
| A    | api2 | ip_address  |

## 7. Enabling HTTPS for your applications

Install Certbot:

```bash
sudo apt update && sudo apt install certbot python3-certbot-nginx.
```

Obtain certificates:

```bash
sudo certbot --nginx -d api1.yourdomain.com
sudo certbot --nginx -d api2.yourdomain.com
```

Manual Renewal:

```bash
sudo certbot renew
```

List all certificates:

```bash
sudo certbot certificates
```

## 8. Monitor the logs

To view the logs for your specific systemctl (systemd) service:

```bash
journalctl -u app1.service
```

Follow the Logs: Add -f to tail the logs (see new logs in real-time):

```bash
journalctl -fu app1.service
```

Show Logs for the Current Boot: Use -b:

```bash
journalctl -u app1.service -b
```

Show Logs Since a Specific Time: Use --since:

```bash
journalctl -u app1.service --since "2024-01-09 12:00:00"
```

Show a Specific Number of Lines: Use -n followed by the number of lines:

```bash
journalctl -u app1.service -n 100
```
