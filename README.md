# Traccar Server Installation on Oracle VPS with Docker

A complete step-by-step guide to install and run Traccar GPS tracking server on Oracle Cloud's free VPS using Docker.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Accessing Traccar](#accessing-traccar)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

## Prerequisites

- Oracle Cloud free VPS (Ubuntu 22.04 or similar)
- SSH access to your VPS
- Basic Linux command line knowledge

## Installation Steps

### Step 1: Verify Docker Installation

Check if Docker is already installed:

```bash
sudo apt update && sudo apt upgrade -y

docker --version

```

If the command is not found, proceed to Step 2. Otherwise, skip to Step 3.

### Step 2: Install Docker

Install Docker on your Ubuntu system:

```bash
sudo apt install docker.io -y
```

This command will:
- Download Docker from Ubuntu's repository
- Install Docker on your system
- Take 1-2 minutes depending on internet speed

### Step 3: Start Docker Service

Start the Docker daemon:

```bash
sudo systemctl start docker
```

### Step 4: Enable Docker on Boot

Enable Docker to automatically start when your VPS reboots:

```bash
sudo systemctl enable docker
```

### Step 5: Install Docker Compose

Install Docker Compose for managing multi-container applications:

```bash
sudo apt install docker-compose -y
```

### Step 6: Create Traccar Directory

Create a directory to store Traccar data and configuration:

```bash
mkdir -p ~/traccar
```

### Step 7: Create Docker Compose Configuration File

Navigate to the traccar directory and create the Docker Compose file:

```bash
cd ~/traccar
nano docker-compose.yml
```

Copy and paste the following configuration into the file:

```yaml
version: '3.3'

services:
  traccar:
    image: traccar/traccar:latest
    container_name: traccar
    ports:
      - "8082:8082"
      - "5055:5055/udp"
      - "5055:5055/tcp"
    volumes:
      - ~/traccar/data:/opt/traccar/data
    environment:
      - TRACCAR_DB_URL=jdbc:h2:./data/database
    restart: unless-stopped
```

**To save the file:**
1. Press `Ctrl + X`
2. Type `Y` to confirm saving
3. Press `Enter` to keep the filename

### Step 8: Start Traccar Container

Start the Traccar server using Docker Compose:

```bash
sudo docker-compose up -d
```

**Expected output:**
- `Pulling traccar/traccar:latest`
- `Creating traccar`
- `Started`

This process takes 1-2 minutes as it downloads the Traccar image.

### Step 9: Verify Traccar is Running

Confirm that the container started successfully:

```bash
sudo docker ps
```

You should see a table with the `traccar` container listed with status `Up X seconds`.

**Example output:**
```
CONTAINER ID   IMAGE                    COMMAND      STATUS          PORTS
d869e1949951   traccar/traccar:latest   "/opt/..."   Up 14 seconds   0.0.0.0:8082->8082/tcp
```

## Accessing Traccar

### Find Your Public IP Address

Get your VPS public IP address:

```bash
curl ifconfig.me
```

This will return your public IP (e.g., `129.159.235.74`).

### Open Traccar Web Interface

Open your web browser and navigate to:

```
http://<YOUR_PUBLIC_IP>:8082
```

Example:
```
http://129.159.235.74:8082
```

### Log In with Default Credentials

- **Username:** `admin`
- **Password:** `admin`

## Security

### ⚠️ Important: Change Default Password

After logging in, immediately change the default admin password:

1. Click on your **username** in the top right corner
2. Select **Settings** or **Account Settings**
3. Find the **Password** field
4. Enter a **strong, unique password**
5. Click **Save** or **Update**

### Configure Oracle VPS Firewall

To access Traccar from the internet, you must open ports in Oracle Cloud's security list:

**Steps in Oracle Cloud Console:**

1. Go to your Oracle Cloud dashboard
2. Find your VPS instance (`traccar-vnic`)
3. Click on **Attached VNICs**
4. Click on the security list
5. Click **Add Ingress Rule** and add these rules:

**Rule 1 - Web Interface (TCP)**
- Protocol: TCP
- Source CIDR: 0.0.0.0/0
- Destination Port: 8082

**Rule 2 - GPS Devices (TCP)**
- Protocol: TCP
- Source CIDR: 0.0.0.0/0
- Destination Port: 5055

**Rule 3 - GPS Devices (UDP)**
- Protocol: UDP
- Source CIDR: 0.0.0.0/0
- Destination Port: 5055

Click **Add Ingress Rule** after each rule.

## Troubleshooting

### Port Already in Use (Error: bind: address already in use)

If you get an error that port 8082 is already in use:

**Find the process using the port:**
```bash
sudo lsof -i :8082
```

**Stop the process:**
```bash
sudo kill <PID>
```

Replace `<PID>` with the process ID from the output above.

**Then restart Traccar:**
```bash
sudo docker-compose up -d
```

### Container Not Starting

Check the Docker logs:

```bash
sudo docker logs traccar
```

This will show you any error messages.

### Cannot Access Web Interface

Make sure:
1. Docker container is running: `sudo docker ps`
2. Firewall rules are added in Oracle Cloud Console
3. You're using the correct IP address and port: `http://<IP>:8082`
4. Wait a few seconds after starting the container for it to fully initialize

### Check Container Status

View real-time container logs:

```bash
sudo docker logs -f traccar
```

Press `Ctrl + C` to exit.

## Docker Commands Reference

**View running containers:**
```bash
sudo docker ps
```

**View all containers (including stopped):**
```bash
sudo docker ps -a
```

**Stop Traccar:**
```bash
sudo docker-compose down
```

**Start Traccar:**
```bash
sudo docker-compose up -d
```

**Restart Traccar:**
```bash
sudo docker-compose restart
```

**View container logs:**
```bash
sudo docker logs traccar
```

**Remove Traccar container:**
```bash
sudo docker-compose rm
```

## Next Steps

After successful installation:

1. **Configure GPS Devices** - Add your GPS trackers to Traccar
2. **Set Up SSL/HTTPS** - Secure your installation with a valid certificate
3. **Configure Domain** - Point a domain name to your VPS
4. **Mobile App Setup** - Install and configure Traccar client on mobile devices
5. **Backup Data** - Set up regular backups of your Traccar database
6. **User Management** - Create additional user accounts for team members

## File Structure

```
~/traccar/
├── docker-compose.yml    # Docker configuration file
└── data/                 # Traccar database and data (auto-created)
    └── database          # H2 database files
```

## Ports Used

| Port | Protocol | Purpose |
|------|----------|---------|
| 8082 | TCP | Web Interface (HTTP) |
| 5055 | TCP | GPS Device Communication |
| 5055 | UDP | GPS Device Communication |

## Support & Resources

- **Traccar Official Website:** https://www.traccar.org/
- **Traccar Documentation:** https://www.traccar.org/documentation/
- **Traccar GitHub:** https://github.com/traccar/traccar
- **Docker Documentation:** https://docs.docker.com/

## License

This guide is provided as-is for educational purposes. Traccar is open-source software licensed under the Apache License 2.0.

## Version Info

- **Created:** May 2026
- **OS:** Ubuntu 22.04 LTS
- **Docker:** 20.10+
- **Docker Compose:** 1.29+
- **Traccar:** Latest (pulled from traccar/traccar:latest)

---

**Happy tracking! 🚀**
