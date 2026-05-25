# 09 - Programmazione su Rete in Python

#python #advanced-computer-programming #networking #socket #TCP #UDP #proxy #skeleton

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Python socket docs](https://docs.python.org/3/library/socket.html#socket-objects)

---

## 📌 Concetti Chiave

- Internet, indirizzi IP (IPv4/IPv6), DNS, routing, IP datagram
- TCP vs UDP: differenze fondamentali
- Socket: concetto, creazione con `socket.socket()`
- API socket: `bind`, `listen`, `accept`, `connect`, `send/recv`, `sendto/recvfrom`
- Implementazione Client-Server TCP e UDP con esempi
- Server Multi-thread e Multi-process
- Pattern Proxy-Skeleton: separazione logica applicativa da comunicazione
- Implementazione Proxy-Skeleton in Python con `ABC`

---

## 🌐 Cenni su Internet

Internet è una **rete di reti** mondiale basata sul protocollo TCP/IP, nata da ARPANET (1969, ARPA/Dipartimento della Difesa USA).

**Componenti principali:**
- **Host/nodi**: PC, server, smartphone, tablet, ...
- **Link di comunicazione**: cavi, fibra ottica, radio, satellitari
- **Router**: dispositivi che instradano i messaggi attraverso la rete

### Indirizzi IP

| Versione | Lunghezza | Esempio | N° indirizzi |
|---|---|---|---|
| **IPv4** | 32 bit (4 campi da 8 bit) | `192.168.0.4` | ~4,3 miliardi |
| **IPv6** | 128 bit (8 campi da 16 bit) | `2001:0db8:85a3:0000:1319:8a2e:0370:7344` | 655.571 miliardi di miliardi |

- La traduzione nome simbolico → indirizzo IP è gestita dal **DNS** (Domain Name Service)
- I dati sono trasmessi in **IP datagram** (max 65.535 byte)
- IP fornisce un servizio **best-effort**: pacchetti potrebbero arrivare corrotti, duplicati, fuori ordine o persi

---

## 📡 TCP vs UDP

| Caratteristica | **TCP** | **UDP** |
|---|---|---|
| Affidabilità | ✅ Affidabile | ❌ Non affidabile |
| Orientamento | Orientato alla connessione | Connectionless (datagrammi) |
| Flusso | Flusso di byte full-duplex | Datagrammi indipendenti |
| Ordine | ✅ Preservato, senza duplicati | ❌ Non garantito |
| Ritrasmissione | ✅ Automatica | ❌ Assente |
| Porte | ✅ Sì | ✅ Sì |
| Overhead | Alto | Basso |

> [!important]
> Il livello IP è **best-effort**. È TCP a implementare l'affidabilità sopra di esso.

**Stack protocollare:**
```
Livello applicativo
      TCP
      IP
```

---

## 🔌 Socket

Una socket è un **meccanismo di comunicazione tra processi** (anche su macchine diverse) basato sulla rete. Fornisce un'interfaccia di accesso alla rete.

**Caratterizzata da:**
- **Indirizzo IP** del nodo
- **Numero di porta**

### Porte Note

| Porta | Protocollo | Servizio |
|---|---|---|
| 20/tcp | FTP | File Transfer (data) |
| 21/tcp | FTP | File Transfer (control) |
| 22/tcp | SSH | Secure login |
| 23/tcp | Telnet | Testo non cifrato |
| 25/tcp | SMTP | Email |
| 53/tcp+udp | DNS | Domain Name Service |
| 69/udp | TFTP | Trivial File Transfer |
| 80/tcp | HTTP | World Wide Web |

### Creazione di una Socket in Python

```python
import socket

s = socket.socket(socket_family, socket_type)
# socket_family = socket.AF_INET       → IPv4
# socket_type   = socket.SOCK_STREAM   → TCP
# socket_type   = socket.SOCK_DGRAM    → UDP
```

---

## 🛠️ API Socket Principali

| Funzione | Descrizione |
|---|---|
| `socket()` | Crea una nuova socket |
| `bind(address)` | Associa la socket all'indirizzo `(IP, porta)` |
| `listen(backlog)` | Mette in ascolto (TCP); `backlog` = max connessioni in coda |
| `accept()` | Accetta una connessione; ritorna `(conn, addr)` |
| `connect(address)` | Connette verso l'indirizzo `(IP, porta)` (client TCP) |
| `send(string)` | Invia dati su una socket connessa (TCP) |
| `recv(bufsize)` | Riceve dati; ritorna una stringa (TCP) |
| `sendto(string, address)` | Invia dati verso un indirizzo (UDP, connectionless) |
| `recvfrom(bufsize)` | Riceve dati; ritorna `(string, address)` (UDP) |
| `close()` | Chiude la socket |

---

## 🔵 Socket TCP

### Flusso Server TCP

```
socket() → bind() → listen() → accept() [bloccante] → recv/send → close()
```

### Flusso Client TCP

```
socket() → connect() → send/recv → close()
```

### Esempio Server TCP

```python
import socket

IP          = 'localhost'
PORT        = 0             # porta disponibile automaticamente
BUFFER_SIZE = 1024

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((IP, PORT))
s.listen(1)
cur_port = s.getsockname()[1]   # porta assegnata
print("server on:", IP, "port:", cur_port)

conn, addr = s.accept()         # bloccante fino alla connessione
print('Connection address: {}'.format(addr))

data = conn.recv(BUFFER_SIZE)
print("received data: " + data.decode("utf-8"))

toClient = "The world never says hello back!\n"
conn.send(toClient.encode("utf-8"))

conn.close()
s.close()
```

### Esempio Client TCP

```python
import socket, sys

def client(PORT):
    IP          = 'localhost'
    BUFFER_SIZE = 1024
    MESSAGE     = "Hello, World!\n"

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, PORT))
    s.send(MESSAGE.encode("utf-8"))
    data = s.recv(BUFFER_SIZE)
    print("received data: " + data.decode("utf-8"))
    s.close()

if __name__ == "__main__":
    try:
        PORT = sys.argv[1]
    except IndexError:
        print("Please, specify PORT arg...")
    assert PORT != '', 'specify port'
    client(int(PORT))
```

---

## 🟠 Socket UDP

UDP è **connectionless**: non serve instaurare una connessione; il mittente viene identificato tramite `recvfrom()`.

### Flusso Server UDP

```
socket() → bind() → recvfrom() [bloccante] → sendto() → close()
```

### Flusso Client UDP

```
socket() → sendto() → recvfrom() → close()
```

### Esempio Server UDP

```python
import socket

msgServer         = "Ciao client!"
serverAddressPort = ("localhost".encode("utf-8"), 7000)
bufferSize        = 1024

s = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM)
s.bind(serverAddressPort)

msgClient, addr = s.recvfrom(bufferSize)
print("[Server]: Messaggio ricevuto: " + msgClient.decode("utf-8"))
print("[Server]: Indirizzo client: {}".format(addr))
s.sendto(msgServer.encode("utf-8"), addr)
s.close()
```

### Esempio Client UDP

```python
import socket, sys

def client(port):
    msgClient         = "Ciao server!"
    serverAddressPort = ("localhost".encode("utf-8"), port)
    bufferSize        = 1024

    s = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM)
    s.sendto(msgClient.encode("utf-8"), serverAddressPort)
    msgServer, addr = s.recvfrom(bufferSize)
    print("[Client]: Risposta server: " + msgServer.decode("utf-8"))
    s.close()
```

---

## 🚀 Server Multi-thread / Multi-process

Un server **singolo thread** può gestire una sola connessione alla volta. I server multi-thread/multi-process creano un nuovo thread/processo per ogni richiesta ricevuta.

**Pattern generale:**
- Loop principale: `accept()` (TCP) o `recvfrom()` (UDP)
- Per ogni richiesta: avvia un nuovo `Thread` o `Process` con il handler

### Server TCP Multi-thread

```python
import socket, threading

def thd_fun(c):
    data   = c.recv(1024)
    result = data[::-1]     # inverte la stringa
    c.send(result)
    c.close()

if __name__ == '__main__':
    host, port = "", 12345
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind((host, port))
    s.listen(5)
    print("Socket is listening")

    while True:
        c, addr = s.accept()
        print('Connected to :', addr[0], ':', addr[1])
        t = threading.Thread(target=thd_fun, args=(c,))
        t.start()
    s.close()
```

### Server TCP Multi-process

```python
import socket, multiprocess as mp

def proc_fun(c):
    data   = c.recv(1024)
    result = data[::-1]
    c.send(result)
    c.close()

if __name__ == '__main__':
    host, port = "", 12345
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind((host, port))
    s.listen(5)

    while True:
        c, addr = s.accept()
        p = mp.Process(target=proc_fun, args=(c,))
        p.start()
    s.close()
```

---

## 🧩 Pattern Proxy-Skeleton

### Problema

In un'applicazione client-server con socket, cliente e server devono gestire:
1. Connessione
2. Esternalizzazione (serializzazione) dei dati
3. Invio/ricezione delle richieste
4. Ricostruzione dei valori ricevuti

> [!warning] Rischio
> L'implementazione dei meccanismi di comunicazione **distrae** dalla logica applicativa, causando **sovrapposizione** tra logica applicativa e logica di comunicazione.

### Soluzione: Pattern Proxy

```
Client → Proxy (lato client) ←→ Skeleton (lato server) → ServerImpl
```

| Componente | Ruolo |
|---|---|
| **Proxy** ("Surrogato"/"Ambassador") | Si presenta come implementazione locale del servizio remoto; incapsula i meccanismi di comunicazione lato client |
| **Skeleton** | Riceve richieste, struttura i parametri, effettua l'up-call al servizio reale, ritorna i risultati al Proxy |
| **ServerImpl** | Implementa effettivamente i servizi; non conosce la comunicazione |
| **InterfacciaServer** | Specifica i servizi offerti (contratto comune) |

### Flusso di una Richiesta

1. Il client chiama `proxy.servizio1()`
2. Il Proxy serializza i parametri e li invia via socket al server
3. Lo Skeleton riceve i dati, li deserializza e chiama `serverImpl.servizio1()`
4. Il risultato risale: ServerImpl → Skeleton → Proxy → Client

### Skeleton: Ereditarietà vs Delega

| | **Skeleton per Ereditarietà** | **Skeleton per Delega** |
|---|---|---|
| Complessità | Più semplice | Più complesso |
| Struttura | Meno pulita | ✅ Più pulita |
| Classi esistenti | ❌ Problemi con ereditarietà singola | ✅ Funziona con gerarchie esistenti |

### Implementazione Python con `ABC`

**Interfaccia comune (`interface.py`):**
```python
from abc import ABC, abstractmethod

class Subject(ABC):
    @abstractmethod
    def request(self, data):
        pass
```

**Proxy (`client.py`):**
```python
import socket, sys
from interface import Subject

class Proxy(Subject):
    def __init__(self, port):
        self.port = port

    def request(self, message):
        IP, BUFFER_SIZE = 'localhost', 1024
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((IP, self.port))
        s.send(message.encode("utf-8"))
        data = s.recv(BUFFER_SIZE)
        print("received data: " + data.decode("utf-8"))
        s.close()

if __name__ == "__main__":
    PORT, MESSAGE = sys.argv[1], sys.argv[2]
    proxy = Proxy(int(PORT))
    proxy.request(MESSAGE)
```

**Skeleton per ereditarietà + RealSubject (`server.py`):**
```python
import socket, sys, threading
from abc import ABC, abstractmethod
from interface import Subject

class Skeleton(Subject, ABC):
    def __init__(self, port):
        self.port = port

    @abstractmethod
    def request(self, data): pass

    def run_skeleton(self):
        host = 'localhost'
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.bind((host, self.port))
        self.port = s.getsockname()[1]
        s.listen(5)
        print("Socket is listening")
        while True:
            c, addr = s.accept()
            t = threading.Thread(target=thd_fun, args=(c, self))
            t.start()
        s.close()

def thd_fun(c, self):
    data   = c.recv(1024)
    result = self.request(data)   # up-call al servizio reale
    c.send(result)
    c.close()

class RealSubject(Skeleton):
    def request(self, data):
        return data[::-1]         # inverte la stringa

if __name__ == "__main__":
    PORT = sys.argv[1]
    realSubject = RealSubject(int(PORT))
    realSubject.run_skeleton()
```

---

## 🔗 Collegamenti

- [[08 - Python Concurrency]]
- [[06 - Python OOP]]
- [[07 - Programmazione Concorrente Richiami]]
- [[Design Patterns Python]]
- [[Socket TCP vs UDP]]

---

## 📚 Risorse Utili

- [Python socket module docs](https://docs.python.org/3/library/socket.html#socket-objects)
- [GARR – rete italiana della ricerca](http://www.garr.it)
