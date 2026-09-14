# OpenRouteService Local Setup

This guide shows the simplest way to run **OpenRouteService (ORS) locally** with **OpenStreetMap data** on **Linux or Windows**.


---

# 1. Requirements

- Docker
- At least 8 GB RAM
- At least 10 GB free disk space

For Windows, use **Docker Desktop** with **WSL 2** enabled.

---

# 2. Install Docker (If Docker have not installed)

## 2.1 On Linux

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

## 2.2 On Windows

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

# 3. Clone Github OpenRouteService Project 

```bash
git clone https://github.com/syahrulazka/openrouteservice.git
```

The following structure will be used on your computer:

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

---

# 4. Download OpenStreetMap Data

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

on folder "ors-docker\files"

---

# 5. Start OpenRouteService

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

# 6. Wait for the Routing Graph to Build

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

# 7. Check ORS Status

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

# 8. Final Result

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

# 9. Important: Do Not Rebuild Every Time

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

# 10. Useful Commands

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

## Test API

```bash
curl -X POST "http://localhost:8080/ors/v2/directions/driving-car" \
  -H "Content-Type: application/json" \
  -d '{
    "coordinates": [
      [101.6869, 3.1390],
      [101.6151, 3.0738]
    ]
  }'
```
---