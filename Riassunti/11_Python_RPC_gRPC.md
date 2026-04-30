# 11 - Remote Procedure Call in Python (gRPC)

#python #advanced-computer-programming #gRPC #protobuf #RPC #streaming #middleware #HTTP2

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [gRPC – Core Concepts](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [gRPC Python Quickstart](https://grpc.io/docs/languages/python/quickstart/)
- [gRPC Python Basics](https://grpc.io/docs/languages/python/basics/)

---

## 📌 Concetti Chiave

- gRPC: framework RPC universale open source di Google (CNCF)
- HTTP/2: Binary Framing Layer, stream, multiplexing, compressione header
- Protocol Buffer (protobuf): IDL + formato di serializzazione binario fortemente tipizzato
- Workflow gRPC: `.proto` → `protoc` → stub client + servicer server
- File generati: `_pb2.py` (messaggi) e `_pb2_grpc.py` (stub + servicer)
- Implementazione server e client Python (esempio Hello World)
- 4 tipi di RPC call: Unary, Server Streaming, Client Streaming, Bidirectional Streaming
- Generatori Python e `yield` per lo streaming
- Thread-safety di channel e stub
- Errori tipici e limitazioni di gRPC

---

## 🚀 gRPC

**gRPC** (grpc.io) è un framework RPC **universale, ad alte prestazioni e open source**, sviluppato da Google e ora parte della **Cloud Native Computing Foundation (CNCF)**.

**Scenari di utilizzo principali:**
- Collegare **microservizi poliglotti** con comunicazione richiesta-risposta
- Connettere **dispositivi mobili e browser** ai servizi di backend
- Generare **librerie client efficienti**

**Adottato da:** Google, Dropbox, Netflix, Square, etcd, CockroachDB

### Caratteristiche Principali

| Feature | Descrizione |
|---|---|
| **Trasporto** | HTTP/2 (streaming bidirezionale, multiplexing) |
| **IDL + serializzazione** | Protocol Buffer (protobuf) |
| **Generazione codice** | Automatica da file `.proto` |
| **Autenticazione** | Supporto integrato |
| **Controllo del flusso** | Supporto integrato |
| **Stub bloccante** | Il chiamante attende la risposta (sincrono) |
| **Stub non bloccante** | La risposta arriva in modo asincrono |

---

## 🌐 Cenni a HTTP/2

gRPC trasporta le RPC su **HTTP/2**, trattando le chiamate come riferimenti a oggetti HTTP.

### Concetti fondamentali HTTP/2

| Concetto | Definizione |
|---|---|
| **Frame** | Unità minima di comunicazione; contiene un header che identifica lo stream di appartenenza |
| **Message** | Sequenza completa di frame che corrispondono a una richiesta/risposta logica |
| **Stream** | Flusso bidirezionale di byte in una connessione stabilita; può trasportare più messaggi |

**Vantaggi rispetto a HTTP/1.x:**
- **Binary Framing Layer**: richieste/risposte suddivise in frame binari → trasmissione efficiente
- **Multiplexing**: una singola connessione TCP per client; evita il blocco della linea di testa (head-of-line blocking)
- **Streaming bidirezionale nativo**
- **Compressione degli header HTTP**: riduce l'overhead del protocollo

---

## 📦 Protocol Buffer (protobuf)

I **protobuf** sono il meccanismo open source di Google per la **serializzazione di dati strutturati**:
- Rappresentazione **binaria** (non human-readable)
- Descrizione **fortemente tipizzata**
- Basati sul modello **Proxy-Skeleton** (stub lato client, servicer lato server)

### Struttura di un Messaggio

- Un **messaggio** è un piccolo record logico con coppie nome-valore chiamate **campi**
- Ogni campo ha un **numero univoco** per l'identificazione nel formato binario

### Sintassi `.proto`

```protobuf
// ProductInfo.proto
syntax = "proto3";
package ecommerce;

service ProductInfo {
    rpc addProduct(Product)   returns (ProductID);
    rpc getProduct(ProductID) returns (Product);
}

message Product {
    string id          = 1;
    string name        = 2;
    string description = 3;
}

message ProductID {
    string value = 1;
}
```

> [!tip] Package
> Aggiungere sempre il `package` nel file `.proto` per evitare conflitti di naming e garantire portabilità tra linguaggi diversi. (In Python il package è ignorato a livello di runtime, ma è utile per altri linguaggi.)

---

## 🔧 Workflow gRPC in Python

### 4 Passi

**1. Installazione (una tantum):**
```bash
python -m pip install grpcio
python -m pip install grpcio-tools
```

**2. Definire servizio e messaggi nel file `.proto`** usando protobuf come IDL

**3. Generare stub client e classi server** con il compilatore `protoc`:
```bash
python -m grpc_tools.protoc -I. \
    --python_out=. \
    --pyi_out=. \
    --grpc_python_out=. \
    NOMESERVIZIO.proto
```

**4. Scrivere server e client** usando le gRPC API di Python

### File Generati

| File | Contenuto | Flag compilatore |
|---|---|---|
| `<SERVIZIO>_pb2.py` | Classi dei messaggi protobuf (serializzazione/deserializzazione) | `--python_out` |
| `<SERVIZIO>_pb2_grpc.py` | Classi stub client + classi servicer server (skeleton) | `--grpc_python_out` |

---

## 💡 Esempio Hello World

### 1. Definizione del servizio (`helloworld.proto`)

```protobuf
syntax = "proto3";
package helloworld;

service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name    = 1;
}

message HelloReply {
    string message = 1;
}
```

### 2. Compilazione

```bash
python -m grpc_tools.protoc --python_out=. --pyi_out=. --grpc_python_out=. helloworld.proto
```

Genera:
- `helloworld_pb2.py` → classi `HelloRequest`, `HelloReply`
- `helloworld_pb2_grpc.py` → `GreeterStub` (proxy client) + `GreeterServicer` (skeleton server)

### 3. Implementazione del Server

**a) Implementare l'interfaccia del servizio:**
```python
class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        # request → dati inviati dal client (HelloRequest)
        # context → metadati RPC (status code, etc.)
        return helloworld_pb2.HelloReply(message="Hello, %s!" % request.name)
```

**b) Creare ed avviare il server gRPC:**
```python
from concurrent import futures
import grpc

def serve():
    port = "50051"
    server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=10),
	        options=(('grpc.so_reuseport', 0),)   # evita socket duplicate sullo                                                     stesso IP:porta
    )
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port("[::]:" + port)
    server.start()
    print("Server started, listening on " + port)
    server.wait_for_termination()

if __name__ == "__main__":
    serve()
```

### 4. Implementazione del Client

```python
import grpc

def run():
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)          # proxy
        response = stub.SayHello(helloworld_pb2.HelloRequest(name="you"))
        print("Client received: " + response.message)
```

### Aggiornare un Servizio gRPC

1. Aggiungere il nuovo metodo al file `.proto`
2. Rigenerare il codice con `protoc`
3. Aggiornare il **server** per implementare il nuovo metodo
4. Aggiornare il **client** per invocare il nuovo metodo

---

## 📡 4 Tipi di RPC Call

| Tipo | Sintassi `.proto` | Direzione stream | Descrizione |
|---|---|---|---|
| **Unary** | `rpc M(Req) returns (Res)` | Nessuno | Una richiesta → una risposta (standard) |
| **Server Streaming** | `rpc M(Req) returns (stream Res)` | Server → Client | Una richiesta → flusso di risposte |
| **Client Streaming** | `rpc M(stream Req) returns (Res)` | Client → Server | Flusso di richieste → una risposta |
| **Bidirectional Streaming** | `rpc M(stream Req) returns (stream Res)` | Entrambi | Flussi indipendenti in entrambe le direzioni |

> [!important]
> gRPC **garantisce l'ordinamento dei messaggi** all'interno di una singola chiamata RPC.

---

## 🔁 Generatori Python e `yield`

Per implementare lo **streaming RPC** occorrono i **Generator** di Python.

> [!info] Generator
> Funzione che usa `yield` invece di `return`: **produce un valore senza interrompere la funzione**. Restituisce un iteratore su cui si può chiamare `next()`.

```python
def range2iter(n):
    """Generator dei quadrati da 0 a n-1."""
    for x in range(n):
        yield x * x

a = range2iter(3)
next(a)   # → 0
next(a)   # → 1
next(a)   # → 4
```

> [!warning]
> Usare `return` invece di `yield` interromperebbe la funzione al primo ciclo, restituendo solo il primo valore.

### Streaming Server (Server → Client)

**Server** usa `yield` per produrre più risposte:
```python
class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello_v1(self, request, context):
        for i in range(0, 5):
            yield helloworld_pb2.HelloReply(
	            message="Hello, " + request.name + "! -" + str(i))
```

**Client** itera sulle risposte:
```python
for response in stub.SayHello_v1(helloworld_pb2.HelloRequest(name="you")):
    print("[CLIENT] received: " + response.message)
```

### Streaming Client (Client → Server)

**Client** usa un generator per inviare più richieste:
```python
def generate_requests():
    names = ['Raf', 'Gigi', 'Pippo', 'Pippozzo']
    for name in names:
        yield helloworld_pb2.HelloRequest(name=name)

response = stub.SayHello_v2(generate_requests())
```

**Server** itera su `request_iterator`:
```python
def SayHello_v2(self, request_iterator, context):
    names = []
    for request in request_iterator:
        names.append(request.name)
    return helloworld_pb2.HelloReply(message="Hello, " + ' '.join(names) + "!")
```

### Bidirectional Streaming

**Server** usa sia `request_iterator` che `yield`:
```python
def SayHello_v3(self, request_iterator, context):
    for request in request_iterator:
        yield helloworld_pb2.HelloReply(message="Hello, " + request.name + "!")
```

**Client** usa sia un generator che itera sulle risposte:
```python
for response in stub.SayHello_v3(generate_requests()):
    print("[CLIENT] received: " + response.message)
```

---

## 🔒 Thread-Safety in gRPC

> [!important]
> - I **channel** creati con il modulo `grpc` sono **thread-safe**
> - Gli **stub** client generati sono **thread-safe**
> - La parte **server** (multithread) è comunque soggetta al **GIL** nonostante gRPC usi il package `concurrent`

---

## ⚠️ Errori Tipici in gRPC

| Errore                                             | Causa                                                      | Lato   |
| -------------------------------------------------- | ---------------------------------------------------------- | ------ |
| `StatusCode.UNAVAILABLE` – `Connection refused`    | Server non attivo o irraggiungibile                        | Client |
| `StatusCode.UNKNOWN` – `has no "wrongfield" field` | Campo inesistente nel messaggio protobuf (lato server)     | Client |
| `ValueError: has no "fieldname" field`             | Campo inesistente nel messaggio protobuf                   | Client |
| `'Order' object is not an iterator`                | Manca `yield` (generator) nel metodo server-side streaming | Client |
| `Exception iterating requests!`                    | Manca `yield` (generator) nel client-side streaming        | Client |

---

## 🚫 Limitazioni di gRPC

- **Supporto browser limitato**: HTTP/2 non completamente supportato; `gRPC-Web` disponibile ma con funzionalità ridotte (solo unary + server streaming limitato)
- **Formato non human-readable**: protobuf è binario; richiede strumenti aggiuntivi per debug e analisi dei payload

---

## 🔗 Collegamenti

- [[10 - Sistemi Middleware]]
- [[09 - Python Networking]]
- [[06 - Python OOP]]
- [[Pattern Proxy-Skeleton]]
- [[Generatori e Iteratori Python]]

---

## 📚 Risorse Utili

- [gRPC Core Concepts](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [gRPC Python Quickstart](https://grpc.io/docs/languages/python/quickstart/)
- [HTTP/2 Performance](https://web.dev/performance-http2)
- [gRPC Channel Options](https://github.com/grpc/grpc/blob/v1.71.x/include/grpc/impl/channel_arg_names.h)
