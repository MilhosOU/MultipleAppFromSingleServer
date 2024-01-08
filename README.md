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

```bash
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

## 4. Nginx Installation

```bash
sudo apt install nginx
```

## 5. Nginx Configuration

Modify /etc/nginx/sites-available/default:

```bash
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
