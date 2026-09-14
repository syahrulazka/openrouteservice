# OpenRouteService Local Setup

This guide shows the simplest way to run **OpenRouteService (ORS) locally** with **OpenStreetMap data** on **Linux or Windows**.


---

# 1. Requirements

Recommended:

- Linux or Windows
- Docker
- Docker Compose
- At least 8 GB RAM
- At least 10 GB free disk space

For Windows, use **Docker Desktop** with **WSL 2** enabled.

---

# 2. Linux Setup

## 2.1 Install Docker

On Ubuntu:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable --now docker
```

Check:

```bash
docker --version
docker compose version
```

Test Docker:

```bash
sudo docker run --rm hello-world
```

Optional: allow your user to run Docker without `sudo`:

```bash
sudo usermod -aG docker $USER
```

Log out and log in again after running the command above.

---

# 3. Windows Setup

## 3.1 Install Docker Desktop

Install Docker Desktop for Windows:

https://www.docker.com/products/docker-desktop/

During installation, use the **WSL 2 backend**.

Open Docker Desktop and make sure it is running.

Then open **PowerShell** and check:

```powershell
docker --version
docker compose version
```

Test:

```powershell
docker run --rm hello-world
```

---

# 4. Create the Project Folder

The following structure will be used on both Linux and Windows:

```text
openrouteservice/
├── docker-compose.yml
└── ors-docker/
    ├── config/
    ├── elevation_cache/
    ├── graphs/
    ├── files/
    └── logs/
```

## Linux

```bash
mkdir -p ~/openrouteservice
cd ~/openrouteservice

mkdir -p ors-docker/{config,elevation_cache,graphs,files,logs}
```

## Windows PowerShell

```powershell
mkdir openrouteservice
cd openrouteservice

mkdir ors-docker
mkdir ors-docker\config
mkdir ors-docker\elevation_cache
mkdir ors-docker\graphs
mkdir ors-docker\files
mkdir ors-docker\logs
```

---

# 5. Download OpenStreetMap Data

The easiest option is the **Malaysia + Singapore + Brunei** Geofabrik extract.

Source:

https://download.geofabrik.de/asia/malaysia-singapore-brunei.html

## Linux

From the project folder:

```bash
wget -O ors-docker/files/malaysia-singapore-brunei-latest.osm.pbf \
https://download.geofabrik.de/asia/malaysia-singapore-brunei-latest.osm.pbf
```

Check:

```bash
ls -lh ors-docker/files/
```

You should see:

```text
malaysia-singapore-brunei-latest.osm.pbf
```

## Windows PowerShell

From the project folder:

```powershell
Invoke-WebRequest `
  -Uri "https://download.geofabrik.de/asia/malaysia-singapore-brunei-latest.osm.pbf" `
  -OutFile "ors-docker\files\malaysia-singapore-brunei-latest.osm.pbf"
```

Check:

```powershell
Get-ChildItem ors-docker\files
```

You should see:

```text
malaysia-singapore-brunei-latest.osm.pbf
```

---

# 6. Create `docker-compose.yml`

Create this file inside:

```text
openrouteservice/
```

The final structure should be:

```text
openrouteservice/
├── docker-compose.yml
└── ors-docker/
```

Use this configuration:

```yaml
services:
  ors-app:
    image: openrouteservice/openrouteservice:latest
    container_name: ors-app

    ports:
      - "8080:8082"

    volumes:
      - ./ors-docker:/home/ors

    environment:
      REBUILD_GRAPHS: "TRUE"
      CONTAINER_LOG_LEVEL: INFO

      XMS: 4g
      XMX: 10g

      ADDITIONAL_JAVA_OPTS: ""

    restart: unless-stopped
```

## Linux

```bash
nano docker-compose.yml
```

Paste the configuration, save, and exit.

## Windows

You can create `docker-compose.yml` using:

- VS Code
- Notepad
- Notepad++

The filename must be exactly:

```text
docker-compose.yml
```

Not:

```text
docker-compose.yml.txt
```

---

# 7. Start OpenRouteService

## Linux

From the project folder:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f ors-app
```

## Windows PowerShell

From the project folder:

```powershell
docker compose up -d
```

Check:

```powershell
docker compose ps
```

View logs:

```powershell
docker compose logs -f ors-app
```

---

# 8. Wait for the Routing Graph to Build

On the first run, ORS needs to process the OSM data and build the routing graph.

You may see messages related to:

```text
Importing OSM data
Building graph
Creating graph
Loading graph
```

This first run can take some time.

Do not stop the container while the graph is being built.

---

# 9. Check ORS Status

Open another terminal.

## Linux

```bash
curl http://localhost:8080/ors/v2/health
```

## Windows PowerShell

```powershell
curl.exe http://localhost:8080/ors/v2/health
```

You are looking for:

```json
{
  "status": "ready"
}
```

Once you see:

```json
{"status":"ready"}
```

the local ORS server is ready.

---

# 10. Final Result

Your setup is now:

```text
   OpenStreetMap
     ↓
   Malaysia OSM PBF
     ↓
   OpenRouteService
     ↓
   Docker
     ↓
   localhost:8080
```

Main ORS URL:

```text
http://localhost:8080
```

API base:

```text
http://localhost:8080/ors/v2/
```

Health check:

```text
http://localhost:8080/ors/v2/health
```

Expected:

```json
{
  "status": "ready"
}
```

---

# 11. Important: Do Not Rebuild Every Time

After ORS successfully becomes:

```json
{"status":"ready"}
```

change file "docker-compose.yml"  
from:

```yaml
REBUILD_GRAPHS: "True"
```

to:

```yaml
REBUILD_GRAPHS: "False"
```

Then restart.

## Linux

```bash
docker compose down
docker compose up -d
```

## Windows

```powershell
docker compose down
docker compose up -d
```

The graph is stored in:

```text
ors-docker/graphs/
```

So ORS can reuse it instead of rebuilding everything.

---

# 12. Useful Commands

## Start

Linux / Windows:

```bash
docker compose up -d
```

## Stop

```bash
docker compose stop
```

## Restart

```bash
docker compose restart
```

## Check container

```bash
docker compose ps
```

## View logs

```bash
docker compose logs -f ors-app
```

## Check health

Linux:

```bash
curl http://localhost:8080/ors/v2/health
```

Windows:

```powershell
curl.exe http://localhost:8080/ors/v2/health
```

---

# 13. Troubleshooting

## ORS is not ready

Check logs:

```bash
docker compose logs --tail=200 ors-app
```

Look for errors related to:

```text
memory
OSM file
graph
Java
```

---

## Out of memory

If you see:

```text
OutOfMemory
```

or:

```text
Java heap space
```

reduce the heap size.

For an 8 GB machine:

```yaml
XMS: 1g
XMX: 4g
```

For a 16 GB machine:

```yaml
XMS: 2g
XMX: 8g
```

---

## OSM file not found

Check the host:

### Linux

```bash
ls -lh ors-docker/files/
```

### Windows

```powershell
Get-ChildItem ors-docker\files
```

Then check inside the container:

```bash
docker exec ors-malaysia ls -lh /home/ors/files/
```

You should see:

```text
malaysia-singapore-brunei-latest.osm.pbf
```

---

# 14. Windows Note

If Docker reports a problem with the mounted folders, make sure:

1. Docker Desktop is running.
2. You are running the commands from the project folder.
3. Docker Desktop has access to the drive containing the project.
4. WSL 2 is enabled.

Using a project directory inside your Windows user folder, such as:

```text
C:\Users\YourName\openrouteservice-malaysia
```

is usually the easiest option.

---

# 15. Official Sources

OpenRouteService:

https://openrouteservice.org/

OpenRouteService Docker:

https://giscience.github.io/openrouteservice/run-instance/running-with-docker

OpenRouteService GitHub:

https://github.com/GIScience/openrouteservice

Geofabrik Malaysia:

https://download.geofabrik.de/asia/malaysia-singapore-brunei.html
