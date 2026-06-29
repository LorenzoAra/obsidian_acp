# 02_JAVA_05 - Java Message Service (JMS)

#java #advanced-computer-programming #JMS #MOM #messaging #ActiveMQ #PTP #PubSub #JNDI #STOMP

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- Kim Haase, *Java Message Service API Tutorial*, cap. 1-5 — [versione online](http://docs.oracle.com/javaee/1.3/jms/tutorial/)

---

## Concetti chiave

- JMS (JSR 914): standard Java per l'accesso a prodotti MOM
- Provider JMS usato nel corso: **Apache ActiveMQ 5.16.6**
- JMS disaccoppia client da implementazione del provider (Abstract Factory pattern)
- Domini di messaging: **Point-to-Point (PTP)** e **Publish-Subscribe**
- Interfacce principali: `ConnectionFactory`, `Destination`, `Connection`, `Session`, `MessageProducer`, `MessageConsumer`
- Administered objects: `ConnectionFactory` e `Destination` recuperati tramite **JNDI**
- Modello di programmazione: 8 passi (lookup → connessione → sessione → producer/consumer → messaggio → invio/ricezione → cleanup)
- Ricezione **sincrona** (`receive()`) vs **asincrona** (`MessageListener.onMessage()`)
- Tipologie di messaggi: `TextMessage`, `MapMessage`, `BytesMessage`, `StreamMessage`, `ObjectMessage`
- **Acknowledgement**: `AUTO_ACKNOWLEDGE`, `CLIENT_ACKNOWLEDGE`, `DUPS_OK_ACKNOWLEDGE`
- Sessioni **transacted**: `commit()`, `rollback()`
- **Persistenza**: `PERSISTENT` (default), `NON_PERSISTENT`
- **Durable Subscription**: `createDurableSubscriber()`, coppia `clientID` + nome sottoscrizione
- Estensioni STOMP/ActiveMQ: header `client-id`, `activemq.subscriptionName`, `content-length`

---

## JMS: definizione e motivazione

**JMS** (Java Message Service, JSR 914) e uno standard Java che definisce un insieme di interfacce per permettere ai programmi Java di inviare e ricevere messaggi tramite un prodotto **MOM** (Message Oriented Middleware):

> "JMS is a set of interfaces and associated semantics that define how a JMS client accesses the facilities of an enterprise messaging product."

**Vantaggi:**
- Un programma scritto con JMS puo essere eseguito con **qualsiasi MOM** che implementi le interfacce JMS.
- La specifica JMS consente il **disaccoppiamento** tra client (sender e receiver) e specifica implementazione del provider.

### Provider: Apache ActiveMQ

```text
Java Application
    |
JMS API (javax.jms.*)
    |
Active MQ
    |
Provider-dependent interface
```

Versione del corso: **Apache ActiveMQ 5.16.6**  
Download: https://activemq.apache.org/components/classic/documentation/download-archives

---

## Domini di Messaging

### Point-to-Point (PTP)

- Ruota intorno al concetto di **coda di messaggi**
- Ogni messaggio ha **un solo consumer**
- La coda conserva il messaggio finche non scade o viene consumato
- Il receiver conferma (ack) la corretta ricezione

```
Client1 --sends--> Queue --consumes--> Client2
              (acknowledges)
```

### Publish-Subscribe (Pub/Sub)

- Ogni messaggio e associato a un **topic**
- Ogni messaggio puo avere **piu consumer** (0..N subscriber)
- I topic conservano il messaggio finche non viene rilasciato ai subscriber correnti

```
Client1 --publishes--> Topic --delivers--> Client2
Client3 --subscribes-+        +-delivers--> Client3
```

---

## Interfacce JMS

### Interfacce di alto livello

| Interfaccia | Ruolo |
|---|---|
| `ConnectionFactory` | Crea una `Connection` (`QueueConnectionFactory` o `TopicConnectionFactory`) |
| `Destination` | Incapsula la destinazione: coda o topic |
| `Connection` | Connessione verso il provider JMS |
| `Session` | Contesto single-threaded per inviare o ricevere messaggi |
| `MessageProducer` | Invia messaggi |
| `MessageConsumer` | Riceve messaggi |

`ConnectionFactory` e `Destination` sono detti **administered objects**.

### Specializzazione per dominio

| Interfaccia generale | Dominio PTP | Dominio Pub/Sub |
|---|---|---|
| `ConnectionFactory` | `QueueConnectionFactory` | `TopicConnectionFactory` |
| `Connection` | `QueueConnection` | `TopicConnection` |
| `Destination` | `Queue` | `Topic` |
| `Session` | `QueueSession` | `TopicSession` |
| `MessageProducer` | `QueueSender` | `TopicPublisher` |
| `MessageConsumer` | `QueueReceiver`, `QueueBrowser` | `TopicSubscriber` |

---

## Pattern Abstract Factory

Le interfacce JMS come `QueueConnectionFactory`, `QueueConnection`, `Queue`, `QueueSession` sono interfacce standard. Le implementazioni concrete sono fornite dal **provider** (es. ActiveMQ).

Il pattern **Abstract Factory** e usato perche:
- il sistema deve essere indipendente da come i prodotti sono creati;
- deve poter essere configurato con una di tante famiglie di prodotti;
- si vuole rivelare l'interfaccia ma non l'implementazione.

---

## Entita di un sistema JMS

| Entita | Descrizione |
|---|---|
| **Client** | Programmi Java che inviano/ricevono messaggi |
| **Provider** | Implementazione concreta JMS; supporta lo scambio messaggi e fornisce funzionalita di gestione |
| **Messaggi** | Astrazioni delle informazioni scambiate |
| **Administered Objects** | Istanze di `ConnectionFactory` e `Destination`; precconfigurati, recuperati tramite JNDI |

Il provider JMS fornisce programmi per creare gli administered objects e registrarli nel **JNDI** (Java Naming and Directory Interface).

---

## Naming Service e JNDI

Un naming service mantiene binding tra nomi e riferimenti a oggetti.

**Concetti JNDI:**
- **Name**: nome dato all'oggetto registrato
- **Binding**: associazione nome-oggetto
- **Reference**: puntatore a un oggetto
- **Context**: insieme di associazioni nome-oggetto

JNDI e indipendente dallo specifico servizio di naming (accesso tramite plugin detti *Service Provider*).

### Utilizzo di JNDI

```java
// Proprietà da configurare
java.naming.factory.initial
java.naming.provider.url

// Interfaccia Context: rappresenta un insieme di binding
// InitialContext è un'implementazione di Context
```

### Metodi principali di Context

| Metodo | Descrizione |
|---|---|
| `bind(String name, Object obj)` | Associa un nome a un oggetto (il nome non deve esistere gia) |
| `rebind(String name, Object obj)` | Come `bind`, sovrascrive se gia esistente |
| `lookup(String name)` | Cerca e restituisce l'oggetto con quel nome |
| `unbind(String name)` | Rimuove il binding |
| `rename(String name, String newName)` | Rinomina un binding |

---

## Modello di Programmazione JMS (8 passi)

```text
1. lookup di una ConnectionFactory da JNDI
2. lookup di una Destination da JNDI
3. creazione di una Connection tramite ConnectionFactory
   3.1. avvio della Connection (solo per Message Consumer!)
4. creazione di una Session tramite Connection
5. creazione di MessageProducer e/o MessageConsumer tramite Session e Destination
6. creazione di un Message tramite Session
7. invio e ricezione di messaggi
8. cleanup delle risorse (close)
```

> Prima dell'avvio del client JMS bisogna avviare il provider: questo attiva il servizio JNDI da cui il client potra recuperare `ConnectionFactory` e `Destination`.

---

## Lookup degli Administered Objects

```java
import java.util.Hashtable;
import javax.jms.*;
import javax.naming.*;

Hashtable<String, String> prop = new Hashtable<>();
prop.put("java.naming.factory.initial",
         "org.apache.activemq.jndi.ActiveMQInitialContextFactory");
prop.put("java.naming.provider.url", "tcp://127.0.0.1:61616");
prop.put("queue.test", "mytestqueue");   // jndi-queue-name, physical-queue-name

Context jndiContext = new InitialContext(prop);
```

---

## Esempio Sender (dominio PTP)

```java
// 1. Lookup ConnectionFactory
QueueConnectionFactory queueConnFactory =
    (QueueConnectionFactory) jndiContext.lookup("QueueConnectionFactory");

// 2. Lookup Destination (Queue)
Queue queue = (Queue) jndiContext.lookup("test");

// 3. Creo la Connection
QueueConnection queueConn = queueConnFactory.createQueueConnection();

// 4. Creo la Session (single-threaded)
QueueSession queueSession = queueConn.createQueueSession(
    false, Session.AUTO_ACKNOWLEDGE);

// 5. Creo il sender (MessageProducer)
QueueSender sender = queueSession.createSender(queue);

// 6. Creo il messaggio
TextMessage message = queueSession.createTextMessage();

// 7. Invio messaggi
for (int i = 0; i < 5; i++) {
    message.setText("hello_" + i);
    sender.send(message);
}
message.setText("fine");
sender.send(message);
System.out.println("I messaggi sono stati inviati!");

// 8. Cleanup
sender.close();
queueSession.close();
queueConn.close();
```

---

## Esempio Receiver (dominio PTP)

```java
// passi 1 e 2: identici al Sender

// 3. Creo la Connection
QueueConnection queueConn = queueConnFactory.createQueueConnection();

// 3.1. AVVIO la Connection (OBBLIGATORIO per Message Consumer)
queueConn.start();   // abilita il delivery dei messaggi!

// 4. Creo la Session
QueueSession queueSession = queueConn.createQueueSession(
    false, Session.AUTO_ACKNOWLEDGE);

// 5. Creo il receiver (MessageConsumer)
QueueReceiver receiver = queueSession.createReceiver(queue);

// 6-7. Ricezione (bloccante)
TextMessage message;
do {
    System.out.println("In attesa di messaggi!");
    message = (TextMessage) receiver.receive();
    System.out.println("+ messaggio ricevuto: " + message.getText());
} while (message.getText().compareTo("fine") != 0);

// 8. Cleanup
receiver.close();
queueSession.close();
queueConn.close();
```

---

## Esempio Publisher (dominio Pub/Sub)

```java
TopicConnectionFactory connFactory =
    (TopicConnectionFactory) jndiContext.lookup("TopicConnectionFactory");
Topic topic = (Topic) jndiContext.lookup("test");

TopicConnection topicConn = connFactory.createTopicConnection();
TopicSession topicSession = topicConn.createTopicSession(
    false, Session.AUTO_ACKNOWLEDGE);

TopicPublisher pub = topicSession.createPublisher(topic);
TextMessage text = topicSession.createTextMessage();

for (int i = 0; i < 5; i++) {
    text.setText("hello_" + i);
    pub.publish(text);
}
text.setText("fine");
pub.publish(text);

pub.close();
topicSession.close();
topicConn.close();
```

---

## Esempio Subscriber (dominio Pub/Sub)

```java
// passi 1-2: lookup come Publisher

TopicConnection topicConn = connFactory.createTopicConnection();
topicConn.start();   // obbligatorio per Message Consumer

TopicSession topicSession = topicConn.createTopicSession(
    false, Session.AUTO_ACKNOWLEDGE);

TopicSubscriber sub = topicSession.createSubscriber(topic);

TextMessage msg;
do {
    System.out.println("In attesa di messaggi!");
    msg = (TextMessage) sub.receive();
    System.out.println("+ messaggio ricevuto: " + msg.getText());
} while (msg.getText().compareTo("fine") != 0);

sub.close();
topicSession.close();
topicConn.close();
```

---

## Avvio delle Applicazioni JMS

```bash
# 1. Avvio ActiveMQ
cd /<ACTIVEMQ_PATH>/bin
./activemq start         # background
./activemq console       # foreground
./activemq stop
./activemq status

# 2. Avvio applicazione
cd /<PROJECT_PATH>/bin
java -cp /<ACTIVEMQ_PATH>/activemq-all-5.16.6.jar: package_name.ClassName
```

> Nota: il separatore nel classpath e `:` su Unix/Mac e `;` su Windows.  
> `activemq-all-5.16.6.jar` va incluso anche nel classpath del progetto VS Code.

---

## Consumo dei Messaggi

### Ricezione sincrona

| Metodo | Comportamento |
|---|---|
| `receive()` | Si blocca finche non c'e un messaggio disponibile |
| `receive(long timeout)` | Riceve un messaggio che arriva prima del timeout |
| `receiveNoWait()` | Riceve solo se il messaggio e immediatamente disponibile |

### Ricezione asincrona: MessageListener

Il client registra un `MessageListener` sul consumer. Quando arriva un messaggio, il provider JMS invoca automaticamente `onMessage()`.

```java
public class TextMsgListener implements MessageListener {
    public void onMessage(Message m) {
        try {
            System.out.println("+ messaggio ricevuto: " + ((TextMessage) m).getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}

// Registrazione del listener
QueueReceiver receiver = queueSession.createReceiver(queue);
TextMsgListener msglistener = new TextMsgListener();
receiver.setMessageListener(msglistener);
```

> Sia il dominio PTP che Pub/Sub supportano la ricezione asincrona.

---

## Messaggi JMS

### Struttura

Ogni messaggio e composto da tre sezioni:

| Sezione | Contenuto |
|---|---|
| **Header** | Campi per l'invio e l'identificazione del messaggio |
| **Properties** | Coppie `<String, value>` specifiche dell'applicazione o dello standard JMS |
| **Body** | Contenuto effettivo del messaggio |

### Header principali

| Campo | Descrizione |
|---|---|
| `JMSMessageID` | Identificativo univoco del messaggio |
| `JMSCorrelationID` | Collega un messaggio a un altro (es. richiesta-risposta) |
| `JMSReplyTo` | Destination verso cui inviare la risposta |
| `JMSPriority` | Priorita del messaggio (0-9, 9 la piu alta) |

Metodi di accesso: `getJMS*()` e `setJMS*()`.

### Properties

- Coppie `<String, value>` (value puo essere un tipo built-in o una String)
- Metodi: `set*Property()`, `get*Property()`, `getPropertyNames()`
- Supportano una **sintassi SQL-like** per selezionare i messaggi (message selector):

```java
createSubscriber(topic, "prop1 > 6 AND prop2 = 'test'");
```

### Tipologie di body

| Tipo | Contenuto | Metodi principali |
|---|---|---|
| `TextMessage` | `String` | `getText()`, `setText()` |
| `MapMessage` | Coppie name/value | `setString()`, `getString()`, `setDouble()`, ecc. |
| `BytesMessage` | Stream di byte non interpretati | `writeBytes()`, `readBytes()` |
| `StreamMessage` | Stream di valori primitivi | `writeString()`, `writeDouble()`, `readString()`, ecc. |
| `ObjectMessage` | Oggetto `Serializable` | `setObject()`, `getObject()` |

---

## Acknowledgement

Il corretto consumo di un messaggio avviene in 3 fasi:
1. il client riceve il messaggio;
2. il client processa il messaggio;
3. il messaggio e **acknowledged**.

La modalita di acknowledgement e specificata dal **secondo parametro** di `createSession()`:

```java
QueueSession queueSession = queueConn.createQueueSession(false, Session.AUTO_ACKNOWLEDGE);
```

| Modalita | Comportamento |
|---|---|
| `AUTO_ACKNOWLEDGE` | La sessione conferma automaticamente quando il client ritorna dalla `receive()` o il `MessageListener` termina |
| `CLIENT_ACKNOWLEDGE` | Il client chiama esplicitamente `message.acknowledge()`; l'ack e a livello di sessione (ack su un messaggio = ack su tutti i precedenti) |
| `DUPS_OK_ACKNOWLEDGE` | Acknowledgement lasco; possibili messaggi duplicati in caso di malfunzionamento del provider |

---

## Sessioni Transacted

E possibile raggruppare operazioni JMS (send/receive) in **transazioni**:

```java
// Primo parametro = true: sessione transacted
// Secondo parametro: ignorato quando transacted
QueueSession queueSession = queueConn.createQueueSession(true, 0);

try {
    // sequenza di operazioni send/receive
    session.commit();    // tutto ok: rende disponibili i messaggi inviati e fa ack di quelli ricevuti
} catch (Exception e) {
    session.rollback();  // anomalia: scarta i messaggi inviati, rimette in coda quelli ricevuti
}
```

| Operazione | Effetto sui messaggi inviati | Effetto sui messaggi ricevuti |
|---|---|---|
| `commit()` | Diventano disponibili per altre applicazioni | Vengono acknowledged; non consegnati nuovamente |
| `rollback()` | Vengono scartati | Tornano disponibili per l'invio (rimessi in coda in PTP) |

---

## Sessioni JMS e Concorrenza

| Oggetto | Thread-safe? |
|---|---|
| `Destination`, `ConnectionFactory`, `Connection` | Si (la specifica prevede accesso concorrente) |
| `Session`, `MessageProducer`, `MessageConsumer` | No (non pensati per uso multithread) |

Motivazione: le Session supportano le transazioni, per le quali e complesso implementare un supporto multithread; la ricezione asincrona multithread non e supportata.

---

## Persistenza dei Messaggi

| Modalita | Comportamento |
|---|---|
| `PERSISTENT` (default) | Ogni messaggio e registrato su memoria stabile; non perso in caso di malfunzionamento del provider |
| `NON_PERSISTENT` | Il provider non garantisce la memorizzazione in caso di malfunzionamento |

```java
// Impostazione tramite metodo
producer.setDeliveryMethod(DeliveryMode.NON_PERSISTENT);

// Impostazione durante il send (deliveryMode, priority, timeToLive)
producer.send(msg, DeliveryMode.NON_PERSISTENT, 3, 1000);
```

---

## Durable Subscription

Per default, un subscriber riceve messaggi **solo se e attivo** al momento della pubblicazione.

**Durable Subscription** disaccoppia sottoscrizione e subscriber:
- registra la sottoscrizione con un **nome univoco** sul provider;
- i messaggi vengono conservati dal provider JMS anche quando non ci sono subscriber attivi;
- identificata dalla coppia `(clientID, nome_subscription)`.

```java
TopicConnection topicConn = connFactory.createTopicConnection();
topicConn.setClientID("subdurableID");
topicConn.start();

TopicSession topicSession = topicConn.createTopicSession(
    false, Session.AUTO_ACKNOWLEDGE);

TopicSubscriber sub = topicSession.createDurableSubscriber(
    topic, "MakeItLastConn");
```

---

## Estensioni STOMP/ActiveMQ per JMS

### Durable Subscription in STOMP

Usare gli header custom:
- `client-id` nel comando `CONNECT`
- `activemq.subscriptionName` nel comando `SUBSCRIBE`

La coppia `(client-id, activemq.subscriptionName)` identifica univocamente la sottoscrizione.

```python
# sender.py (stomp.py)
import stomp
conn = stomp.Connection([('127.0.0.1', 61613)], auto_content_length=False)
conn.connect(wait=True)
conn.send('/topic/mytesttopic', 'test message')
conn.disconnect()
```

```python
# subscriber durable
conn.connect(wait=True, headers={"client-id": "IDtestsub_durable"})
conn.subscribe(
    destination='/topic/mytesttopic',
    id=1,
    ack='auto',
    persistent=True,
    headers={"activemq.subscriptionName": "IDtestsubscription"}
)
```

### Messaggi JMS Text/Bytes tramite STOMP

| Condizione STOMP | Tipo messaggio JMS creato |
|---|---|
| Header `content-length` incluso | `BytesMessage` |
| Header `content-length` non incluso | `TextMessage` |

La stessa logica vale anche nel passaggio da JMS a STOMP.

---

## Collegamento con slide precedenti

- [[01_PYTHON_10 - Sistemi Middleware]]: concetti MOM, PTP, Pub/Sub
- [[01_PYTHON_12 - Python MOM STOMP]]: STOMP con Python, interoperabilita STOMP-JMS
- [[02_JAVA_01 - Multithreading Java]]: sessioni JMS non thread-safe
- [[02_JAVA_02 - Sincronizzazione Java]]: accesso concorrente a Connection/ConnectionFactory
