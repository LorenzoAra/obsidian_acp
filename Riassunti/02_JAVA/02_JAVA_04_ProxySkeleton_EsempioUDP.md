# 02_JAVA_04 - Realizzazione di Oggetti Remoti e Proxy-Skeleton

#java #advanced-computer-programming #proxy #skeleton #udp #socket #oggetti-remoti #design-pattern #rpc

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte

---

## Concetti chiave

- Problema delle applicazioni client-server con socket: la logica di comunicazione si mescola alla logica applicativa
- Soluzione: separazione tramite **Proxy** lato client e **Skeleton** lato server
- Pattern Proxy: implementazione locale al client del servizio remoto
- Skeleton: ricezione richieste, parsing dati, up-call al servizio reale, invio risposta
- Due varianti di Skeleton: **per ereditarieta** e **per delega**
- Esempio completo: **Contatore Remoto** con socket **UDP**
- Serializzazione manuale dei messaggi UDP tramite `String` e separatore `#`
- Uso di `StringTokenizer`, `Integer.valueOf()`, `Integer.toString()`
- Classi principali: `ICounter`, `CounterProxy`, `CounterSkeleton`, `CounterImpl`, `CounterWorker`

---

## Problema e motivazione

In un'applicazione con comunicazione basata su socket, client e server devono gestire connessione, esternalizzazione dei dati, invio-ricezione delle richieste e ricostruzione dei valori ricevuti. Questo puo distrarre dalla realizzazione delle funzionalita effettive dell'applicazione e creare sovrapposizione tra logica applicativa e logica di comunicazione client-server.

L'idea e quindi separare i due livelli:
- lato client, il programmatore si concentra sull'invocazione dei servizi descritti da un'interfaccia;
- lato server, il programmatore si concentra sulla codifica dei servizi offerti.

---

## Esempio: Contatore Remoto

Il server gestisce un oggetto `Contatore`, cioe il **servizio reale**, che permette inizializzazione, incremento e aggiornamento di una variabile contatore. Il client invoca i servizi offerti dal server tramite una specifica interfaccia Java.

```java
public interface ICounter {
    public void setCount(String id, int s);
    public int sum(int s);
    public int increment();
}
```

| Metodo | Significato |
|---|---|
| `setCount(String id, int s)` | Imposta il contatore al valore iniziale `s` per il client identificato da `id` |
| `sum(int s)` | Incrementa il contatore di `s` e restituisce il valore aggiornato |
| `increment()` | Incrementa il contatore di `1` e restituisce il valore aggiornato |

---

## Pattern Proxy

Il pattern introduce nel modello client-server un **Proxy** (detto anche *Surrogato* o *Ambassador*):
- il Proxy si presenta come implementazione del servizio remoto, ma e **locale al client**;
- i meccanismi di basso livello necessari per instaurare la comunicazione e per inviare/ricevere i parametri tramite TCP o UDP sono **incapsulati nel Proxy**.

### Vista statica

- `InterfacciaServer` specifica il servizio fornito.
- `ServerImpl` implementa effettivamente i servizi dichiarati nell'interfaccia.
- `ServerProxy` si fa carico dell'inoltro delle richieste dal client verso il lato server.
- L'associazione direzionale tra `ServerProxy` e `ServerImpl` sintetizza tutti i meccanismi necessari al proxy per riferirsi al server.

### Uso del Proxy

Il client possiede un riferimento a un oggetto di tipo `InterfacciaServer`, che nella pratica e il `ServerProxy`. Quando il client invoca un servizio, contatta direttamente il proxy; poi il proxy contatta il server e restituisce al client eventuali valori di ritorno.

---

## Ruolo dello Skeleton

Con l'aggiunta del Proxy il client e sollevato dalle problematiche di comunicazione. In maniera analoga, un proxy lato server detto **Skeleton** si fa carico della comunicazione con il Proxy lato client.

Lo Skeleton:
1. riceve le richieste di servizio;
2. struttura l'informazione ricevuta tramite socket TCP o UDP;
3. effettua l'**up-call** al servizio reale;
4. riceve eventuali risultati;
5. invia una risposta al proxy lato client.

### Schema concettuale

```text
Programma cliente -> Proxy -> Software di rete -> Software di rete -> Skeleton -> Oggetto remoto
                  <-       Messaggio di risposta       <-
                  ->      Messaggio di invocazione     ->
```

Il client effettua una chiamata locale al Proxy, che invia un messaggio di invocazione sulla rete. Sul lato server, lo Skeleton riceve il messaggio, effettua una chiamata locale all'oggetto remoto e poi ritorna la risposta al Proxy.

---

## Skeleton per ereditarieta

Lo Skeleton puo essere implementato per **ereditarieta**:

- una classe astratta `ServerSkeleton` implementa solo gli opportuni schemi di comunicazione;
- i metodi dell'interfaccia restano senza implementazione;
- `ServerImpl` estende `ServerSkeleton` e fornisce l'implementazione dei metodi astratti.

Schema concettuale:

```text
InterfacciaServer <<interface>>
        ^
        |
ServerSkeleton (abstract class)
        ^
        |
    ServerImpl
```

---

## Skeleton per delega

Lo Skeleton puo anche essere implementato per **delega**:

- la classe `ServerSkeleton` contiene un riferimento a `InterfacciaServer`, chiamato `delegate`;
- i metodi dello skeleton inoltrano la chiamata al delegate.

Esempio:

```java
void Servizio1() { delegate.Servizio1(); }
```

Schema concettuale:

```text
ServerSkeleton --delegate--> ServerImpl
```

---

## Architettura dell'esempio UDP

Il programma client-server che realizza un contatore remoto tramite Proxy-Skeleton e socket UDP e composto da queste classi:

| Classe | Ruolo |
|---|---|
| `Client` | Contiene `main()` lato client |
| `Server` | Contiene `main()` lato server |
| `ICounter` | Interfaccia del servizio remoto |
| `CounterProxy` | Proxy lato client, `implements ICounter` |
| `CounterSkeleton` | Skeleton lato server, `implements ICounter` |
| `CounterImpl` | Implementazione reale del contatore |
| `CounterWorker` | Thread worker per elaborare le richieste UDP |

---

## Main program

### Client

```java
public static void main(String[] args) {
    ICounter counter = new CounterProxy();
    int x = 0;

    System.out.println("set count to 10");
    counter.setCount("user-ACP", 10);

    System.out.print("sum 15 to count; ");
    x = counter.sum(15);
    System.out.println("current count: " + x);

    System.out.print("increment count; ");
    x = counter.increment();
    System.out.println("current count: " + x);
}
```

### Server

```java
public static void main(String[] args) {
    CounterImpl counter = new CounterImpl();
    counter.runSkeleton();
}
```

Nel client il riferimento e al **proxy**; nel server viene attivato l'oggetto remoto tramite `CounterImpl` e `runSkeleton()`.

---

## Proxy UDP: `CounterProxy`

Il proxy implementa `ICounter` e incapsula tutta la logica di comunicazione UDP.

### Costruttore

```java
private DatagramSocket socket;

public CounterProxy() {
    try {
        socket = new DatagramSocket();
    } catch (SocketException e) {
        e.printStackTrace();
    }
}
```

### `setCount`

```java
public void setCount(String id, int s) {
    String message = new String("setCount#" + id + "#" + s + "#");
    try {
        DatagramPacket request = new DatagramPacket(
            message.getBytes(),
            message.getBytes().length,
            InetAddress.getLocalHost(),
            9000
        );
        socket.send(request);

        byte[] buffer = new byte[100];
        DatagramPacket reply = new DatagramPacket(buffer, buffer.length);
        socket.receive(reply);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### `sum`

```java
public int sum(int s) {
    int x = 0;
    String message = new String("sum#" + s + "#");
    try {
        DatagramPacket request = new DatagramPacket(
            message.getBytes(),
            message.getBytes().length,
            InetAddress.getLocalHost(),
            9000
        );
        socket.send(request);

        byte[] buffer = new byte[100];
        DatagramPacket reply = new DatagramPacket(buffer, buffer.length);
        socket.receive(reply);

        String replyMessage = new String(reply.getData(), 0, reply.getLength());
        x = Integer.valueOf(replyMessage).intValue();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return x;
}
```

### `increment`

```java
public int increment() {
    int x = 0;
    String message = new String("increment#");
    try {
        DatagramPacket request = new DatagramPacket(
            message.getBytes(),
            message.getBytes().length,
            InetAddress.getLocalHost(),
            9000
        );
        socket.send(request);

        byte[] buffer = new byte[100];
        DatagramPacket reply = new DatagramPacket(buffer, buffer.length);
        socket.receive(reply);

        String replyMessage = new String(reply.getData(), 0, reply.getLength());
        x = Integer.valueOf(replyMessage).intValue();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return x;
}
```

---

## Gestione dei pacchetti UDP

Il payload del messaggio UDP e costituito da un **array di byte**. Serve quindi adattare il nome del metodo e gli eventuali parametri dell'invocazione al pacchetto di richiesta.

### Strategia usata nelle slide

Creare una `String` concatenando:
- nome del metodo
- singoli parametri
- un carattere separatore, per esempio `#`

Formato generale:

```text
nomeMetodo#argomento1#argomento2#argomentoN#
```

Esempio:

```java
String message = new String("setCount#" + id + "#" + s + "#");
```

La stringa viene poi inserita nel pacchetto UDP tramite `getBytes()`.

### Lato ricevente

Il ricevente:
- estrae la `String` dal pacchetto UDP ricevuto;
- individua nome metodo e parametri tramite `StringTokenizer` e `nextToken()`;
- converte i parametri da `String` al tipo richiesto tramite i metodi `valueOf()` delle classi Wrapper.

Per esempio, per un `int`:

```java
int x = Integer.valueOf(messageTokens.nextToken()).intValue();
```

### `StringTokenizer` e Wrapper

- `StringTokenizer` scompone una stringa in token delimitati dal separatore scelto.
- Invocazioni successive di `nextToken()` restituiscono i token uno alla volta.
- Per esempio: `setCount#user-ACP#10#` viene scomposto in `setCount`, `user-ACP`, `10`.

Metodi utili delle Wrapper:

```java
public static String toString(int i)
static Integer valueOf(String s)
```

---

## Skeleton UDP per ereditarieta

`CounterSkeleton` e una classe astratta che implementa l'infrastruttura di comunicazione UDP:

```java
public abstract class CounterSkeleton implements ICounter {
    public void runSkeleton() {
        try {
            DatagramSocket socket = new DatagramSocket(9000);
            System.out.println("[CounterSkeleton] Entering main loop...");
            while (true) {
                byte[] buffer = new byte[100];
                DatagramPacket request = new DatagramPacket(buffer, buffer.length);
                socket.receive(request);

                CounterWorker worker = new CounterWorker(socket, request, this);
                worker.start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Il buffer va dimensionato in base ai dati scambiati; la dimensione massima indicata nelle slide e **65508** byte.

---

## `CounterWorker`

`CounterWorker` estende `Thread` e si occupa del parsing del pacchetto, della dispatch della richiesta e dell'invio della risposta.

### Avvio e parsing

```java
public class CounterWorker extends Thread {
    private DatagramSocket socket;
    private DatagramPacket request;
    private ICounter skeleton;

    public CounterWorker(DatagramSocket s, DatagramPacket r, ICounter sk) {
        socket = s;
        request = r;
        skeleton = sk;
    }

    public void run() {
        String message = new String(request.getData(), 0, request.getLength());
        StringTokenizer messageTokens = new StringTokenizer(message, "#");
        String method = messageTokens.nextToken();
```

- Il primo token e il **nome del metodo**.
- I token successivi sono i parametri dell'invocazione.

### Dispatch delle richieste

#### Caso `setCount`

```java
if (method.compareTo("setCount") == 0) {
    String id = messageTokens.nextToken();
    int x = Integer.valueOf(messageTokens.nextToken()).intValue();
    skeleton.setCount(id, x);   // upcall
    String replyMessage = "ack";
    DatagramPacket reply = new DatagramPacket(
        replyMessage.getBytes(),
        replyMessage.getBytes().length,
        request.getAddress(),
        request.getPort()
    );
    socket.send(reply);
}
```

#### Caso `sum`

```java
else if (method.compareTo("sum") == 0) {
    int x = Integer.valueOf(messageTokens.nextToken()).intValue();
    int res = skeleton.sum(x);  // upcall
    String replyMessage = Integer.toString(res);
    DatagramPacket reply = new DatagramPacket(
        replyMessage.getBytes(),
        replyMessage.getBytes().length,
        request.getAddress(),
        request.getPort()
    );
    socket.send(reply);
}
```

#### Caso `increment`

```java
else if (method.compareTo("increment") == 0) {
    int res = skeleton.increment();
    String replyMessage = Integer.toString(res);
    DatagramPacket reply = new DatagramPacket(
        replyMessage.getBytes(),
        replyMessage.getBytes().length,
        request.getAddress(),
        request.getPort()
    );
    socket.send(reply);
} else
    System.out.println("* BAD METHOD! *");
```

Qui si vede bene il ruolo dello skeleton/worker:
- parse del messaggio;
- **upcall** al servizio reale;
- creazione e invio del messaggio di risposta.

---

## `CounterImpl`

`CounterImpl` estende `CounterSkeleton` e contiene l'implementazione reale del servizio:

```java
public class CounterImpl extends CounterSkeleton {
    private int count;

    public void setCount(String id, int s) {
        System.out.println("[CounterImpl]: count set to " + s + " by client " + id + ".");
        count = s;
    }

    public int sum(int s) {
        System.out.println("[CounterImpl]: adding " + s + " to count.");
        count = count + s;
        return count;
    }

    public int increment() {
        System.out.println("[CounterImpl]: adding 1 to count.");
        count = count + 1;
        return count;
    }
}
```

**Nota importante:** eventuali politiche di sincronizzazione per disciplinare l'accesso ai servizi remoti vanno implementate nella classe `*Impl`, cioe nell'implementazione reale dei servizi dichiarati nell'interfaccia.

---

## Flusso completo della chiamata

Per una chiamata come `counter.sum(15)` il flusso e:
1. Il client invoca `sum(15)` sul `CounterProxy`.
2. Il proxy crea il messaggio `sum#15#`.
3. Il proxy invia il datagramma UDP verso la porta `9000`.
4. `CounterSkeleton` riceve il pacchetto.
5. `CounterWorker` estrae il metodo `sum` e il parametro `15`.
6. Il worker effettua l'upcall `skeleton.sum(15)`.
7. `CounterImpl` aggiorna il contatore e restituisce il risultato.
8. Il worker serializza il risultato come stringa e lo invia al client.
9. Il proxy riceve la risposta, la converte con `Integer.valueOf(...)` e la restituisce al client.

---

## Collegamenti utili

- [[02_JAVA_03 - Java Networking]]
- [[02_JAVA_01 - Multithreading Java]]
- [[02_JAVA_02 - Sincronizzazione Java]]
- [[01_PYTHON_11 - Python RPC]]
