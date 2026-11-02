## 6. Docker Networking
 
### 6.1 Network Drivers
 
```mermaid
flowchart TD
    NET["Docker Networking"] --> BR["bridge (default)<br/>private network on one host"]
    NET --> HO["host<br/>shares the host network, no isolation"]
    NET --> NO["none<br/>no network at all"]
    NET --> OV["overlay<br/>multi-host networking (Swarm)"]
    NET --> MV["macvlan<br/>container gets its own MAC / LAN IP"]
```
 
| Driver | Description | Use case |
|---|---|---|
| `bridge` | Default. Containers get private IPs behind the host | Most single-host apps |
| `host` | Container uses the host's network directly | Max network performance |
| `none` | Fully isolated, no networking | Security / batch jobs |
| `overlay` | Spans multiple Docker hosts | Swarm / clusters |
| `macvlan` | Container appears as a physical device on the LAN | Legacy apps needing a LAN IP |
 
### 6.2 Bridge Network and Port Mapping
 
```mermaid
flowchart LR
    USER["Browser / Client"] -->|"http://host:8080"| HOSTPORT["Host port 8080"]
 
    subgraph DockerHost["Docker Host"]
        HOSTPORT -->|"port mapping -p 8080:80"| BRIDGE["docker0 / bridge network<br/>172.17.0.0/16"]
        BRIDGE --> C1["web container<br/>172.17.0.2 :80"]
        BRIDGE --> C2["api container<br/>172.17.0.3 :3000"]
    end
```
 
```
-p 8080:80
     |    |
     |    +-- port INSIDE the container
     +------- port on the HOST machine
```
 
### 6.3 Container-to-Container Communication
 
On the **default** bridge you must use IP addresses. On a **user-defined bridge**, containers reach each other **by name** (built-in DNS).
 
```mermaid
flowchart LR
    subgraph appnet["User-defined network: appnet"]
        WEB["web"] -->|"http://api:3000"| API["api"]
        API -->|"db:5432"| DB["db (postgres)"]
    end
    DNS["Docker built-in DNS<br/>name to IP"] -.-> appnet
```
 
```bash
docker network create appnet
docker run -d --name db  --network appnet -e POSTGRES_PASSWORD=secret postgres:16
docker run -d --name api --network appnet -p 3000:3000 myapi
# inside "api", connect to host "db" on port 5432
```
 
### 6.4 Network Commands
 
| Command | What it does |
|---|---|
| `docker network ls` | List networks |
| `docker network create mynet` | Create a network |
| `docker network inspect mynet` | Show details (connected containers, IPs) |
| `docker network connect mynet web` | Attach a running container |
| `docker network disconnect mynet web` | Detach a container |
| `docker network rm mynet` | Remove a network |
| `docker port web` | Show port mappings of a container |
 
### 6.5 Isolation Between Networks
 
```mermaid
flowchart LR
    subgraph frontend["Network: frontend"]
        NGX["nginx"]
        APP["app"]
    end
    subgraph backend["Network: backend"]
        APP2["app"]
        DB["database"]
    end
    NGX --> APP
    APP2 --> DB
    NGX -. cannot reach .-> DB
```
 
Put the database only on the `backend` network, so the public-facing container cannot reach it directly.
 