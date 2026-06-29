# 02_JAVA_06 - Remote Procedure Call in Java (gRPC)

#java #advanced-computer-programming #gRPC #protobuf #RPC #channel #stub #StreamObserver #protoc

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte

---

## Concetti chiave

- **gRPC** (grpc.io): middleware RPC universale, ad alte prestazioni, open source, CNCF
- Framework **multilinguaggio** e **multipiattaforma** (Google, Dropbox, Netflix, Square, etcd, CockroachDB)
- Scenari: microservizi poliglotti, mobile/browser, librerie client efficienti
- Setup Java: compilatore **protoc** + plugin **protoc-gen-grpc-java 1.73.0** + librerie JAR
- Nota Protobuf per Java: `option java_multiple_files = true;`, campo `package` non ignorato
- Struttura applicazione: Protobuf, classe/i client-side, classe/i server-side, classe/i di implementazione
- **Tabella API client-side Java** vs Python (channel, stub, builder, chiamate, errori, cleanup)
- **Tabella API server-side Java** vs Python (server builder, addService, executor, shutdown hooks)
- Esempio Hello World completo: `HelloWorldServer`, `HelloWorldClient`, `GreeterImpl`
- Chiamata bloccante: `blockingStub.sayHello(request)` + catch `StatusRuntimeException`
- Risposta unary: `responseObserver.onNext(reply)` + `responseObserver.onCompleted()`

---

## gRPC: cos'e

**gRPC** (grpc.io) e un middleware RPC **universale**, ad **alte prestazioni** e **open source**, sviluppato da Google e ora sotto la *Cloud Native Computing Foundation* (CNCF).

- Puo essere eseguito in qualsiasi ambiente
- Framework **multilinguaggio** e **multipiattaforma**
- Usato da: Google, Dropbox, Netflix, Square, etcd, CockroachDB

### Principali scenari di utilizzo

- Collegare **microservizi** poliglotti con stile richiesta-risposta
- Connettere **dispositivi mobili** e browser ai servizi di backend
- Generare **librerie client efficienti**

---

## gRPC per Java: setup

### 1. Installare il compilatore `protoc`

**Linux/macOS:**

```bash
sudo unzip -o protoc-XX.X_linux-x86_64.zip -d /usr/local bin/protoc
sudo unzip -o XXX_linux-x86_64.zip -d /usr/local 'include/*'
```

Download da: https://github.com/protocolbuffers/protobuf/releases  
Versioni: `protoc-XX.X_linux-x86_64.zip`, `protoc-XX.X-linux-x86_32.zip`, `protoc-XX.X_osx-universal_binary.zip`

**Windows:**  
Scaricare `protoc-XX.X-win64` o `protoc-XX.X-win32`, decomprimere e aggiungere `bin/protoc` alla variabile `PATH`.

### 2. Compilare i file Protobuf

Scaricare il plugin **protoc-gen-grpc-java 1.73.0** da Maven:

| OS | URL |
|---|---|
| Linux | `protoc-gen-grpc-java-1.73.0-linux-x86_64.exe` |
| macOS | `protoc-gen-grpc-java-1.73.0-osx-x86_64.exe` |
| Windows | `protoc-gen-grpc-java-1.73.0-windows-x86_64.exe` |

> Dopo il download, dare i permessi di esecuzione: `chmod +x NOMEFILE`

```bash
protoc --java_out=. --grpc-java_out=. \
       --plugin=protoc-gen-grpc-java=/PATH/TO/protoc-gen-grpc-java-1.9.1-osx-x86_64 \
       nome_file.proto
```

> Su alcuni OS (es. macOS) vengono sollevate eccezioni di sicurezza: impostare il plugin come sicuro nelle preferenze di sistema.

### 3. Librerie JAR da includere nel classpath

Versioni testate per il corso (disponibili sul repo come JAR):

```
failureaccess-1.0
grpc-api-1.73.0
grpc-core-1.73.0
grpc-netty-shaded-1.73.0
grpc-protobuf-1.73.0
grpc-protobuf-lite-1.73.0
grpc-stub-1.73.0
guava-33.4.8-jre
perfmark-api-0.26.0
protobuf-java-4.31.1
```

> Per includere piu JAR: `-cp /path/folder/jar/files/*` seguito dal separatore (`:` Unix, `;` Windows)

---

## gRPC e API Java: concetti generali

I concetti di gRPC come **channel**, **stub** e **messaggi Protobuf** sono gli stessi descritti in Python. Tuttavia:
- Java e **piu verboso** nell'uso delle API gRPC
- Usa **costruttori** (pattern builder) ed **eccezioni checked**
- Sia Java che Python supportano chiamate RPC **bloccanti (sincrone)** e **non bloccanti (asincrone)**

### Struttura di una applicazione gRPC Java

| Componente | Responsabilita |
|---|---|
| **File Protobuf** | Definisce messaggi e servizi; va compilato con `protoc` |
| **Classe/i client-side** | Crea channel, istanzia stub, invoca RPC, gestisce risposte |
| **Classe/i server-side** | Crea server gRPC, fa bind con il servente, avvia il server |
| **Classe/i di implementazione** | Contiene la logica dei metodi RPC (estende `*ImplBase`) |

### Nota Protobuf per Java

- Aggiungere `option java_multiple_files = true;` dopo la riga `syntax` (non obbligatorio, ma migliora l'organizzazione delle classi generate)
- Il campo `package` **non e ignorato** in Java: determina il percorso del package dove verra generato il codice, a meno che non sia specificata l'opzione `java_package`
- Deve corrispondere allo stesso valore di `package` usato per le applicazioni Python

---

## API Client-Side Java vs Python

| Concetto | Java gRPC API | Note vs Python |
|---|---|---|
| **Import** | `import io.grpc.*` | Python: modulo `grpc` piu semplice e unificato |
| **Stub** | `MyServiceGrpc.MyServiceBlockingStub` / `MyServiceStub` | Java genera piu tipologie (bloccante, asincrono, future); Python ha un singolo stub |
| **Channel** | `Grpc.newChannelBuilder(target, InsecureChannelCredentials.create()).build()` | Java usa `ManagedChannel` con pattern builder; Python: `grpc.insecure_channel(...)` |
| **Messaggio di richiesta** | `MyRequest.newBuilder().setField(...).build()` | Java usa pattern builder immutabile; Python usa keyword arguments |
| **Chiamata bloccante** | `blockingStub.myMethod(request)` | Stessa semantica di Python, ma Java richiede tipo stub esplicito |
| **Chiamata asincrona** | `asyncStub.myMethod(request, responseObserver)` | Java usa `StreamObserver` (pattern observer); Python usa `.future()` o `asyncio` |
| **Gestione risposta** | `response.getField()` | Python: accesso diretto agli attributi (`response.field`) |
| **Gestione errori** | `catch (StatusRuntimeException e)` | Python: `try/except grpc.RpcError` |
| **Cleanup canale** | `channel.shutdownNow().awaitTermination(timeout, unit)` | Python: `channel.close()` piu semplice ma meno configurabile |

---

## API Server-Side Java vs Python

| Concetto | Java gRPC API | Note vs Python |
|---|---|---|
| **Istanziare un server** | `Grpc.newServerBuilderForPort(port, credentials)` — credentials: `InsecureServerCredentials` o `TlsServerCredentials` | Python: istanziazione diretta |
| **Aggiungere un servente** | `.addService(new MyServiceImpl())` | Java usa classe che estende `*ImplBase`; Python usa funzione helper |
| **Avviare il server** | `.build().start()` | Esplicito in entrambi; Java usa chaining con il builder |
| **Fermare il server** | `server.shutdown()` o `server.shutdownNow()` | Entrambi supportano shutdown graceful; Java ha controllo piu preciso |
| **Implementare un servizio** | Estende `MyServiceGrpc.MyServiceImplBase` + override metodi | Entrambi OOP |
| **Risposta unary** | `responseObserver.onNext(response)` + `responseObserver.onCompleted()` | Python usa `return` |
| **Risposta streaming** | Chiamate multiple a `onNext()`, poi `onCompleted()` | Python usa generator (stile pull); Java usa StreamObserver (stile push) |
| **Threading** | `ExecutorService` custom via `.executor(...)` (es. `FixedThreadPool`) | Java offre maggiore controllo tramite `ExecutorService` |
| **Gestione terminazione** | `Runtime.getRuntime().addShutdownHook(...)` | Python usa signal handler o context manager |
| **Gestione errori** | `responseObserver.onError(StatusException)` | Entrambi espongono codici di stato gRPC |

---

## Esempio Hello World (server-side)

```java
public class HelloWorldServer {
    private Server server;

    private void start() throws IOException {
        int port = 0;
        ExecutorService executor = Executors.newFixedThreadPool(2);

        server = Grpc.newServerBuilderForPort(port, InsecureServerCredentials.create())
                .executor(executor)
                .addService(new GreeterImpl())
                .build()
                .start();

        System.out.println("Server started, listening on " + server.getPort());

        try {
            server.awaitTermination();   // blocca il thread finche il server non viene fermato
        } catch (InterruptedException e) {
            // gestione interruzione
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        final HelloWorldServer server = new HelloWorldServer();
        server.start();
    }
}
```

**Forma alternativa (builder esplicito):**

```java
ServerBuilder<?> serverBuilder = Grpc.newServerBuilderForPort(port, InsecureServerCredentials.create());
serverBuilder.executor(executor);
serverBuilder.addService(new GreeterImpl());
server = serverBuilder.build();
server.start();
```

---

## Esempio Hello World (client-side)

```java
public class HelloWorldClient {
    private final GreeterGrpc.GreeterBlockingStub blockingStub;

    // Il channel viene passato al costruttore: non e responsabilita del client chiuderlo
    public HelloWorldClient(Channel channel) {
        blockingStub = GreeterGrpc.newBlockingStub(channel);
    }

    public void greet(String name) {
        System.out.println("Will try to greet " + name + " ...");

        // Costruzione del messaggio di richiesta tramite builder
        HelloRequest request = HelloRequest.newBuilder().setName(name).build();
        HelloReply response;

        try {
            response = blockingStub.sayHello(request);   // chiamata bloccante
        } catch (StatusRuntimeException e) {
            System.out.println("RPC failed: " + e.getStatus());
            return;
        }

        System.out.println("Greeting: " + response.getMessage());
    }

    public static void main(String[] args) throws Exception {
        String user = "world";
        String target = "localhost:" + args[0];

        // Crea il ManagedChannel (responsabile della chiusura)
        ManagedChannel channel = Grpc.newChannelBuilder(target, InsecureChannelCredentials.create()).build();

        try {
            HelloWorldClient client = new HelloWorldClient(channel);
            client.greet(user);
        } finally {
            channel.shutdownNow().awaitTermination(5, TimeUnit.SECONDS);
        }
    }
}
```

---

## Esempio Hello World (implementazione)

```java
public class GreeterImpl extends GreeterGrpc.GreeterImplBase {

    @Override
    public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
        System.out.println("[SERVER JAVA GRPC] Invoked sayHello");

        HelloReply reply = HelloReply.newBuilder()
                .setMessage("Hello " + req.getName())
                .build();

        System.out.println("[SERVER JAVA GRPC] Set reply and send...");

        responseObserver.onNext(reply);       // invia la risposta
        responseObserver.onCompleted();       // segnala la fine della risposta
    }
}
```

### Punti chiave dell'implementazione

| Elemento | Dettaglio |
|---|---|
| `extends GreeterGrpc.GreeterImplBase` | La classe generata da protoc da cui si deve ereditare |
| `@Override` | Si fa override dei metodi generati dall'interfaccia protobuf |
| `req.getName()` | Accesso al campo del messaggio tramite getter generato da protoc |
| `HelloReply.newBuilder().setMessage(...).build()` | Costruzione immutabile del messaggio di risposta |
| `responseObserver.onNext(reply)` | Invio della risposta al client |
| `responseObserver.onCompleted()` | Notifica che la risposta e completa (obbligatorio per chiamate unary) |

---

## Confronto Java vs Python per gRPC

| Aspetto | Java | Python |
|---|---|---|
| Verbosita | Alta (builder, costruttori, cast espliciti) | Bassa |
| Tipi stub | Multipli (`BlockingStub`, `AsyncStub`, `FutureStub`) | Uno solo |
| Accesso attributi messaggio | Metodi getter (`getName()`, `getMessage()`) | Accesso diretto (`req.name`) |
| Chiamata bloccante | `blockingStub.method(request)` | `stub.method(request)` |
| Chiamata asincrona | `asyncStub.method(req, StreamObserver)` | `.future()` o `asyncio` |
| Risposta unary | `onNext()` + `onCompleted()` | `return response` |
| Risposta streaming | Multipli `onNext()` + `onCompleted()` | `yield` (generator) |
| Gestione errori | `StatusRuntimeException` (checked) | `grpc.RpcError` |
| Chiusura channel | `channel.shutdownNow().awaitTermination(...)` | `channel.close()` |
| Threading server | `ExecutorService` via `.executor(...)` | Configurazione `ThreadPoolExecutor` |

---

## Collegamento con slide precedenti

- [[01_PYTHON_11 - Python RPC]]: stessi concetti gRPC in Python (channel, stub, protobuf)
- [[02_JAVA_04 - Proxy Skeleton]]: pattern stub/skeleton a basso livello vs stub gRPC generati
- [[02_JAVA_01 - Multithreading Java]]: `ExecutorService`, thread pool per il server gRPC
