## Multithreading
Java supporta il **multithreading** a livello di linguaggio: di norma i thread sono implementati a livello di sistema, ma in Java è possibile creare programmi multithread in maniera **standardizzata**, cioè indipendentemente dalla piattaforma.

### Creazione di un Thread

Per creare un `Thread` in Java esistono due approcci:

- **Estensione della classe `Thread`**: si crea una classe che deriva da `Thread` e ridefinisce il metodo `run()`. Per istanziare l'oggetto si istanzia direttamente un oggetto di tipo `Thread` tramite il costruttore della classe che estende `Thread`.
- **Implementazione dell'interfaccia `Runnable`**: si implementa nella classe l'interfaccia `Runnable`, ridefinendo sempre il metodo `run()`. In questo caso, bisogna prima istanziare l'oggetto `Runnable` della classe che implementa l'interfaccia, e poi istanziare un oggetto `Thread` passandogli come parametro l'oggetto `Runnable` appena creato.

A differenza di Python, Java **non consente l'eredità multipla**, quindi:

- di base conviene usare la **derivazione da `Thread`**;
- se la classe è già coinvolta in un'eredità (estende un'altra classe), bisogna necessariamente usare l'**implementazione di `Runnable`**.

### Gestione della memoria tra Thread

In Java, le aree di memoria si comportano in modo diverso a seconda del tipo:

| Area di memoria | Condivisione tra Thread | Contenuto |
|---|---|---|
| **Method area** | Condivisa | Variabili statiche |
| **Heap** | Condivisa | Variabili d'istanza |
| **Stack** | Unico per ogni Thread | Frame (ogni frame rappresenta l'invocazione di un metodo) |

In Java, il **context switch** viene effettuato dalla **JVM**.

### Metodi principali di un Thread

I metodi fondamentali della classe `Thread` sono:

- **`start()`**: avvia il thread.
- **`run()`**: definisce il comportamento del thread.
- **`sleep()`**: sospende il thread per il periodo di tempo specificato (in millisecondi).
- **`yield()`**: a differenza di Python, serve a sospendere il thread a favore di un altro.

### Priorità dei Thread

I thread in Java hanno una **priorità**, espressa su **10 livelli** compresi tra `MIN_PRIORITY` e `MAX_PRIORITY` (rispettivamente 1 e 10), modificabile tramite `setPriority()`.

La JVM concede più tempo di processore ai thread con priorità più alta:

- se c'è più di un `Runnable` in attesa, viene scelto quello con **priorità maggiore** (a parità di priorità, la scelta è **FIFO**);
- il thread selezionato continua l'esecuzione finché non:
  - **finisce**;
  - invoca una `yield()`;
  - arriva un `Runnable` a priorità maggiore;
  - il suo **time-slicing** finisce;
  - va in stato **Blocked**.

#### Il problema dei thread egoisti (Selfish Thread)

Nel caso in cui ci siano più thread con la stessa priorità (e nessuno a priorità maggiore), si può incappare nel problema dei **thread egoisti** (*Selfish Thread*): un thread con un'esecuzione molto lunga non lascia mai spazio ad altri thread per eseguire.

Per ovviare a questo problema si può fare uso di:

- **Time-slicing**: impone un quanto di tempo massimo entro cui i thread possono eseguire prima di essere messi in stato `Blocked`.
- **`yield()`**: metodo alternativo utile a risolvere il problema, dato che non è detto che il sistema sia basato su time-slicing.

### Interruzione dei Thread

Un'**interruzione** indica che un thread deve fermarsi e fare altro, in base a cosa decide il programmatore.

- Alcuni metodi dei Thread invocano l'eccezione `InterruptedException` quando si chiama `interrupt()`, permettendo così di gestire il comportamento nel blocco `catch`.
- Se il metodo non genera automaticamente l'eccezione, si può programmare per eseguire periodicamente un controllo tramite `Thread.interrupted()`, che ritorna un valore **booleano**.
- Oltre a `interrupt()` esistono anche `suspend()`, `resume()` e `stop()` per gestire le interruzioni, anche se sono **sconsigliati**, poiché non garantiscono una corretta gestione delle risorse allocate e potrebbero generare un **deadlock**.

# Sincronizzazione

## Il modello a Monitor (synchronized)

In Java, la sincronizzazione è basata sui **mutex**: ne è associato uno ad ogni oggetto ed è possibile accedervi tramite **metodi** e **blocchi sincronizzati**.

- Quando un thread esegue un metodo o blocco sincronizzato, entra in possesso del **mutex** dell'oggetto, accedendo così alla **sezione critica**.
- Qualsiasi altro thread che voglia accedere alla stessa sezione critica viene posto in **stato di attesa**.
- Un metodo si definisce sincronizzato tramite la keyword `synchronized`.
- Esiste anche un **mutex associato alla classe stessa** (separato dai mutex delle singole istanze), accessibile tramite i **metodi sincronizzati statici**.
- L'ottenimento e il rilascio del mutex è **automatico** all'ingresso/uscita dal metodo sincronizzato.

### Blocco sincronizzato

Un blocco sincronizzato è una singola porzione di codice che delimita una sezione critica, definita come segue:

```java
synchronized(this) {
    // sezione critica
}
```

### Monitor

Una qualsiasi classe con metodi `synchronized` è detta **monitor**. Ogni monitor ha una singola (implicita) **condition variable**, che fornisce le primitive di:

- `wait()`
- `notify()` (o `notifyAll()`)

### Modello Signal and Continue

Il monitor che usa Java di default è un monitor **Signal and Continue**, quindi non può assicurarsi che al risveglio di un thread (tramite `notify()`), la sua condizione sia vera.

Per questo motivo bisogna utilizzare due insiemi:

- un set per i thread **pronti ad entrare** nel monitor (**entry set**);
- un set per i thread **in attesa** (**wait set**).

Quando viene invocata una `notify()`, un thread nel *wait set* passa all'*entry set* (o più di uno, in caso di `notifyAll()`).

### Limitazioni

Questi monitor, per quanto semplici da usare, hanno delle limitazioni, come ad esempio un'**unica coda di attesa**.

Esistono tre package che forniscono il supporto per implementare i paradigmi di programmazione concorrente più comuni:

- `java.util.concurrent`
- `java.util.concurrent.atomic`
- `java.util.concurrent.locks`

## La classe Semaphore

La classe `Semaphore` gestisce vari **permessi**:

| Metodo | Comportamento |
|---|---|
| `acquire()` | Blocca il thread se non c'è almeno un permesso disponibile |
| `release()` | Aggiunge un permesso al semaforo, sbloccando eventuali thread in attesa |
| `tryAcquire()` | Acquisisce un permesso solo se disponibile, senza bloccare il thread in caso di mancato permesso |

Un semaforo può essere inizializzato ad **1** (quindi **semaforo binario**) per funzionare come *lock*, che — a differenza di un `Lock` implementato tramite `java.util.concurrent.locks` — può essere **rilasciato da qualsiasi altro thread**.

## Il package java.util.concurrent.locks

### Interfaccia Lock

Un lock implementato da `java.util.concurrent.locks` permette un accesso flessibile alle risorse protette. Fornisce i seguenti metodi:

- `lock()`
- `tryLock()`
- `lockInterruptibly()`
- `unlock()`

Per usare uno di questi lock, si invoca il metodo `lock()` su un oggetto `Lock` e si definisce la sezione critica all'interno di un blocco `try`, ponendo successivamente l'`unlock()` nel blocco `finally`:

```java
lock.lock();
try {
    // sezione critica
} finally {
    lock.unlock();
}
```

### ReentrantLock

Esiste anche una classe `ReentrantLock` che si comporta però come i metodi e i blocchi `synchronized`.

### Interfaccia Condition

Tramite i lock si gestisce la **mutua esclusione**; inoltre, essi forniscono anche l'implementazione dell'interfaccia `Condition`, che istanzia oggetti che rappresentano **condition variables**, utili per gestire la sospensione e l'esecuzione di thread concorrenti in base al verificarsi di una condizione.

- Ogni istanza di `Condition` è legata ad un `Lock`.
- Si crea invocando `newCondition()` su un oggetto `Lock`:

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
```

## Networking

Per la comunicazione su rete, **Java** mette a disposizione le classi `Socket` e `ServerSocket` per la comunicazione **TCP**, e le classi `DatagramPacket` e `DatagramSocket` per la comunicazione **UDP**.

### Comunicazione TCP

- Il processo **Server** si pone in attesa di richieste su una porta tramite la classe `ServerSocket`:
  - si istanzia un oggetto `ServerSocket`, passando la porta al costruttore;
  - si chiama la primitiva `accept()`, che ritorna il `Socket` del client.
- Lato **client**, si istanzia un oggetto `Socket()`, passando al costruttore l'indirizzo e la porta del server: in automatico viene inviata la richiesta al server, che verrà accolta dalla `accept()`.
- La classe `Socket` gestisce la comunicazione **TCP**.
- Tramite i metodi `getInputStream()` e `getOutputStream()` di un oggetto `Socket`, si può ottenere il canale di comunicazione tra client e server, su cui scrivere e leggere tramite le classi `Reader` e `Writer`.

### Comunicazione UDP

Per implementare la comunicazione **UDP**, si usano le classi `DatagramPacket` e `DatagramSocket`.

- Un oggetto `DatagramPacket` rappresenta un **datagramma**, composto da indirizzo e dati.
- Un oggetto `DatagramSocket` viene utilizzato sia per **inviare** che per **ricevere** datagrammi, tramite i metodi `send()` e `receive()`:
  - poiché UDP è **connectionless**, lo stesso `DatagramSocket` può essere usato sia per ricevere che per inviare;
  - in base al costruttore utilizzato: se si specifica la porta, il socket serve anche a ricevere, altrimenti serve solo ad inviare.

### Gestione degli indirizzi IP (`InetAddress`)

Per gestire gli indirizzi IP in Java si usa la classe `InetAddress`, che espone i seguenti metodi:

| Metodo | Descrizione |
|---|---|
| `getLocalHost()` | Restituisce l'**IP della macchina locale** |
| `getByName()` | Restituisce l'**IP associato** a un host name |
| `getAddress()` | Restituisce l'**IP contenuto** in un oggetto |
| `getHostName()` | Restituisce il **nome della macchina locale** |

## JMS (Java Message Service)

**JMS** è un **MOM** (Message-Oriented Middleware) che definisce uno standard per l'invio e la ricezione di messaggi. Durante il corso viene utilizzato sempre tramite **Apache ActiveMQ**, allo stesso modo di STOMP.

- JMS permette il **disaccoppiamento** tra client e implementazione.
- Come per STOMP, esistono due **domini di messaggi**:
  - **Point-To-Point** (code)
  - **Publish-Subscribe** (topic)

### Interfacce principali

JMS definisce delle interfacce di alto livello per accedere ad entrambi i domini di messaggi:

| Interfaccia | Ruolo |
|---|---|
| **ConnectionFactory** | Crea una `Connection` |
| **Destination** | Incapsula la destinazione (coda o topic) |
| **Connection** | Collegamento al provider (es. ActiveMQ) |
| **Session** | Contesto per inviare e ricevere messaggi tramite `MessageProducer` e `MessageConsumer` |
| **MessageProducer** | Invia messaggi nel contesto di una `Session` |
| **MessageConsumer** | Riceve messaggi nel contesto di una `Session` |

Tutte queste interfacce hanno delle specifiche in base al dominio di messaggi che si vuole implementare. Una qualsiasi istanza di `ConnectionFactory` e `Destination` prende il nome di **administered object**.

## Pattern AbstractFactory e JNDI

Quando si vuole creare un sistema **indipendente da come i suoi prodotti sono realizzati**, e in cui esistono più famiglie di prodotti, l'ideale è usare il pattern **AbstractFactory**, che fornisce interfacce da implementare per creare:

- **Factory**: i provider (es. ActiveMQ)
- **Product**: qualsiasi interfaccia usata direttamente dai client (es. `Connection`, `Session`, `MessageProducer`/`Consumer`)

Un **administered object** (quindi un'istanza di `ConnectionFactory` o `Destination`) è un oggetto pre-configurato che colma il gap tra le interfacce di JMS e le specifiche del provider (ActiveMQ).

### Binding e lookup

- Quando un administered object viene creato, viene registrato in **JNDI** (*Java Naming and Directory Interface*), che gli associa un nome logico. Questa fase è detta **binding**.
- Un qualsiasi client può poi effettuare un **lookup** in JNDI per ottenere l'administered object a partire dal nome.
- JNDI è un **naming service**: mantiene un binding tra nome e riferimento a un oggetto.

Per utilizzarlo si usa **Context**, un'interfaccia che permette di aggiungere, cancellare e cercare oggetti in JNDI. I suoi metodi principali sono:

- `bind(nome, oggetto)`
- `rebind(nome, oggetto)`
- `lookup(nome)`
- `unbind(nome)`
- `rename(nome, nuovonome)`

## Flusso di un client JMS

Un client JMS, dopo aver configurato il servizio JNDI tramite le proprietà `java.naming.factory.initial` e `java.naming.provider.url`, deve seguire questi passaggi:

1. **Lookup** di una `ConnectionFactory` in JNDI
2. **Lookup** di una `Destination` in JNDI
3. Utilizzo della `ConnectionFactory` per creare (e successivamente avviare, in caso di receiver/subscriber) una `Connection`
4. Utilizzo della `Connection` per creare una `Session`
5. Utilizzo di una `Session` e della `Destination` per creare `MessageProducer` e `MessageConsumer`
6. Utilizzo di una `Session` per creare un `Message`
7. Invio/ricezione di messaggi
8. Cleanup delle risorse

**Ricapitolando:**

- **Connection**: la connessione tra client JMS e provider
- **Destination**: indica il tipo di dominio di messaggi (queue o topic)
- **Session**: un contesto **single-threaded** per l'invio e la ricezione di dati
- **MessageProducer/Consumer**: inviano e ricevono messaggi nel contesto di una `Session`

## Modalità di consumo dei messaggi

Un messaggio JMS può essere consumato in due modalità:

- **Sincrona**: la `receive` si blocca finché non arriva un messaggio.
- **Asincrona**: il client registra un `MessageListener`, che invoca il metodo `onMessage` ogni volta che arriva un messaggio (come in STOMP con Python).

### Receive sincrona

Si usa uno dei seguenti metodi:

```java
receive();
receive(timeout);
receiveNoWait();
```

### Receive asincrona

Non si usano metodi `receive()`; si crea invece una classe che implementa l'interfaccia `MessageListener`, contenente un solo metodo, `onMessage`, che rappresenta le istruzioni da eseguire ogni volta che viene rilevato un messaggio.

- Il client istanzia il listener.
- Lo registra sull'oggetto **receiver** (ovvero il `MessageConsumer` istanziato dalla `Session`) tramite il metodo:

```java
setMessageListener(listener);
```

## Struttura di un messaggio JMS

Un messaggio JMS è strutturato in tre parti:

### Header

Contiene:

- **Identificativo** del messaggio
- **Correlation ID** (usato ad esempio per collegare messaggi di richiesta a quelli di risposta)
- **Destinazione**
- **Priorità**

Accessibile tramite i metodi `getJMS*` e `setJMS*`.

### Properties

- Proprietà specifiche dell'applicazione o di JMS.
- Coppie **stringa-valore**.
- Gestibili tramite `set*Property` e `get*Property` (dove `*` è il tipo della proprietà).
- I nomi delle proprietà si ottengono tramite `getPropertyNames()`, che restituisce un elenco.

### Body

Contiene il messaggio vero e proprio, gestito con i metodi previsti dal tipo di body:

| Tipo di Body | Contenuto |
|---|---|
| **Text** | Stringhe |
| **Map** | Coppie nome-valore |
| **Bytes** | Stream di byte |
| **Stream** | Stream di tipi primitivi Java |
| **Object** | Oggetto serializzabile |

## Acknowledgement

La corretta procedura di consumo di un messaggio prevede alla fine anche l'**acknowledgement**, effettuato dal client o dal provider, in base alla configurazione della sessione:

- **AUTO_ACKNOWLEDGE**: invia l'ack automaticamente quando il client termina una `receive` o la `onMessage` di un `MessageListener`.
- **CLIENT_ACKNOWLEDGE**: richiede che il client invochi esplicitamente l'acknowledge sul messaggio consumato.
- **DUPS_OK_ACKNOWLEDGE**: forma "lasca" di ack che potrebbe generare duplicati in caso di malfunzionamento.

## Transazioni

È possibile raggruppare una serie di operazioni JMS in una **transazione**, ponendo il primo parametro di creazione di una sessione a `true`; in questo caso non ha senso parlare di acknowledgement, quindi il secondo parametro si lascia a `0`.

Una transazione (che esiste solo nel contesto di una `Session`) termina quando viene invocato `commit()` o `rollback()` sull'oggetto sessione:

- **Commit**:
  - Tutti i messaggi inviati diventano disponibili per la consegna.
  - Tutti i messaggi ricevuti vengono **acknowledged** (e, in caso di PTP, vengono anche rimossi dalle code).
- **Rollback**:
  - Tutti i messaggi inviati vengono scartati.
  - Tutti i messaggi ricevuti diventano nuovamente disponibili, quindi nessun acknowledge (in caso di PTP, i messaggi restano in coda).

## Concorrenza

JMS prevede la gestione dell'accesso concorrente su **administered objects** e oggetti **Connection**, mentre gli altri (quindi `Session` e `MessageProducer`/`Consumer`) **non sono progettati per l'uso multithread**, poiché la `Session` rappresenta un contesto single-threaded, per via della difficoltà di gestione di transazioni e ricezioni asincrone multithread.

## Modalità di consegna dei messaggi

Un messaggio in JMS può essere inviato in due modalità, configurabili tramite il metodo `setDeliveryMode` del producer oppure come parametro della `send`:

```java
DeliveryMode.PERSISTENT
DeliveryMode.NON_PERSISTENT
```

La differenza tra le due modalità è che, in modalità **PERSISTENT**, i messaggi sono registrati su memoria stabile e quindi, anche in caso di malfunzionamento, non vengono persi.

## Durable subscription

Come per STOMP, di default un subscriber riceve i messaggi pubblicati solo se è attivo (**non-durable subscription**). È possibile specificare un subscriber **durable** tramite il metodo:

```java
createDurableSubscriber(...)
```

invece di `createSubscriber`. Quando un topic JMS ha dei durable subscriber, conserverà i messaggi anche se non ci sono subscriber attivi in quel momento, finché tutti i durable subscriber non l'avranno ricevuto.

## Interoperabilità JMS-STOMP

Nel caso di comunicazione tra JMS e STOMP, bisogna tenere conto dei campi dell'header dei messaggi, che naturalmente sono diversi tra i due MOM.

- STOMP è un protocollo molto semplice e non riconosce i messaggi JMS come `TextMessage` o `BytesMessage`.
- Bisogna quindi utilizzare l'header **`content-length`**, che determina la lunghezza del contenuto:
  - Se questo header è **incluso** nel messaggio → si tratta di un **BytesMessage**.
  - Se questo header è **assente** → si tratta di un **TextMessage**.

# gRPC

## Concetti generali

I concetti di **channel**, **stub** e **protobuf** sono gli stessi visti in Python; tuttavia, l'uso delle API gRPC in **Java** risulta più complesso, in quanto prevede l'utilizzo di:

- **Costruttori** dedicati
- **Eccezioni checked**

## Struttura di un'applicazione gRPC in Java

Per ricapitolare, in Java è necessario implementare i seguenti componenti:

### 1. Protobuf

Il file `.proto` deve contenere la seguente riga subito dopo la dichiarazione `syntax`:

```proto
option java_multiple_files = true;
```

### 2. Client

Il client deve:

- **Creare** un *channel*
- **Istanziare** uno *stub* per effettuare le chiamate RPC

### 3. Server

Il server deve:

- **Creare** e **istanziare** un *servicer*
- **Avviare** un server gRPC
- **Bindare** il servicer al server, in modo da gestire le richieste in arrivo e inviare le relative risposte