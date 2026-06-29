# 03 - Container-based Virtualization and Services Deployment

#docker #containers #virtualizzazione #DevOps #advanced-computer-programming #DockerCompose #DockerSwarm #Dockerfile #Namespace #cgroups

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Docker documentation](https://docs.docker.com/)
- [Docker Swarm](https://docs.docker.com/engine/swarm/)
- [Swarm tutorial](https://medium.com/romin-irani/docker-swarm-tutorial-b67470cf8872)
- [Esempi Docker Compose](https://github.com/docker/awesome-compose/)

---

## Concetti chiave

- **VM** vs **Container**: VM = hypervisor + guest OS (lento, pesante); Container = virtualizzazione leggera sul kernel host
- Container: nessuna emulazione hardware, usa **Namespace** e **Cgroups** del kernel Linux
- Un server fisico medio regge 10-100 VM vs **100-1000 container**
- **Namespace Linux**: isolamento di risorse per processo (mnt, pid, net, ipc, uts, user)
- **Cgroups**: isolamento di risorse per **gruppi** di processi (memory, cpuset, net_prio, devices, blkio)
- **Docker engine**: daemon (API REST), containerd (gestione ciclo di vita), runc (runtime OCI)
- **Immagine Docker**: layer di sola lettura, Union File System, OverlayFS, senza stato
- **Docker registry**: Docker Hub, registro locale, registro privato
- Comandi base: `docker build`, `docker run`, `docker ps`, `docker stop`, `docker exec`
- **Dockerfile**: `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENTRYPOINT`, `CMD`, `EXPOSE`
- **Docker Compose**: deploy multi-container tramite file `.yaml`, comandi `up`/`down`/`ps`/`logs`
- **Docker Swarm**: orchestratore cluster; nodi manager (algoritmo Raft) e nodi worker; task atomici
- Servizi Swarm: **replicati** (N repliche su N nodi scelti) vs **globali** (un task su ogni nodo)
- Comandi Swarm: `swarm init`, `swarm join`, `service create`, `stack deploy`

---

## VM vs Container

### Problemi delle Virtual Machine

- **Overhead di prestazioni** dovuto alle indirezioni software (guest OS, hypervisor, VM innestate)
- **Grande ingombro** di memoria e storage
- **Avvio lento** (ordine di minuti)
- **Costi** di licenza e manutenzione del sistema operativo guest
- E davvero necessario virtualizzare tutti i dispositivi hardware e un OS completo?

### Soluzione basata su container

Un container **non e una macchina virtuale** nel senso tradizionale:
- Nessuna emulazione di dispositivi fisici
- **Virtualizzazione leggera**
- Nessun overhead dovuto al livello di hypervisor
- Sfrutta l'astrazione dei processi e le capacita di gestione delle risorse del kernel host tramite **Namespace** e **Cgroups**

| Aspetto | VM (Type-2 Hypervisor) | Container |
|---|---|---|
| Guest OS | Si (completo) | No |
| Overhead | Alto (hypervisor + guest OS) | Basso (solo kernel host) |
| Avvio | Ordine di minuti | Ordine di secondi/ms |
| Densita su server fisico | 10-100 | 100-1000 |
| Footprint disco | GB | Pochi MB |
| Isolamento | Forte | Piu leggero (namespace) |

---

## Namespace Linux

I **Namespace** forniscono un dominio di denominazione per varie risorse del sistema, realizzando l'**isolamento per processo**.

| Namespace | Risorsa isolata |
|---|---|
| `mnt` | Mount point e filesystem |
| `pid` | Processi (PID) |
| `net` | Stack di rete |
| `ipc` | System V IPC |
| `uts` | Hostname |
| `user` | UID (User IDs) |

### System call principali

| Syscall | Comportamento |
|---|---|
| `clone()` | Crea un nuovo processo e un nuovo namespace; il processo viene "attaccato" al nuovo namespace |
| `fork()` / `exit()` | Modificati per gestire nuovi flag per i namespace (es. `CLONE_NEWxxx`) |
| `unshare()` | Non crea un nuovo processo; crea un nuovo namespace e lo "attacca" al processo corrente (aggiunta nel 2005) |
| `setns()` | Fa il join di un namespace esistente |

### Esempi di namespace

**Network namespace:** ogni namespace ha il suo stack di rete (interfacce `eth0`, `lo`); i namespace sono collegati tramite un Linux bridge e coppie `veth`.

**Mount namespace:** ogni container ha la propria vista del filesystem, indipendente dal namespace globale.

**PID namespace:** ogni container vede solo i propri processi; il PID 1 dentro il container puo essere diverso da quello dell'host.

---

## Cgroups (Control Groups)

I **Cgroups** forniscono un framework per la **gestione delle risorse** a livello di **gruppi di processi** (a differenza dei Namespace che isolano per singolo processo).

- Progetto nato da Google nel 2006 come *process container*, rinominato `cgroup` nel 2007
- I moduli sono distribuiti nell'albero del kernel in base alla funzionalita

### Moduli principali

| Modulo | Funzione | File kernel |
|---|---|---|
| `memory` | Limiti uso memoria e report automatici | `mm/memcontrol.c` |
| `cpuset` | Assegna CPU (in sistemi multicore) e nodi di memoria | `kernel/cpuset.c` |
| `net_prio` | Priorita dinamica del traffico di rete per interfaccia | `net/core/netprio_cgroup.c` |
| `devices` | Consente/nega accesso ai dispositivi | `security/device_cgroup.c` |
| `blkio` | Limiti I/O su dispositivi a blocchi (disco, SSD, USB) | - |
| `cpu`, `cpuacct`, `freezer`, `net_cls`, `ns` | Varie | - |

> Il controller della memoria (`memcg`) e il piu complesso ed e gestito separatamente.

---

## Docker

**Docker Inc.** fondata come dotCloud, Inc. nel 2010 da Solomon Hykes, rinominata nel 2013. Valutazione stimata oltre 1 miliardo di dollari.

**Docker** e un motore di container scritto in **Go**, basato su container Linux, costruito per eseguire un'applicazione in un container. Il repository GitHub (Moby) conta oltre 16.2K fork.

### Docker Engine

| Componente | Ruolo |
|---|---|
| **daemon** | API REST (ricezione istruzioni) e funzionalita aggiuntive |
| **containerd** | Logica di esecuzione (avvio, arresto, pausa, disattivazione, cancellazione dei container) |
| **runc** | CLI leggera per il runtime conforme allo standard OCI |

---

## Immagini Docker

Un'immagine Docker **non** e un disco rigido virtuale ne un file system tradizionale:
- Utilizza un **Union File System** (OverlayFS)
- E un **layer di sola lettura**
- **Non ha stato**
- E fondamentalmente un file compresso
- Ha una **gerarchia** (profondita arbitraria)

Ogni istruzione nel Dockerfile aggiunge un nuovo layer all'immagine; i layer sono condivisi tra immagini diverse che partono dalla stessa base.

### Docker Registry

| Tipo | Descrizione |
|---|---|
| Registro locale | Sullo stesso host |
| Docker Hub | Registro pubblico su docker.com |
| Registro privato | Condiviso globalmente su infrastruttura propria |

---

## Installazione di Docker

```bash
# Metodo automatico (non interattivo)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

Guida ufficiale Ubuntu: https://docs.docker.com/engine/install/ubuntu/

---

## Comandi di base Docker

| Comando | Funzione |
|---|---|
| `docker build` | Costruisce un'immagine dal Dockerfile |
| `docker run` | Avvia un nuovo container da un'immagine |
| `docker ps` | Elenca i container in esecuzione |
| `docker stop` | Arresta un container in esecuzione |
| `docker rm` | Rimuove un container |
| `docker images` | Elenca le immagini disponibili sul sistema |
| `docker pull` | Scarica un'immagine da un registro |
| `docker exec` | Esegue un comando in un container in esecuzione |

Cheatsheet: https://docs.docker.com/get-started/docker_cheatsheet.pdf

---

## Dockerfile

Riferimento: https://docs.docker.com/reference/dockerfile/

| Direttiva | Funzione |
|---|---|
| `FROM` | Crea un nuovo layer a partire dall'immagine specificata |
| `WORKDIR` | Cambia la working directory di default |
| `COPY` | Copia file in una directory |
| `RUN` | Esegue i comandi specificati, creando un nuovo layer |
| `ENTRYPOINT` | Specifica l'eseguibile di default lanciato all'avvio del container |
| `CMD` | Specifica i comandi di default (ignorato se si passa un argomento a `docker run`) |
| `EXPOSE` | Definisce le porte su cui l'app nel container si mette in ascolto |

---

## Esempio: App Flask containerizzata

### 1. Applicazione Flask

```python
# flask_app.py
from flask import Flask, request
import socket

app = Flask(__name__)

@app.route("/")
def index():
    return "This is an example containerized Flask app served from {} to {}".format(
        socket.gethostname(), request.remote_addr)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5001, debug=True)
```

### 2. Dockerfile

```dockerfile
FROM python:3.10-alpine
WORKDIR /app
# assumiamo che l'app flask sia all'interno della dir app
COPY requirements.txt /app
RUN  pip3 install -r requirements.txt
COPY flask_app.py /app
ENTRYPOINT ["python3"]
# ENTRYPOINT viene trattato come parametro da dare al docker run
CMD ["flask_app.py"]
# viene ignorato nel caso in cui passiamo un argomento (oltre a quello di ENTRYPOINT) tramite docker run
```

### 3. Build e Run

```bash
cd /path/to/Dockerfile
docker build -t flask_image_hello_world .
docker run -it --name flask_app_container -p 5001:5001 flask_image_hello_world
```

---

## Docker Compose

Docker Compose permette di fare il deploy di applicazioni **multi-container** tramite un file `.yaml`, risolvendo il problema di build/run/link manuali e gestione delle dipendenze.

### Parametri principali del Compose file

| Parametro | Funzione |
|---|---|
| `build` | Specifica il Dockerfile di compilazione (con sottocampo `context`) |
| `command` | Sovrascrive il CMD del Dockerfile |
| `ports` | Espone porte del container (`host:container`) |
| `volumes` | Mountpoint del filesystem host accessibili dal container |
| `links` | Collegamento di rete a container di un altro servizio |
| `image` | Immagine da cui avviare il container |
| `environment` | Variabili d'ambiente impostate nel container |

### Installazione e comandi principali

```bash
# Installazione (Linux)
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Comandi
docker compose up           # build e run di tutti i container
docker compose up -d        # modalita daemonized (background)
docker compose down         # stop e distruzione di tutti i container
docker compose ps           # lista i container del progetto
docker compose logs         # stampa i log del progetto
```

### Esempio Compose file

```yaml
# compose.yaml
services:
  web:
    build:
      context: app
    ports:
      - '5001:5001'
```

```bash
docker compose -f compose.yaml up -d
```

---

## Docker Swarm

**Docker Swarm** orchestra un **cluster di istanze Docker**. Le funzionalita di gestione sono incorporate nel Docker engine stesso.

### Nodi dello Swarm

| Tipo di nodo | Ruolo |
|---|---|
| **Manager** | Gestisce il join allo swarm, distribuisce task, usa algoritmo **Raft** per il consenso |
| **Worker** | Riceve ed esegue i task assegnati dai manager |

Un host Docker puo svolgere il ruolo di manager, worker o entrambi.

### Algoritmo Raft (manager)

- Tollera fino a \( \lfloor (N-1)/2 floor \) fallimenti di manager
- Richiede una maggioranza (**quorum**) di \( \lfloor N/2 floor + 1 \) membri per accordarsi

---

## Servizio Swarm

Per ogni servizio si puo definire lo **stato ottimale (desiderato)**:
- Numero di **repliche**
- Risorse di rete e storage disponibili
- Porte esposte verso l'esterno
- Vincoli e preferenze di posizionamento

Docker lavora per **mantenere** lo stato desiderato. Es.: se un nodo worker diventa indisponibile, Docker ripianifica i suoi task su altri nodi.

### Task Swarm

- Un **task** implementa un container Docker + i comandi da eseguire
- E l'**unita atomica di scheduling** in Docker Swarm
- Una volta assegnato a un nodo, non puo spostarsi

---

## Tipi di servizio in Swarm

| Tipo | Comportamento |
|---|---|
| **Replicato** | Lo swarm manager distribuisce un numero specifico di repliche tra i nodi |
| **Globale** | Lo swarm esegue un task del servizio su **ogni nodo disponibile**; quando si aggiunge un nodo, gli viene assegnato un task |

### Controllo del placement

- Servizio replicato o globale
- Riserva di memoria o CPU per un servizio
- **Vincoli di placement**: il servizio viene eseguito solo su nodi con una specifica etichetta
- **Preferenze di placement**: distribuzione uniforme dei task sui nodi appropriati

---

## Service Update in Swarm

Docker Swarm supporta **aggiornamenti online** di configurazione di rete e volumi:
- Avvia nuovi nodi con la nuova configurazione
- Arresta automaticamente i vecchi nodi
- Gestione di **rollback** e aggiornamenti dei servizi

---

## Utilizzo di Docker Swarm

```bash
# Creazione dello swarm (sul nodo master)
docker swarm init --advertise-addr IP_NODO_MASTER

# Join allo swarm (sui nodi worker)
docker swarm join --token TOKEN_GENERATO IP_NODO_MASTER:2377

# Creazione di un servizio
docker service create ...
# Es.: con replicazione
docker service create --replicas 3 --name flask_service --publish 5001:5001 flask_image_hello_world

# Rimozione del servizio
docker service rm ...

# Deploy tramite stack (con file yaml)
docker stack deploy --compose-file=compose.yaml nome_stack

# Rimozione dello stack
docker stack rm nome_stack

# Uscire dallo swarm
docker swarm leave
```

---

## Esempio: App Flask replicata con Swarm

### Sequenza operazioni

1. Creazione dello swarm sul nodo master
2. Join allo swarm dei nodi worker
3. Build dell'immagine su **tutti i nodi** dello swarm
4. Creazione del servizio con 3 repliche:

```bash
docker service create --replicas 3   --name flask_helloworld_service   --publish 5001:5001   flask_image_hello_world
```

- Le richieste GET verso il nodo manager vengono **bilanciate** tra tutti i container in esecuzione
- **Availability**: se un nodo fallisce, Swarm ripianifica automaticamente le repliche

### Emulare indisponibilita di un nodo

```bash
docker node update --availability drain IP_HOST
```

### Deploy con Docker Stack e file yaml

```yaml
# compose_with_stack.yaml (estratto)
...
deploy:
  replicas: 5
  restart_policy:
    condition: on-failure
    max_attempts: 3
    window: 120s
...
```

```bash
# Build su tutti i nodi worker
docker build -t /PATH/TO/flask_image_hello_world_DOCKERFILE

# Deploy dal nodo master
docker stack deploy --compose-file=/PATH/TO/compose_with_stack.yaml dockerized_flask_service_with_stack
```

---

## Schema riassuntivo: Docker, Compose, Swarm

| Strumento | Ambito | Uso |
|---|---|---|
| `docker` | Singolo container | Build, run, stop, exec di container |
| `docker compose` | Multi-container su singolo host | Deploy di applicazioni composte da piu servizi |
| `docker swarm` | Multi-container su cluster di host | Orchestrazione, scaling, availability |

---

## Collegamento con slide precedenti

- [[01_PYTHON_13 - Python Flask]]: esempio Flask usato come app da containerizzare
- [[01_PYTHON_10 - Sistemi Middleware]]: concetti di deployment e disaccoppiamento tra servizi
- [[02_JAVA_05 - MOM JMS]]: ActiveMQ puo essere deployato come container Docker
