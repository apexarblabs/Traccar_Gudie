# Traccar Server Installation on Oracle VPS with Docker + Nginx + Cloudflare

A complete production setup guide to install and run Traccar GPS tracking server on Oracle Cloud (or any Ubuntu VPS) using Docker, PostgreSQL, Nginx reverse proxy, and Cloudflare DNS.

---

## Table of Contents

- Prerequisites  
- System Update  
- Install Docker  
- Run PostgreSQL Container  
- Install Traccar  
- Install Nginx  
- Configure Reverse Proxy  
- Cloudflare DNS Setup  
- Firewall and Ports  
- Accessing Traccar  
- Security  
- Troubleshooting  
- Next Steps  

---

## Prerequisites

- Oracle Cloud Free VPS (Ubuntu 20.04 / 22.04)
- SSH access to server
- Domain added in Cloudflare (gps.busvahan.in)
- Basic Linux knowledge

---

## System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Install Docker

```bash
sudo apt install docker.io -y
```

Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verify installation:

```bash
docker --version
```

---

## Run PostgreSQL Container

```bash
sudo docker run --name traccar-postgres \
-e POSTGRES_DB=traccar \
-e POSTGRES_USER=traccar \
-e POSTGRES_PASSWORD='555!ytrewQ' \
-v ~/traccar:/var/lib/postgresql \
-p 5432:5432 \
-d postgres:latest
```

---

## Install Traccar

```bash
sudo docker run -d \
--name traccar \
--restart always \
-p 8082:8082 \
traccar/traccar:latest
```

---

## Install Nginx

```bash
sudo apt install nginx -y
```

---

## Nginx Configuration

```nginx
server {
    listen 80;
    server_name gps.busvahan.in;

    location / {
        proxy_pass http://localhost:8082;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/gps.busvahan.in /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## Cloudflare DNS

Add A record:

- Name: gps  
- Value: YOUR_PUBLIC_IP  
- Proxy: DNS only  

Get IP:
```bash
curl ifconfig.me
```

---

## Access

```
http://gps.busvahan.in
```

## Edit traccar.xml in VS Code

```
Connect using Remote SSH form bottom left corner in the VS Code
Once after connection, copy file to VS Code using
docker cp traccar:/opt/traccar/conf/traccar.xml ~/traccar.xml
docker cp ~/traccar.xml traccar:/opt/traccar/conf/traccar.xml

Once done copying back restart docker
docker restart traccar
docker ps
docker logs -f traccar

http://gps.busvahan.in
```


Login:
- admin / admin

---

## Ports

- 80 → Nginx  
- 8082 → Traccar  
- 5432 → PostgreSQL  

---

## Done
