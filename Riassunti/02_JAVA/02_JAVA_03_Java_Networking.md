# 02_JAVA_03 - Socket e Programmazione Client-Server in Java

#java #advanced-computer-programming #socket #TCP #UDP #networking #ServerSocket #InetAddress #multithread

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Oracle Java Networking Tutorial](http://docs.oracle.com/javase/tutorial/networking/)
- E.R. Harold, *Java Network Programming*, 4th Ed., O'Reilly — cap. 8, 9, 12
- Hughes, Shoffner et al., *Java Networking Programming*, 2nd Ed., Manning, 2000 — cap. 14, 16, 20

---

## Concetti Chiave

- Socket TCP: classi `Socket` e `ServerSocket` nel package `java.net`
- Flusso di connessione TCP: `ServerSocket(port)` → `accept()` → `Socket`
- Costruttori `ServerSocket`: `port`, `backlog` (default 50, porte riservate 1-1023)
- `accept()`: bloccante, restituisce la `Socket` per la comunicazione con il client
- `Socket(nomeHost, numport)`: apertura connessione lato client
- Comunicazione client/server: `getInputStream()`, `getOutputStream()`, `Reader`/`Writer`
- Server multithread: un thread worker per connessione
- Socket UDP: `DatagramPacket` + `DatagramSocket`, modello connectionless
- Ricezione datagramma: buffer preallocato, `receive()` bloccante, max 65508 byte
- Invio datagramma: `DatagramPacket(data, length, InetAddress, port)` + `send()`
- `InetAddress`: astrazione IP, DNS lookup con `getByName()`, metodi utili ed eccezioni

---

## Socket TCP/IP in Java

Per comunicare su rete tra applicazioni distribuite, Java mette a disposizione due classi principali del package `java.net`:

| Classe | Ruolo |
|---|---|
| `Socket` | Crea e gestisce una connessione TCP |
| `ServerSocket` | Usata dal server per accettare richieste di connessione |

### Programmazione client-server TCP/IP

Il processo server che gira sul nodo X si pone in attesa di richieste di connessione sulla porta Y usando `ServerSocket`.

### Lato server

**1. Creazione della `ServerSocket`**
```java
ServerSocket server = new ServerSocket(port);
```

**2. Attesa della connessione**
```java
Socket client = server.accept();
```

Il server si blocca su `accept()` finche non arriva una richiesta di connessione.

### Classe `ServerSocket`

```java
ServerSocket(int port) throws IOException
ServerSocket(int port, int backlog) throws IOException
```

| Parametro | Significato |
|---|---|
| `port` | Port number sull'host locale (1-65535; da 1 a 1023 sono riservate) |
| `backlog` | Numero massimo di richieste accodabili dal sistema operativo |

- Usando il primo costruttore, `backlog` assume il valore di default **50**.

### Metodo `accept()`

```java
Socket accept() throws IOException
```

- Comporta l'attesa di una richiesta di connessione (**listening**)
- Alla ricezione della connessione, `ServerSocket` crea una **Socket** che rappresenta la connessione TCP con il client
- Il riferimento alla `Socket` viene restituito al chiamante

> Invocando nuovamente `accept()`, il server puo essere rimesso in attesa di una nuova connessione client.

---

## Programmazione lato client

Il client richiede una connessione creando una `Socket`:

```java
Socket socket = new Socket(address, PORTNUM);
```

oppure in forma piu generale:

```java
Socket conn = new Socket(nomeHost, numport);
```

- La classe `Socket` fornisce un'interfaccia socket per i **client TCP**
- La socket rappresenta il canale tra `Host:port` client e `host:port` server tramite TCP/IP

### Effetto lato server

Quando il client si connette:
- il server **esce** dalla `accept()`
- riceve la `Socket` dedicata alla comunicazione con quel client

---

## Comunicazione client/server

Una volta stabilita la connessione, la comunicazione avviene tramite stream:

```java
InputStream  in  = conn.getInputStream();
OutputStream out = conn.getOutputStream();
```

In alternativa si possono usare wrapper di piu alto livello:

```java
writer = new OutputStreamWriter(out);
reader = new InputStreamReader(in);
```

| Approccio | Uso |
|---|---|
| `InputStream` / `OutputStream` | Dati binari o gestione a basso livello |
| `Reader` / `Writer` | Più comodo per dati testuali |

---

## Server Multithread

Un server multithread migliora l'efficienza creando **uno o piu thread per connessione**.

### Idea generale

- Il server principale resta in ascolto su `ServerSocket`
- Per ogni client accettato crea un **worker thread**
- Ogni worker gestisce la propria `Socket`

### Thread Worker

```java
public class MyWorker extends Thread {
    private Socket socket;

    MyWorker(Socket skt) {
        socket = skt;
    }

    public void run() {
        InputStream  in = socket.getInputStream();
        OutputStream ot = socket.getOutputStream();
        // elabora richiesta
        ...
    }
}
```

### Schema generale del server

```java
while (...) {
    Socket client = server.accept();
    MyWorker myWorker = new MyWorker(client);
    myWorker.start();
}
```

> Nel `run()` del worker e contenuto tutto il codice per il soddisfacimento della richiesta del client.

---

## Socket UDP/IP

Per la comunicazione basata su UDP, Java fornisce due classi principali nel package `java.net`:

| Classe | Ruolo |
|---|---|
| `DatagramPacket` | Astrazione del pacchetto UDP (datagramma) |
| `DatagramSocket` | Astrazione del socket UDP |

La struttura dati di un `DatagramPacket` consiste di due parti:
- `address`
- `data`

### Classe `DatagramSocket`

UDP e **connectionless**, quindi la stessa socket puo essere usata per inviare pacchetti a destinazioni diverse.

```java
DatagramSocket() throws SocketException
DatagramSocket(int port) throws SocketException
```

### Metodi principali

```java
void send(DatagramPacket packet)    throws IOException
void receive(DatagramPacket packet) throws IOException
```

---

## Ricezione di un datagramma

```java
DatagramSocket socket = new DatagramSocket(port);
byte buffer[] = new byte[65508];
// 65508 e la max dimensione di un pacchetto UDP

DatagramPacket pkt = new DatagramPacket(buffer, buffer.length);
// Se forniamo un buffer troppo piccolo,
// la parte eccedente del pacchetto andrà persa

socket.receive(pkt);   // chiamata bloccante

InetAddress fromAddr = pkt.getAddress();
int fromPort         = pkt.getPort();
int length           = pkt.getLength();
byte[] data          = pkt.getData();
```

### Note importanti

- `buffer` e un array di byte che conterra il contenuto del pacchetto all'arrivo
- L'array e **preallocato**
- `length` rappresenta il massimo numero di byte letti nel buffer
- I dati extra nel pacchetto UDP vengono **scartati**

---

## Invio di un datagramma

```java
DatagramSocket socket = new DatagramSocket();

DatagramPacket pkt = new DatagramPacket(
    data,
    data.length,
    InetAddress.getByName("pippo.pluto.it"),
    portNum
);

socket.send(pkt);
```

Qui si costruisce il pacchetto UDP specificando:
- payload (`data`)
- lunghezza (`data.length`)
- indirizzo destinazione (`InetAddress`)
- porta destinazione (`portNum`)

---

## Classe `InetAddress`

`InetAddress` rappresenta l'**astrazione dell'indirizzo IP** in Java.

### Perche serve

Gli indirizzi numerici sono difficili da ricordare per gli esseri umani, quindi si preferisce usare stringhe come:

```text
www.grid.unina.it
```

Queste stringhe vengono convertite nell'indirizzo IP associato interrogando il **DNS** tramite:

```java
InetAddress.getByName("www.grid.unina.it")
```

### Metodi di utilita

| Metodo | Descrizione |
|---|---|
| `InetAddress getLocalHost()` | Restituisce l'indirizzo IP della macchina locale |
| `InetAddress getByName(String host)` | Restituisce l'IP associato a un host name |
| `InetAddress[] getAllByName(String host)` | Restituisce tutti gli IP associati a un host name |
| `byte[] getAddress()` | Restituisce l'indirizzo come sequenza di byte |
| `String getHostName()` | Restituisce il nome della macchina su cui gira il programma |
| `boolean isMulticastAddress()` | Restituisce `true` se l'indirizzo e multicast |

### Eccezioni

| Eccezione | Significato |
|---|---|
| `UnknownHostException` | L'host non puo essere correttamente identificato |
| `SecurityException` | Il `SecurityManager` non permette l'operazione richiesta |

---

## TCP vs UDP in Java

| Aspetto | TCP | UDP |
|---|---|---|
| Classi principali | `Socket`, `ServerSocket` | `DatagramSocket`, `DatagramPacket` |
| Connessione | Si | No |
| Modalita | Stream-oriented | Packet-oriented |
| `accept()` | Si, lato server | No |
| Affidabilita | Alta | Nessuna garanzia di consegna |
| Ordine dei dati | Garantito | Non garantito |
| Ricezione | Stream di byte | Datagrammi |
| Multi-client | `accept()` + thread worker | Gestione per pacchetti |

---

## Collegamenti utili

- [[02_JAVA_01 - Multithreading in Java]]
- [[02_JAVA_02 - Sincronizzazione in Java]]
- [[01_PYTHON_09 - Python Networking]]
- [[01_PYTHON_10 - Sistemi Middleware]]
