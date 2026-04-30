# 12 - Message-Oriented Middleware in Python (MOM + STOMP)

#python #advanced-computer-programming #MOM #middleware #STOMP #ActiveMQ #messaging #event-based #publish-subscribe

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Apache ActiveMQ](https://activemq.apache.org/)
- [stomp.py docs](https://jasonrbriggs.github.io/stomp.py/)
- [STOMP Specification 1.2](https://stomp.github.io/stomp-specification-1.2.html)

---

## 📌 Concetti Chiave

- Motivazioni per i MOM: scalabilità e high availability rispetto a RPC
- Comunicazione indiretta: group communication, shared memory, code, publish-subscribe
- MOM come message broker con store-and-forward
- Disaccoppiamento spaziale e temporale
- Pattern Observer → Notification Service
- Sistemi event-based nell'industria (FSE, SESAR, NASPI)
- Modelli PTP (Point-to-Point) e Pub-Sub
- Protocolli MOM: AMQP, MQTT, STOMP
- Apache ActiveMQ: broker open source multiprotocollo
- `stomp.py`: libreria Python per STOMP
- API STOMP: `connect`, `send`, `subscribe`, `Listener`
- Transazioni STOMP: `begin`, `commit`, `abort`
- Durable Subscriber

---

## 💡 Motivazioni per i MOM

I MOM sono usati per **rimpiazzare** le soluzioni basate su RPC quando:
- Le chiamate RPC sono **sincrone** → problemi di scalabilità
- Le richieste RPC arretrate **rallentano l'intero sistema**

**Soluzioni offerte:**
- Meccanismo **asincrono** per incrementare la scalabilità
- **High availability**: se un server fallisce, si migra su un altro tramite i broker
- Il sistema rimane operativo anche dopo il fallimento di uno o più server

---

## 🔀 Comunicazione Indiretta

La **comunicazione indiretta** avviene tra entità di un sistema distribuito tramite un **intermediario**, senza accoppiamento diretto tra sender e receiver.

| Tipo | Descrizione |
|---|---|
| **Group Communication** | Un messaggio è mandato ad un gruppo e recapitato ai suoi membri |
| **Shared Memory** | Distributed shared memory, tuple space |
| **Code di messaggi** | Meccanismo point-to-point; un solo receiver per messaggio |
| **Publish-Subscribe** | Il publisher genera messaggi; il subscriber esprime interesse per tipologia; uno-a-molti |

---

## 📬 Message-Oriented Middleware (MOM)

> [!info] Definizione
> Il MOM è un intermediario (**message broker**) nella comunicazione indiretta basata su messaggi. Garantisce lo scambio tramite tecniche di **store-and-forward** ed espone API di alto livello (es. `send`/`receive message`).

```
Applicazione X → [API] → Message Broker (MOM) → [API] → Applicazione Y
```

**Aspetti chiave:**
- Recapita il messaggio anche se A e B risiedono su macchine differenti
- Conserva il messaggio finché la **rete diventa disponibile**
- Conserva il messaggio finché B **diventa disponibile** (anche se non è in esecuzione)
- A **non si blocca** in attesa che B riceva il messaggio → **comunicazione asincrona**

### Proprietà di Disaccoppiamento

| Tipo | Descrizione |
|---|---|
| **Disaccoppiamento spaziale** | Sender e receiver non si conoscono; possono essere sostituiti, migrati, aggiornati |
| **Disaccoppiamento temporale** | Sender e receiver non devono essere in esecuzione contemporaneamente |

---

## 🔔 Evento, Notifica e Pattern Observer

- **Evento**: condizione rilevata da un'applicazione e comunicata al MOM come messaggio
- **Notifica**: l'atto di informare un insieme di applicazioni dell'occorrenza dell'evento

L'approccio evento-notifica richiama il **pattern Observer**:
1. Un observer dichiara interesse tramite **sottoscrizione**
2. Il subject rileva l'occorrenza dell'evento
3. Il subject **notifica** gli observer invocando un'apposita funzione

**Limiti del puro Observer:**
- Il subject deve mantenere la lista degli observer → scarsa scalabilità
- Gli observer devono conoscere il subject

**Soluzione → Notification Service (NS):**
- La responsabilità di gestire observer/sottoscrizioni e notificarli è delegata a una **terza entità (NS)**
- Il NS può utilizzare code di messaggi:
  1. All'occorrenza di un evento, il sender inserisce il messaggio in una coda
  2. Il messaggio viene recapitato ai receiver sottoscritti alla coda

### Sistemi Event-based nell'Industria

| Sistema | Ambito |
|---|---|
| **FSE** | Fascicolo Sanitario Elettronico in Italia e Europa |
| **SESAR** | Controllo del traffico aereo in Europa |
| **NASPI** | Monitoraggio e controllo della rete elettrica nel Nord America |

---

## 📨 Modelli di Messaging

### Point-to-Point (PTP)

```
Client1 → send → Queue → consume → Client2
                       ← ack ←
```

- Ogni messaggio ha **un solo consumer**
- La coda conserva il messaggio finché non scade o viene consumato
- Il receiver conferma la ricezione con un **acknowledgment (ack)**

### Publish/Subscribe (Pub-Sub)

```
Client1 → publishes → Topic → delivers → Client2 (subscriber)
                            → delivers → Client3 (subscriber)
```

- Ogni messaggio è associato a un **topic**
- Un messaggio può essere processato da **0..N subscriber**
- Il topic conserva il messaggio finché non viene rilasciato ai subscriber correnti

---

## 🔌 Protocolli MOM

| Protocollo | Tipo | Uso principale |
|---|---|---|
| **AMQP** | Binario wire-level | Interoperabilità tra fornitori diversi; rimpiazza middleware proprietari |
| **MQTT** | Pub-Sub leggero (TCP/IP), ISO/IEC PRF 20922 | Basso consumo energetico, banda limitata (IoT, fog/edge computing) |
| **STOMP** | Testo (wire-level, 2-way streaming) | Interoperabilità con qualsiasi broker che supporta STOMP |

---

## 🐦 Apache ActiveMQ

**Apache ActiveMQ** è un popolare broker di messaggi **open source, multiprotocollo, basato su Java**.

- Supporta: AMQP, MQTT, **STOMP**, e altri
- Supporta client in: JavaScript, C, C++, **Python**, .NET, Java, …
- Interfaccia di amministrazione web: `http://localhost:8161/` (credenziali: `admin/admin`)
- Porta di default STOMP: **61613**

**Comandi da terminale:**
```bash
activemq start   # avvia il broker
activemq stop    # ferma il broker
```

> [!important]
> È necessario avviare ActiveMQ **prima** dell'esecuzione di un applicativo STOMP.

---

## 🐍 STOMP in Python: `stomp.py`

**STOMP** è un protocollo FRAME-based che usa un 2-way streaming network protocol (es. TCP).  
I frame STOMP hanno la struttura:

```
COMMAND
Header1:value1
Header2:value2

Body^@
```

**Installazione:**
```bash
pip install stomp.py
```

### Connessione al Provider

```python
import stomp

conn = stomp.Connection([('127.0.0.1', 61613)])
conn.connect(wait=True)
# ...
conn.disconnect()
```

**Parametri di `connect()`:**

| Parametro | Tipo | Descrizione |
|---|---|---|
| `username` | str | Username per il login |
| `passcode` | str | Password dell'utente |
| `wait` | bool | Attende il completamento della connessione prima di ritornare |
| `headers` | dict | Header aggiuntivi da inviare con la connessione |
| `with_connect_command` | bool | Se `True`, usa `CONNECT` invece di `STOMP` |

> [!tip]
> `Connection()` accetta una **lista di tuple** `(IP, porta)`: se una connessione fallisce, il client tenta le coppie successive finché una ha successo.

### Invio dei Messaggi

```python
conn.send('/queue/test', 'test message')
# oppure per un topic:
conn.send('/topic/test', 'test message')
```

**Parametri di `send()`:**

| Parametro      | Tipo | Descrizione                                                |
| -------------- | ---- | ---------------------------------------------------------- |
| `destination`  | str  | Destinazione: coda (`/queue/nome`) o topic (`/topic/nome`) |
| `body`         | str  | Contenuto del messaggio                                    |
| `content_type` | str  | MIME type del messaggio                                    |
| `headers`      | dict | Header aggiuntivi                                          |

### Ricezione dei Messaggi (Listener)

La ricezione è **asincrona** tramite un **Listener** e una sottoscrizione a una destination.

```python
class MyListener(stomp.ConnectionListener):
    def on_message(self, frame):
        print('received a message "%s"' % frame.body)
        # frame.body    → corpo del messaggio
        # frame.headers → headers del messaggio
        # frame.cmd     → comando STOMP

conn.set_listener('name', MyListener())
conn.subscribe(destination='/queue/test', id=1, ack='auto')
```

**Parametri di `subscribe()`:**

| Parametro | Tipo | Descrizione |
|---|---|---|
| `destination` | str | Topic o queue a cui sottoscriversi |
| `id` | str | Identificativo univoco della sottoscrizione |
| `ack` | str | Modalità di acknowledgment: `auto`, `client`, `client-individual` |
| `headers` | dict | Header aggiuntivi |

---

## 📦 Esempi Completi

### Point-to-Point con ActiveMQ

**Receiver (`receiver.py`):**
```python
import time, stomp

class MyListener(stomp.ConnectionListener):
    def __init__(self, conn):
        self.conn = conn
    def on_message(self, frame):
        print('received a message "%s"' % frame.body)

if __name__ == "__main__":
    conn = stomp.Connection([('127.0.0.1', 61613)])
    conn.set_listener('', MyListener(conn))
    conn.connect(wait=True)
    conn.subscribe(destination='/queue/test', id=1, ack='auto')
    time.sleep(60)
    conn.disconnect()
```

**Sender (`sender.py`):**
```python
import stomp

conn = stomp.Connection([('127.0.0.1', 61613)])
conn.connect(wait=True)
conn.send('/queue/test', 'test message')
conn.disconnect()
```

### Publish/Subscribe con ActiveMQ

La differenza rispetto al PTP è solo nella **destination**: `/topic/test` invece di `/queue/test`.

**Subscriber:**
```python
conn.subscribe(destination='/topic/test', id=1, ack='auto')
```

**Publisher:**
```python
conn.send('/topic/test', 'test message')
```

---

## 💳 Transazioni STOMP

STOMP supporta la trasmissione di messaggi all'interno di una **transazione**:

- I messaggi sono mantenuti dal server fino al `commit` o `abort`
- `begin()` → ritorna un `transaction_id` da usare nelle successive `send()`

```python
# Commit — i messaggi vengono effettivamente inoltrati
txid = conn.begin()
conn.send('/queue/test', 'test1', transaction=txid)
conn.send('/queue/test', 'test2', transaction=txid)
conn.send('/queue/test', 'test3', transaction=txid)
conn.commit(txid)

# Abort — i messaggi vengono scartati
txid = conn.begin()
conn.send('/queue/test', 'test4', transaction=txid)
conn.send('/queue/test', 'test5', transaction=txid)
conn.abort(txid)
```

---

## 📌 Durable Subscriber

Di default in ActiveMQ, un subscriber riceve i messaggi **solo se pubblicati quando è attivo**.  
Per un **durable subscriber** (riceve anche i messaggi pubblicati quando era offline):

```python
conn.connect(wait=True, headers={"client-id": "IDtestsub_durable"})
# ...
conn.subscribe(
    destination='/topic/mytesttopic',
    id=1,
    ack='auto',
    persistent=True,
    headers={"activemq.subscriptionName": "DURABLESUB_NAME"}
)
```

> [!important]
> La coppia `(client-id, activemq.subscriptionName)` identifica **univocamente** la sottoscrizione duratura. Se `client-id` non è impostato esplicitamente, viene usato l'hostname.

---

## 🔗 Collegamenti

- [[10 - Sistemi Middleware]]
- [[11 - Python RPC gRPC]]
- [[09 - Python Networking]]
- [[Pattern Observer]]
- [[Architettura Event-Driven]]

---

## 📚 Risorse Utili

- [Apache ActiveMQ docs](https://activemq.apache.org/)
- [stomp.py API reference](http://jasonrbriggs.github.io/stomp.py/api.html)
- [STOMP Specification 1.2](https://stomp.github.io/stomp-specification-1.2.html)
