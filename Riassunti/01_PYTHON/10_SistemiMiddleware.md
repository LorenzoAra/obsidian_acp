# 10 - Sistemi Middleware

#python #advanced-computer-programming #middleware #sistemi-distribuiti #RPC #MOM #DOM #EAI

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- S. Russo, C. Savy, D. Cotroneo, A. Sergio. *Introduzione a CORBA*, McGraw-Hill, Cap. 1.2

---

## 📌 Concetti Chiave

- Sistemi distribuiti: definizione, implicazioni, eterogeneità
- Enterprise Application Integration (EAI): sviluppo vs integrazione
- Middleware: definizione, ruolo, "glue technologies"
- Proprietà di trasparenza del middleware (6 tipi)
- Modelli e tecnologie middleware: RPC, MOM, DOM, WS, TS, TP, RDA, CM
- RPC: stub, marshalling/unmarshalling, semantiche di esecuzione
- Sun RPC: port mapper, binding dinamico, dispatcher
- MOM (Message-Oriented Middleware): code di messaggi, publish-subscribe
- DOM (Distributed Objects Middleware): ORB, Java RMI

---

## 🌐 Sistemi Distribuiti

> [!info] Definizioni
> *"Un sistema distribuito è un sistema i cui componenti, localizzati in computer connessi in rete, comunicano e coordinano le loro azioni solo attraverso scambio di messaggi."* — G. Coulouris
>
> *"Il mio programma gira su un sistema distribuito quando non funziona per colpa di una macchina di cui non ho mai sentito parlare."*
>  — L. Lamport

**Implicazioni di un sistema distribuito:**
- **Elaborazione concorrente**
- **Assenza di clock globale**
- **Malfunzionamenti indipendenti**

**Domini applicativi:**

| Dominio | Esempi |
|---|---|
| Finance & Commerce | Amazon, eBay, PayPal, online banking |
| Information Society | WWW, Google, Wikipedia, YouTube, social network |
| Entertainment | Online gaming, streaming audio-video |
| Transportation | Google Maps, traffic management |
| Science | Grid computing, big data |
| Utility & Cloud | AWS, Azure (hardware/software come servizio) |

---

## 🔀 Eterogeneità nei Sistemi Distribuiti

I sistemi distribuiti sono **eterogenei per definizione**: hardware, SO, linguaggi di programmazione e DBMS diversi sono la norma, non l'eccezione.

**Fonti di eterogeneità:**
- Piattaforme hardware diverse (mainframe, server, workstation, PC)
- Sistemi operativi diversi e spesso incompatibili
- Linguaggi di programmazione diversi:
  - **Alto livello** (C#, Java): presentation logic e business logic
  - **Tradizionali** (COBOL, C, C++): funzionalità legacy o di basso livello
  - **Scripting** (Python, PHP, JSP, ASP.NET): funzionalità server-side
- DBMS diversi su nodi diversi
- Problemi di sicurezza, guasti, contesa/condivisione risorse → più complessi che nei sistemi centralizzati

---

## 🏢 Enterprise Application Integration (EAI)

> [!important]
> L'integrazione di applicazioni (**EAI**) è spesso **più complessa** dello sviluppo ex novo ed è il **principale scenario** nello sviluppo software su larga scala.

**Componenti tipici da integrare:**
- Nuove applicazioni
- Applicazioni **legacy** (sistemi ereditati)
- Componenti **COTS** (Commercial Off-The-Shelf)
- Applicazioni web, sistemi transazionali, DBMS, sistemi ERP

### Sviluppo vs Integrazione

| Aspetto | Sviluppo di Applicazioni | Integrazione di Applicazioni |
|---|---|---|
| Punto di partenza | Applicazioni nuove | Riuso di applicazioni esistenti (spesso poco documentate) |
| Tecnologie | Libera scelta | Vincoli da sistemi esistenti spesso non interoperanti |
| Competenze | Analisi, progettazione, programmazione | Architetture software, reingegnerizzazione, adattamento componenti |
| Chi la svolge | Giovani sviluppatori | Specialisti con anni di esperienza |

> [!tip]
> Nella prospettiva EAI, la **progettazione delle interfacce** e la capacità di adattare componenti esistenti sono più importanti della codifica pura.

---

## 🧩 Il Middleware

> [!info] Definizione
> Il **middleware** è uno strato software interposto tra il **sistema operativo** e le **applicazioni**, che fornisce astrazioni e servizi per lo sviluppo di applicazioni distribuite.

```
┌────────────────────────────────┐
│         Applicazioni           │
├────────────────────────────────┤
│         Middleware             │  ← "Glue Technology"
├────────────────────────────────┤
│      Sistema Operativo         │
├────────────────────────────────┤
│         Hardware               │
└────────────────────────────────┘
```

Il middleware è anche detto **"glue technology"** (tecnologia collante): integra applicazioni mascherando l'eterogeneità dei sistemi su rete tramite **middleware API**.

### Proprietà di Trasparenza del Middleware

| Tipo di Trasparenza | Descrizione |
|---|---|
| **Sistema Operativo** | I servizi sono indipendenti dall'OS → portabilità tra piattaforme |
| **Linguaggio di programmazione** | Interoperabilità tra componenti scritti in linguaggi diversi (sistema di tipi intermedi) |
| **Locazione** | Accesso logico alle risorse senza conoscere la loro posizione fisica (*location transparency*) |
| **Migrazione** | I componenti possono spostarsi in rete senza che i client lo percepiscano (*migration transparency*) |
| **Guasti** | Meccanismi ad alto livello per mascherare malfunzionamenti parziali (*failure transparency*) |
| **Replicazione** | Più copie di un componente per tolleranza ai guasti o bilanciamento del carico, trasparenti ai client (*replication transparency*) |
| **Implementazioni commerciali** | Interoperabilità tra implementazioni diverse dello stesso standard |

---

## 🗂️ Modelli e Tecnologie Middleware

```
orientati all'accesso ai dati:  RDA  TP
orientati alla comunicazione:   RPC  MOM  TS  DOM  CM  WS
```

| Acronimo | Nome completo                  | Esempi                                      |
| -------- | ------------------------------ | ------------------------------------------- |
| **RDA**  | Remote Data Access             | ODBC, JDBC, Oracle DB Integrators           |
| **TP**   | Transaction Processing         | X/Open DTP                                  |
| **RPC**  | Remote Procedure Call          | Sun RPC, OSF DCE RPC, gRPC                  |
| **MOM**  | Message-Oriented Middleware    | IBM MQSeries, AMQP, JMS, DDS, STOMP         |
| **TS**   | Tuple Space                    | Linda, JavaSpaces, Jini                     |
| **DOM**  | Distributed Objects Middleware | Java RMI, OMG CORBA, MS DCOM, .NET Remoting |
| **CM**   | Component Model                | OMG CCM, EJB                                |
| **WS**   | Web Services                   | JAX-WS, MS WCF, Flask                       |

---

## 📞 Modello RPC (Remote Procedure Call)

> [!info] Definizione
> Il modello RPC è un'**estensione ai sistemi distribuiti** del modello procedurale. Permette a un processo di invocare una procedura su una macchina remota come se fosse locale.

**Caratteristiche:**
- Comunicazione **sincrona e bloccante**: il chiamante resta sospeso fino al completamento
- Meccanismi asincroni (non bloccanti) sono l'eccezione

### Meccanismo degli Stub

```
Programma cliente → Stub cliente → Rete → Stub servente → Procedura remota
                  ← Stub cliente ← Rete ← Stub servente ←
```

**Stub cliente:**
1. Preleva i parametri dal chiamante
2. **Marshalling**: li impacchetta in un messaggio
3. Affida il messaggio al software di rete
4. Attende la risposta
5. **Unmarshalling**: spacchetta i valori di ritorno
6. Restituisce i risultati al chiamante

**Stub servente:**
1. Attende messaggi di invocazione dalla rete
2. **Unmarshalling**: spacchetta i parametri
3. Invoca la procedura reale (up-call)
4. **Marshalling**: impacchetta i parametri di uscita
5. Invia la risposta e si rimette in attesa

### Marshalling dei Parametri

Il **marshalling** è l'impacchettamento dei parametri per la trasmissione su rete:

1. **Conversione di formato** → gestisce differenze di rappresentazione (big-endian vs little-endian, complemento a 2, etc.)
2. **Serializzazione** → trasforma i dati strutturati (array, record) in sequenze di byte

**Approcci di External Data Representation:**

| Approccio | Descrizione |
|---|---|
| CORBA CDR | Common Data Representation |
| Sun XDR | usato in Sun RPC |
| Java Object Serialization | specifico per Java |
| XML | formato testuale estensibile |
| JSON | formato testuale leggero (JavaScript Object Notation) |

### Semantiche delle RPC

In presenza di malfunzionamenti (perdita messaggi, caduta nodo):

| Semantica | Descrizione |
|---|---|
| **Exactly once** | La procedura è eseguita una e una sola volta |
| **At most once** | Se eseguita, lo è stata al massimo una volta |
| **At least once** | La procedura è eseguita almeno una volta |
| **Zero or more** | Non è possibile sapere se e quante volte è stata eseguita |

> [!important]
> Se si usa la semantica **at least once**, le procedure devono essere **idempotenti** (eseguirle più volte ha lo stesso effetto di eseguirle una volta).

---

## ☀️ Sun RPC

### Caratteristiche

- Un **programma remoto** contiene una o più procedure invocabili remotamente
- Le procedure dello stesso programma condividono variabili globali
- Una tripla `(programma, versione, procedura)` identifica univocamente la procedura
- **Mutua esclusione** tra procedure dello stesso programma: al più una alla volta
- Serializzazione tramite **XDR** (eXternal Data Representation)

### Semantiche TCP vs UDP in Sun RPC

| Trasporto | Risposta ricevuta | Risposta non ricevuta |
|---|---|---|
| **UDP** | At least once | Zero or more |
| **TCP** | Exactly once | At most once |

### Port Mapper (binding dinamico)

> [!info] Problema
> Client e server devono accordarsi sulla porta: con il binding **statico** il client deve conoscere la porta in anticipo. Sun RPC usa invece il **binding dinamico**.

**Algoritmo:**
1. Il server ottiene dinamicamente una porta disponibile
2. Registra la coppia `(numero programma, numero porta)` presso il **RPC Port Mapper**
3. Il Port Mapper è sempre in ascolto sulla **porta 111**
4. Il client contatta il Port Mapper con il numero programma → riceve il numero porta
5. Il client effettua la RPC alla porta ottenuta

### Dispatcher

Sul nodo server, il **dispatcher** riceve i messaggi dai client e li instrada verso lo **stub corretto** in base alla procedura invocata.

```
Client → stub cliente → [Rete] → Port Mapper (111) → Dispatcher → stub N → Procedura N
```

---

## 📬 Modello MOM (Message-Oriented Middleware)

> [!info] Definizione
> I sistemi MOM si basano sull'astrazione di una **coda di messaggi** tra processi interoperanti su rete (generalizzazione della mailbox dei sistemi operativi).

**Caratteristiche:**
- Comunicazione **peer-to-peer** (non client-server)
- Modello **produttore-consumatore**
- Comunicazione tipicamente **asincrona**
- Supporto al modello **publish-subscribe**:
	- I **produttori** pubblicano messaggi differenziati per tipo
	- I **consumatori** si dichiarano interessati a tipi specifici di messaggi
- Adatto per applicazioni **event-driven**
- Supporto alla **persistenza dei messaggi**

```
				|----------------|  → Coda 1 →  |----------------|
				| Applicazione A | 		        | Applicazione B |
				|----------------|  ← Coda 2 ←  |----------------|
```

---

## 🧱 Modello DOM (Distributed Objects Middleware)

> [!info] Definizione
> Il modello a oggetti distribuiti è un'estensione in ambiente distribuito dell'OOP:
> **Distributed Objects = OOP + Client-Server**

**Generazioni:**
```
Mainframe/Monolithic → 1ª gen: Client/Server → 2ª gen: Distributed Objects
	                                                    (↑ complessità e                                                                   distribuzione)
```

**Caratteristiche:**
- Invocazione di metodi su oggetti remoti come se fossero locali
- Strumenti simili a RPC: stub, Interface Definition Language (IDL)
- Il meccanismo di invocazione è analogo all'RPC

### Object Request Broker (ORB)

Le piattaforme DOM si presentano spesso come **ORB** (bus software per la comunicazione tra oggetti su rete):

```
Cliente → Stub → ORB → Skeleton → Oggetto remoto
```

Esempi: **OMG CORBA**, **Microsoft COM/DCOM**

### Java RMI

**Java RMI** è un middleware DOM specifico per la piattaforma Java (ambiente omogeneo: solo JVM).

**Componenti:**
- **RMI registry**: catalogo dove i server registrano gli oggetti remoti con un nome
- I client ottengono **riferimenti agli oggetti remoti** tramite il registry (URL)
- I client invocano i metodi come se fossero locali

**Funzionalità aggiuntive:**
- Scambio di oggetti come **parametri o valori di ritorno**
- Trasferimento del **codice** di un oggetto su richiesta
- **Serializzazione** e trasferimento dello stato di un oggetto
- Supporto a sistemi con **codice mobile**

---

## 🔗 Collegamenti

- [[09 - Python Networking]]
- [[08 - Python Concurrency]]
- [[06 - Python OOP]]
- [[Pattern Proxy-Skeleton]]
- [[Serializzazione e Marshalling]]

---

## 📚 Risorse Utili

- S. Russo et al. *Introduzione a CORBA*, McGraw-Hill
- [gRPC – Google Remote Procedure Call](https://grpc.io)
- [Apache Kafka – MOM](https://kafka.apache.org)
- [Java RMI docs](https://docs.oracle.com/javase/8/docs/technotes/guides/rmi/)
