## Introduzione

L'uso di una **virtual machine (VM)** per simulare un sistema operativo comporta un onere prestazionale non indifferente, dovuto ai vari **overhead** presenti (hypervisor, sistema guest, ecc...).

Le criticità principali delle VM sono:

- **Overhead prestazionale**: dovuto alla presenza di componenti come hypervisor e sistema guest.
- **Occupazione di memoria**: le VM occupano grandi spazi di memoria.
- **Avvio lento**: il tempo di avvio non è dei più veloci.
- **Problema principale**: esistono numerosissimi componenti applicativi che, se gestiti tramite macchine virtuali o computer classici, richiedono **configurazioni apposite** in base all'ambiente necessario.

## I Container come Alternativa

Un'alternativa alle macchine virtuali sono i **Container**, ovvero una forma di **virtualizzazione leggera** in cui:

- non c'è **emulazione dei dispositivi fisici**, quindi l'overhead dell'hypervisor è nullo;
- si fa invece uso dell'**astrazione dei processi** e della capacità di gestione delle risorse del sistema host, tramite **Namespaces** e **Cgroups**.

### Confronto tra VM e Container

| Caratteristica | Virtual Machine | Container |
|---|---|---|
| **Velocità di avvio** | Più lenta | Decisamente più veloce |
| **Conoscenze richieste** | Ambiente di esecuzione + sistema operativo guest | Solo ambiente di esecuzione e codice dell'applicazione |
| **Leggerezza** | Dischi virtuali pesanti | Dischi virtuali di pochi MB |
| **Densità per server fisico** | Es. 100 VM | Es. 1000 Container (di media) |

### Vantaggi principali

L'uso di Container offre una soluzione al problema principale delle VM evidenziato in precedenza: eseguendo i componenti applicativi in un Container, è possibile farne uso da **praticamente qualsiasi hardware**.

## Namespaces

Un **Namespace** è un dominio di denominazione di varie risorse. Ne esistono di vari tipi, in base alla risorsa che isolano:

| Tipo di Namespace | Risorsa isolata |
|---|---|
| `mnt` | Mount point (i filesystem) |
| `pid` | Processi |
| `net` | Stack di rete |
| `ipc` | Inter-Process Communication |
| `uts` | Hostname |
| `user` | User ID |

Tramite i **Namespaces**, un **Container** agisce senza consapevolezza dell'astrazione, come se i programmi fossero in esecuzione direttamente su un sistema operativo. Sarà poi il **kernel**, riconoscendo che le system call derivano da un Container, a permettergli di accedere alle risorse collegate ai suoi Namespaces.

### Principali System Call

Le principali chiamate di sistema relative ai Namespace sono:

- **`clone()`**: crea un processo (ed un namespace, se si specificano i flag `CLONE_NEWxxx`, dove `xxx` indica il tipo di Namespace).
- **`unshare()`**: non crea nuovi processi, ma viene utilizzata per attaccare nuovi namespace al processo chiamante, sempre tramite l'uso dei flag `CLONE_NEWxxx`.
- **`setns()`**: permette di attaccare un namespace già esistente ad un container in esecuzione.

## Cgroups

I **Cgroups** (*control groups*) rappresentano un sottosistema del kernel Linux dedicato alla **gestione delle risorse**.

### Struttura nel Kernel

I moduli di Cgroups **non si trovano in un'unica cartella**, ma sono distribuiti all'interno del kernel in base alla loro funzionalità specifica.

### Controller Principali

Tra i controller più rilevanti si trovano:

| Controller   | Funzionalità                                                            |
| ------------ | ----------------------------------------------------------------------- |
| **memory**   | Definisce i limiti di uso di RAM da parte di un Cgroup                  |
| **cpuset**   | Assegna processori e nodi di memoria ai Cgroups                         |
| **net_prio** | Permette di configurare il traffico di rete per ogni Cgroup             |
| **devices**  | Regola l'accesso ai dispositivi hardware da parte di un Cgroup          |
| **blkio**    | Regola l'uso della banda di input/output da e verso le memorie di massa |
| ...          | ...                                                                     |

### Definizione di Cgroup

In pratica, un **Cgroup** è un **gruppo logico di processi** che condividono:

- le stesse **regole**
- le stesse **limitazioni**
- le stesse **metriche** per l'uso di risorse di sistema

(definite dai controller sopra descritti), la cui traccia è tenuta e gestita dal **kernel**.

## Docker

### Cos'è Docker
Docker è un **motore di container** scritto in **Go**.

### Architettura Client-Server
Il funzionamento di Docker si basa su un'architettura **Client-Server**:

- Il **Client** invia comandi da terminale (es. `docker build`, `docker pull`, `docker run`) per interagire con il **DOCKER_HOST** (il Server).
- Il Client non ha bisogno di eseguire il container localmente: tramite questi comandi si interfaccia con il motore Docker per istanziare container remoti.

Il Server gestisce due risorse principali:

- **Containers**: le istanze effettive in esecuzione.
- **Images**: template pronti all'uso sulla base dei quali lanciare i container.

> Se un Client richiede un template non presente localmente, il DOCKER_HOST lo scarica dal **Registry**, ovvero un repository esterno come Docker Hub.

### Componenti del motore Docker
I componenti principali del motore Docker sono: **daemon**, **containerd**, **shim** e **runc**.

#### Workflow di esecuzione
1. Viene eseguito un comando (es. `docker container run ...`).
2. Arriva tramite **API REST** una richiesta **POST** al **daemon**.
3. Il daemon comunica con **containerd** (ad esempio tramite **gRPC**) e gli delega la gestione del ciclo di vita del container: containerd si occupa infatti della logica di esecuzione del container.
4. Per ogni container in esecuzione viene istanziato un processo **shim**, che ha la responsabilità di:
   - accogliere i processi generati da runc;
   - mantenere aperti gli stream input/output finché i rispettivi processi non terminano.
   
   Questo avviene in maniera disaccoppiata da containerd, in modo che quest'ultimo possa continuare a gestire la logica di esecuzione degli altri container senza rallentarsi. Soprattutto, garantisce che un malfunzionamento sul daemon o su containerd non causi il crash del singolo container, che continua a girare sul proprio shim.
5. **runc** è il runtime di livello più basso e si interfaccia direttamente con il kernel (ad esempio, è lui a creare **namespaces** e **cgroups**). La sua caratteristica fondamentale è che non rimane in esecuzione dopo aver configurato e generato il processo principale del container (che verrà accolto dallo shim apposito).

### Immagini Docker
Un'immagine Docker **non** rappresenta un disco virtuale o un file system, ma un'entità **di sola lettura** e **senza stato**, rappresentata concretamente da un file compresso dotato di gerarchia:

- alla base troviamo il **kernel host**;
- su di esso poggiano i **layer** dell'immagine, solitamente a partire da un'immagine base (es. una distribuzione Debian);
- seguono altri layer che rappresentano installazioni o istruzioni aggiuntive;
- il layer più alto è il **container**, l'unico strato **scrivibile**, che ospita tutto ciò che viene creato, modificato e cancellato a runtime.

Per far sì che il container layer agisca come un sistema operativo unico, Docker utilizza uno **Union File System**, che unifica tutte le gerarchie e le directory presenti nell'immagine in un unico file system coerente. Lo Union File System utilizzato da Docker è **OverlayFS**.

### Registry
Il Registry può essere di vari tipi:

- locale sull'host che esegue Docker;
- registro pubblico **Docker Hub**;
- registri privati ma condivisi globalmente su docker.com.

### Dockerfile
Per creare un'immagine Docker, il motore legge un **Dockerfile**, ovvero un file contenente una serie di istruzioni eseguite sequenzialmente da Docker che impostano:

- l'immagine base;
- le installazioni aggiuntive;
- le porte su cui comunica il container;
- i comandi da eseguire all'avvio;
- ecc.

#### Direttive principali del Dockerfile

| Direttiva    | Funzione                                                                   |
| ------------ | -------------------------------------------------------------------------- |
| `FROM`       | Crea un nuovo layer per l'immagine Docker                                  |
| `WORKDIR`    | Imposta la directory di default                                            |
| `COPY`       | Copia i file nella workdir                                                 |
| `RUN`        | Esegue i comandi creando un nuovo layer al di sopra di quello attuale      |
| `ENTRYPOINT` | Specifica l'eseguibile da lanciare automaticamente all'avvio del container |
| `CMD`        | Specifica dei comandi di default che il container eseguirà all'avvio       |
| `EXPOSE`     | Definisce le porte su cui le applicazioni del container saranno in ascolto |

### Comandi principali

| Comando         | Funzione                                               |
| --------------- | ------------------------------------------------------ |
| `docker build`  | Costruisce un'immagine Docker a partire dal Dockerfile |
| `docker run`    | Avvia un container Docker da un'immagine               |
| `docker ps`     | Elenca tutti i container Docker in esecuzione          |
| `docker stop`   | Arresta l'esecuzione di un container Docker            |
| `docker rm`     | Rimuove un container Docker                            |
| `docker images` | Elenca tutte le immagini Docker disponibili            |
| `docker pull`   | Scarica un'immagine Docker da un registro              |
| `docker exec`   | Esegue un comando in un container Docker in esecuzione |

## Docker Compose

### Il problema del deploy multi-container

Per deployare un'applicazione **multi-container** bisognerebbe:

- fare il **build** e il **run** di ogni singolo container manualmente;
- collegare eventuali container che devono comunicare tra loro;
- prestare attenzione alle **dipendenze** tra i container (quindi all'ordine di avvio).

Per ovviare a questo problema è possibile fare uso di **Docker Compose**, grazie al quale si specifica l'applicazione multi-container all'interno di un file `.yaml`. Questo rende possibile:

- effettuare il deploy dell'intera applicazione con un **singolo comando**;
- gestire automaticamente le **dipendenze di avvio**, in base a come viene configurato il file `.yaml`.

### Parametri principali del file `.yaml`

| Parametro | Descrizione |
|---|---|
| **build** | Specifica la configurazione di compilazione. Al suo interno si può specificare anche il parametro **context**, per definire il percorso o l'URL che contiene il `Dockerfile`. |
| **command** | Sovrascrive il comando predefinito dichiarato dal `Dockerfile` (il parametro `CMD`). |
| **ports** | Espone le porte per il container. A differenza di `EXPOSE`, questo parametro le **mappa** anche. |
| **volumes** | Definisce i **mount points** accessibili dal container. |
| **links** | Definisce un collegamento (in rete) tra container/servizi. |
| **image** | Specifica l'**immagine** da cui avviare il container. |
| **environment** | Definisce le **variabili d'ambiente** da impostare nel container. |

### Comandi principali di Docker Compose

| Comando | Descrizione |
|---|---|
| **up** | Fa il **build** e il **run** di tutti i container. |
| **down** | Ferma e distrugge tutti i container in esecuzione. |
| **ps** | Lista i container del file `.yaml` presente nella directory. |
| **logs** | Stampa i log dei container del file `.yaml` presente nella directory. |

# Docker Swarm

## Architettura e ruoli dei nodi

Uno swarm è un **orchestratore** di un cluster di istanze Docker, le cui funzionalità sono incorporate nel Docker engine.

Gli host Docker che vengono eseguiti in modalità swarm possono agire in due modi:

- **Manager**: gestiscono il join e la delega all'interno dello swarm.
- **Worker**: eseguono i servizi effettivi, secondo gli incarichi ricevuti dai manager.

È possibile che un singolo host Docker sia contemporaneamente **manager** di altri worker e **worker** per altri manager.

## Algoritmo di consenso Raft

I nodi manager gestiscono lo stato globale del cluster tramite l'algoritmo di consenso **Raft**.

| Concetto | Formula | Significato |
|---|---|---|
| Fallimenti tollerati | **(N-1)/2** | numero massimo di manager che possono fallire mantenendo il sistema correttamente amministrabile |
| Quorum | **(N/2)+1** | maggioranza necessaria per prendere decisioni gestionali |

dove **N** rappresenta il numero di manager dello swarm.

## Servizi

Un **servizio** in Docker Swarm non rappresenta un singolo container, ma bensì la modalità secondo la quale un'applicazione deve operare all'interno del cluster.

Si può definire lo stato ottimale di un servizio stabilendo:

- il numero di **repliche**
- le **risorse** (di rete e di memoria) da assegnargli
- le **porte** che vengono esposte
- i **vincoli** e le **preferenze** sul posizionamento
- ecc.

Il compito di Docker è quindi quello di assicurarsi che lo stato ottimale di uno Swarm venga mantenuto.

Per effettuare il **deploy** di un servizio in Docker Swarm, bisogna inviare una definizione di servizio a un nodo manager, che distribuirà i task ai nodi worker, assicurandosi sempre che lo stato desiderato dello Swarm sia mantenuto (tramite funzioni di orchestrazione e gestione del cluster).

### Modalità di distribuzione dei servizi

I servizi in uno Swarm possono essere distribuiti in due maniere principali:

| Modalità | Descrizione |
|---|---|
| **Servizi replicati** | task replicati assegnati a un numero di nodi deciso in base alla configurazione |
| **Servizi globali** | una replica assegnata ad ogni nodo disponibile |

Inoltre, è possibile controllare anche dove e con quali risorse posizionare i vari servizi, specificando **vincoli** e **preferenze**.

## Task

Un **task** rappresenta l'implementazione pratica di un container Docker, definita come unità atomica di scheduling per uno Swarm.

Una volta che un task viene assegnato ad un nodo, può solo essere **eseguito** o **fallire**, non si può spostare.

## Aggiornamenti e rollback

Docker Swarm permette l'**aggiornamento a caldo** dei servizi:

- avvia i nuovi nodi con il servizio aggiornato
- gestisce i vecchi nodi con il servizio non aggiornato
- implementa anche una gestione dei **rollback**

## Comandi principali

### Creazione dello swarm

```bash
docker swarm init --advertise-addr <IP>
```

Il flag `--advertise-addr` permette di dichiarare l'IP del manager. Dopo l'esecuzione di questo comando, il nodo corrente diventa il master e si dà il via al cluster.

### Aggiungere nodi allo swarm

Per aggiungere nodi (worker o manager) allo swarm si esegue, sulla macchina da unire, specificando l'IP del nodo master dello swarm a cui unirsi e il token generato dalla `init`:

```bash
docker swarm join --token TOKEN IP:2377
```

> **Nota di revisione tecnica:** il comando originale riportava `---token` (tre trattini), sintassi non valida per Docker CLI. Il flag corretto è `--token` (due trattini).

### Gestione di servizi e stack

```bash
# Creazione e rimozione di un servizio singolo
docker service create
docker service rm

# Creazione e rimozione di stack complessi (gestibili con Docker Compose)
docker stack deploy
docker stack rm
```

### Uscita dallo swarm

```bash
docker swarm leave
```