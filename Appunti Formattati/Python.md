## Introduzione

Python è un linguaggio di programmazione **interpretato**, ovvero un linguaggio in cui il codice non viene prima tradotto interamente in linguaggio macchina, ma le singole istruzioni vengono eseguite riga per riga dall'**interprete Python** (`CPython`).

### Esecuzione del codice: CPython e la PVM

- `CPython` traduce il codice in un **byte code** (file `.pyc`)
- Il byte code è un codice generato appositamente per una **macchina virtuale**, la **PVM** (`Python Virtual Machine`)
- La PVM esegue le istruzioni del byte code una alla volta

## Data Objects

Python, in quanto linguaggio **orientato agli oggetti**, manipola i cosiddetti **data objects**, ovvero oggetti che possiedono un tipo.

### Tipi scalari e non-scalari

| Tipo | Descrizione |
|---|---|
| **Scalare** | Oggetto indivisibile, privo di struttura interna accessibile |
| **Non-scalare** | Oggetto con una struttura interna accessibile |

- Le **stringhe** sono un caso particolare: sono per definizione **non-scalari** (dato che sono letteralmente array di caratteri)
- Le stringhe vengono tuttavia **trattate come scalari**: non è possibile modificare con una normale assegnazione un singolo carattere di una stringa

## Funzioni

Una **funzione** in Python è composta dai seguenti elementi:

- **Nome**
- **Parametri**
- **Docstring** (opzionale)
- **Corpo**
- **Valore di ritorno** (se non specificato, una funzione in Python ritorna sempre `None`)

## Scope delle variabili

Per **scope** di una variabile si intende il **mapping** tra il nome della variabile e l'oggetto a cui punta, mapping che può cambiare in base alla funzione in esecuzione.

## Moduli in Python

In Python, le definizioni di funzioni e variabili non vengono salvate all'uscita dell'interprete. Per risolvere questo problema, si utilizzano i **moduli**, che permettono di eseguire istruzioni come script eseguibili (sono infatti dei normali file `.py`).

- I moduli si importano tramite la keyword **import**.
- Nel momento in cui l'interprete esegue l'`import`, vengono eseguite tutte le **dichiarazioni** e le **definizioni** presenti nel modulo importato.

### Esecuzione condizionata: il blocco `__main__`

Per definire delle funzionalità di un modulo che non devono essere eseguite automaticamente all'importazione, è possibile racchiuderle in un blocco condizionale che verifica il valore della variabile speciale `__name__`. In questo modo, il codice al suo interno viene eseguito **solo** quando il modulo viene lanciato direttamente dall'utente, e non quando viene importato da un altro modulo.

```python
def funzione_principale():
    print("Logica del modulo")

if __name__ == "__main__":
    funzione_principale()
```

### Ricerca dei moduli (Module Search Path)

CPython cerca i moduli importati in luoghi specifici, seguendo un ordine ben definito:

1. **Directory corrente** (quella da cui è stato lanciato lo script).
2. Elenco delle directory elencate nella variabile d'ambiente **PYTHONPATH**.
3. **Percorso predefinito**, dipendente dall'installazione di Python.

### La libreria standard

Esiste inoltre una **libreria standard** di moduli Python, contenente moduli che svolgono operazioni importanti ma che non fanno parte del nucleo del linguaggio.

- Un esempio è il modulo **sys**, che tramite la sua variabile `argv` permette di fornire parametri a un programma Python direttamente dalla riga di comando.
- I parametri vengono specificati dopo il nome del programma, con la seguente sintassi:

```bash
python programma.py arg1 arg2 ...
```

## Package

I moduli in Python sono organizzati in **Package**, ovvero directory che raggruppano moduli correlati.

- Ogni directory di un package contiene un file speciale chiamato `__init__.py`.
- Questo file istruisce l'interprete a trattare la directory come un package.
- Il file `__init__.py` può essere **vuoto**, oppure contenere istruzioni di **inizializzazione** per il package.

# Tuple, Liste e Dizionari

Le **strutture dati** principali di Python sono **Tuple**, **Liste** e **Dizionari**, indicate rispettivamente con la sintassi `()`, `[]` e `{}`.

## Tuple

Le **Tuple** sono una sequenza **ordinata** di elementi **eterogenei** che non possono cambiare.

- Sono **immutabili**: una volta creata, una tupla non può essere modificata.

```python
tupla = (1, 2, 3)
tupla = 99  # TypeError: 'tuple' object does not support item assignment
```

## Liste

Una **Lista** è una sequenza **ordinata** di informazioni, accessibile tramite **indice**.

- Può contenere elementi **eterogenei**, ma non è buona norma farlo.
- Gli elementi possono cambiare: è **mutabile**.
- Su una lista si possono effettuare varie **operazioni**:

| Operazione | Descrizione |
|---|---|
| `add` | Aggiunge un elemento |
| `extend` | Estende la lista con altri elementi |
| `remove` | Rimuove un elemento |
| `sort` | Ordina gli elementi |
| `reverse` | Inverte l'ordine degli elementi |

### Alias e copia di una lista

Dichiarando una lista e poi assegnando il nome di questa lista a un'altra variabile, si crea un **alias**: ogni modifica fatta sull'alias si rispecchia sulla lista originale (e viceversa).

```python
lista_originale = 
alias = lista_originale
alias.append(4)
# lista_originale è ora 
```

Se si vuole creare una **nuova lista** partendo da una già esistente (senza effetto alias), bisogna usare l'operatore di **slicing** `[:]`.

```python
copia = lista_originale[:]
copia.append(5)
# lista_originale rimane inalterata da questa modifica
```

## Dizionari

Il terzo tipo di struttura dati è il **Dizionario**, ovvero una struttura composta da elementi associati a un identificativo (una coppia **chiave/valore**).

- I **valori** possono essere di qualsiasi tipo, possono essere duplicati e possono essere anche altre strutture dati (un dizionario di dizionari è possibile).
- Le **chiavi** devono essere **uniche** e **immutabili**.
- Né le chiavi né i valori sono **ordinati**.

```python
diz = {"a": 1, "b": 2}
diz[] = "x"  # TypeError: unhashable type: 'list'
```

## Parametri

In Python **tutto è un oggetto**, con cui si può interagire tramite i **riferimenti**, e ciò vale anche per il **passaggio di parametri**.

- Passando un parametro ad una funzione, il **riferimento** contenuto nel parametro attuale è copiato in quello formale.
- Se si tratta di un **oggetto mutabile**, ogni modifica del parametro formale si riflette su quello attuale.

### Numero variabile di argomenti posizionali

Per passare un numero variabile di argomenti si può dichiarare, tra i parametri nella firma della funzione, il parametro `*args`.

- Deve essere l'**ultimo parametro** della firma (a meno che non sia seguito da `**kwargs`, che deve comunque venire dopo).

### Keyword arguments

Il chiamante può specificare quale **parametro attuale** rappresenta quale **parametro formale** tramite i **keyword arguments**.

- È inoltre possibile usare il parametro `**kwargs` per passare un numero variabile di **keyword arguments**.

## File e Eccezioni

### Flussi di Dati Standard

Quando viene eseguito un programma **Python** si creano 3 oggetti file che rappresentano i flussi di dati standard che collegano il programma al terminale:

- `sys.stdin`
- `sys.stdout`
- `sys.stderr`

### Gestione delle Risorse con `with`

È possibile utilizzare l'istruzione **with** per acquisire risorse e rilasciarle automaticamente al termine del loro utilizzo. Questa istruzione può essere usata sia per **file** che per **lock**.

### Funzioni Principali sui File

Le funzioni principali sui file sono:

- `open`
- `write`
- `readlines`
- `seek`

### Il Modulo `os`

Per gestire i file si fa uso del modulo **os**, che permette di:

- Creare file del filesystem
- Rimuovere file del filesystem
- Rinominare file del filesystem

### Gestione delle Eccezioni

Quando una funzione produce delle condizioni non attese si genera un'**eccezione**. Le eccezioni possono essere gestite tramite il blocco:

```python
try:
    # codice che può generare un'eccezione
    pass
except:
    # gestione dell'eccezione
    pass
else:
    # eseguito se non si verificano eccezioni
    pass
finally:
    # eseguito sempre, con o senza eccezione
    pass
```

Si può anche sollevare un'eccezione manualmente tramite l'istruzione **raise**.

### Asserzioni

Si possono usare le **asserzioni** per essere sicuri che l'esecuzione sia come attesa. Si utilizza quindi l'istruzione **assert**, che verifica la condizione attesa e, se questa fallisce, solleva un'eccezione di tipo `AssertionError`:

```python
assert condizione_attesa
```

## OOP in Python

Python è **Object-Oriented**, quindi ogni tipo di dato rappresenta una classe. Gli oggetti possono essere:

- **Istanziati**
- **Manipolati**
- **Distrutti** (esplicitamente o lasciando il compito al **garbage collector**)

### Definizione di una classe

Una classe può essere implementata tramite la keyword `class`, seguita da nome e eventuali relazioni con altre classi. I dati e le procedure che appartengono ad una classe sono detti **attributi** e **metodi**.

### Metodi speciali (dunder methods)

Il **costruttore** in Python è un metodo chiamato `__init__`, che prende come primo parametro `self`, che rappresenta l'istanza della classe.

Un altro metodo particolare è `__str__`, utilizzato per permettere l'uso di un oggetto di una classe in una `print`. Ci sono molti altri metodi che si possono overridare per adattare il comportamento di operatori speciali, come:

- `__add__` per la **somma**
- `__eq__` per l'**uguaglianza**

Altri metodi importanti sono i **getter** e **setter**, utilizzati per implementare il concetto di **information hiding** all'interno del programma.

## Ereditarietà

Una **superclasse** è una classe da cui altre classi, le **sottoclassi**, ereditano attributi e metodi. Una sottoclasse può ereditare da più superclassi (**ereditarietà multipla**), ma questa possibilità introduce un problema noto come il **Diamond Problem**: se una classe eredita da due classi che overloadano lo stesso metodo di una superclasse, quale metodo verrà usato se invocato dalla prima classe? Python risolve il problema dando la priorità in base all'ordine di ereditarietà.

### Variabili e metodi statici

Le classi in Python possono anche avere delle **variabili statiche**, ovvero condivise tra tutte le istanze della classe. I **metodi statici** invece sono metodi di una classe che possono essere invocati senza averne istanziato un oggetto.

## Eccezioni

In Python ogni eccezione è un oggetto della classe `BaseException` (o di una classe che eredita da quella), tra cui le quattro principali sottoclassi:

- `SystemExit`
- `KeyboardInterrupt`
- `GeneratorExit`
- `Exception`

## Polimorfismo

Il **polimorfismo** in Python consente di avere sottoclassi con metodi omonimi alle superclassi, lasciando poi all'interprete il compito di scegliere il più adatto a **run-time**. La caratteristica principale che consente ciò è il **duck typing** (un tipo di tipizzazione dinamica), che fa uso di **late binding** e **dynamic dispatch** per identificare di che tipo è un oggetto, in base alla presenza di metodi o di proprietà varie.

## Classi astratte e interfacce

Una classe che contiene almeno un metodo astratto (ovvero un metodo senza implementazione, quindi solo la firma) è detta **classe astratta**, mentre se tutti i metodi sono astratti, allora è un'**interfaccia**.

Le classi astratte (e quindi le interfacce) non possono essere istanziate, ma sono solo dei "modelli" per altre classi che ereditano da esse. In Python l'infrastruttura che definisce il comportamento di una classe astratta è data dal modulo `abc`, che va sempre ereditato da qualsiasi classe astratta. Di questo modulo bisogna anche usare il decoratore `@abc.abstractmethod` per indicare che un metodo è astratto.

### Interfacce informali vs formali

Per quanto riguarda le interfacce, ci sono due approcci principali:

| Tipo di interfaccia | Caratteristiche                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Informale**       | Sfrutta la natura dinamica di Python per definire metodi astratti da overridare, ma senza obbligare le sottoclassi ad effettuare questo overriding |
| **Formale**         | Sfrutta l'eredità da `abc` e il decoratore `@abc.abstractmethod` per obbligare le sottoclassi ad effettuare l'overriding di tutti i metodi         |

## Programmazione Concorrente

In Python abbiamo a disposizione due moduli per gestire la concorrenza: **threading** e **multiprocessing**.

## Il modulo `threading`

Il modulo `threading` fornisce un'interfaccia di alto livello per il **multithreading**, tramite varie classi.

### La classe Thread

La classe `Thread` rappresenta un flusso di controllo separato da quello principale (il **Main Thread**). I due metodi principali per creare un Thread sono:

- Istanziare direttamente un oggetto `Thread` (passando un **callable object** come parametro)
- Creare una classe che estende `Thread` e **overrida** il metodo `run()`

Un thread può essere contrassegnato come **daemon**: ciò significa che il programma principale non dovrà attendere che esso si concluda per terminare, ma verrà interrotto automaticamente alla terminazione del processo padre.

È possibile usare la classe `local` per definire degli attributi di un oggetto thread i cui valori sono esclusivi al singolo thread in esecuzione, nonostante esso condivida lo spazio di memoria con il padre e i fratelli.

### Sincronizzazione tra thread

#### Lock e RLock

Per la sincronizzazione viene utilizzata la classe `Lock`, i cui oggetti possono trovarsi in uno di due stati:

- **locked**
- **unlocked**

Di base è *unlocked*, ma quando un thread lo acquisisce diventa *locked*, tornando *unlocked* al momento del rilascio. Per gestire l'acquisizione e il rilascio si possono usare le primitive `acquire` e `release`, oppure, preferibilmente, lo statement `with`.

Un'alternativa al `Lock` è il `RLock` (**Reentrant Lock**), che può essere acquisito più volte dallo stesso thread. A differenza di un `Lock` classico, ha anche una memoria su chi l'ha acquisito ed è in grado di "accettare" la riacquisizione da parte dello stesso thread, incrementando un contatore interno che indica il numero di acquisizioni. È utile per gestire il problema del **self-deadlock**.

#### Condition

La classe `Condition` implementa una **condition variable**, ovvero un oggetto che rappresenta una condizione che uno o più thread possono controllare e restare in attesa finché essa non si verifica (ovvero finché non vengono notificati da un altro thread). Ad ogni condition variable è associato un `Lock` (che di default è **reentrant**, ma può anche essere classico). Fornisce i seguenti metodi:

- `wait`: mette il thread in attesa della notifica
- `wait_for`: mette il thread in attesa della condizione
- `notify`: notifica *n* thread in attesa su quella condition variable
- `notify_all`: notifica tutti i thread in attesa su quella condition variable

#### Semaphore

La classe `Semaphore` introduce l'oggetto **semaforo**, che gestisce un contatore interno che viene decrementato ad ogni `acquire` e incrementato ad ogni `release`. Quando il contatore arriva a zero, blocca il thread chiamante finché non viene incrementato da un altro thread.

#### Event

La classe `Event` fornisce un meccanismo di comunicazione tra thread: è possibile istanziare un oggetto `Event` che ha un flag interno che può essere **Set** o **Clear** (`True` o `False`). Tramite il metodo `is_set` è possibile controllarne lo stato, oppure porre un thread in attesa del set tramite l'apposita `wait`.

### Il Global Interpreter Lock (GIL)

Python utilizza un meccanismo noto come **Global Interpreter Lock (GIL)** per assicurarsi che solo un thread alla volta possa eseguire bytecode, rendendo di fatto l'esecuzione dei thread non parallela e creando un grosso collo di bottiglia per i task **cpu-bound**.

## Il modulo `multiprocessing`

Il modulo `multiprocessing` supporta lo *spawning* (in Windows) oppure il *forking* (in Linux) di processi multipli, aggirando così il problema del GIL — che agisce solo sui thread — dato che crea più processi, ognuno con il suo interprete (**CPython**) e il suo GIL.

### Metodi di avvio di un processo

| Metodo         | Piattaforma di default | Comportamento                                                                                                                         |
| -------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Spawn**      | Windows e MacOS        | Il padre avvia un nuovo Interpreter Process e il figlio eredita solo le risorse necessarie ed esegue il metodo `run`                  |
| **Fork**       | Linux                  | Il figlio eredita tutte le risorse del padre; considerato *unsafe* perché duplica solo il flusso di controllo principale del processo |
| **Forkserver** | —                      | Crea un Server Process a cui il padre si connette quando necessario, richiedendo la fork di un nuovo processo                         |

Con **Fork**, se il padre aveva dei thread figli con dei lock acquisiti, questi lock restano acquisiti anche nel processo figlio; non avendo però quei thread figli a sua volta, il lock non potrà mai essere rilasciato, causando un **deadlock**.

Con **Forkserver**, si evita di ereditare risorse non necessarie e, dato che il Server Process è *single-thread*, l'uso di `fork` al suo interno è *safe*.

### Il fork `multiprocess`

Esiste un fork del modulo `multiprocessing` chiamato **multiprocess** che sostituisce il serializzatore standard (`pickle`) con `dill`, permettendo di passare ai processi paralleli **callable** più complesse, come ad esempio le funzioni **lambda**.

### La classe Process

Un oggetto della classe `Process` rappresenta un'entità da eseguire in un processo separato, comportandosi allo stesso modo dei `Thread` di `threading` (stessa istanziazione). Anche `multiprocessing` fornisce le classi:

- `Lock`
- `RLock`
- `Condition`
- `Semaphore`
- `Event`

### Comunicazione tra processi

Per scambiare oggetti tra processi ci sono due canali di comunicazione principali:

- **Pipe**: consente la connessione tra più processi
- **Queue**: implementa il problema **produttori-consumatori**

Una `Pipe` è una coppia di **Connection Objects** (gli estremi del tubo), mentre una `Queue` è una *process shared queue*, implementata tramite `Pipe` e `Lock` (o semafori).

Gli oggetti `Connection` della `Pipe` rappresentano gli estremi del tubo; si possono utilizzare i metodi:

```python
connection.send(obj)   # invia un oggetto attraverso la connessione
connection.recv()      # riceve un oggetto dalla connessione
```

Quando un processo immette un oggetto in una `Queue`, viene avviato un thread che trasferisce gli oggetti da un buffer alla `Pipe` usata per implementare la `Queue`.

### Shared Memory

Uno degli strumenti offerti dal modulo `multiprocessing` per la condivisione di dati è la **Shared Memory**, rappresentata dagli oggetti di tipo:

- `Value`
- `Array`

Questi oggetti sono compatibili con il **C**.

## Programmazione su Rete

### Introduzione a Internet e TCP/IP

Internet è la grande rete che si estende in tutto il mondo, basata sul protocollo **TCP/IP**. È una rete di reti, i cui componenti sono:

- **Nodi**
- **Collegamenti**
- **Router**

Ad ogni nodo è assegnato un **indirizzo IP** (IPv4 e IPv6), che serve a identificare mittenti e destinatari. I dati sono trasmessi tramite **pacchetti** (IP Datagrams) attraverso due protocolli principali: **TCP** e **UDP**.

### Confronto tra TCP e UDP

| Caratteristica | TCP | UDP |
|---|---|---|
| **Affidabilità** | Affidabile | Non affidabile (come IP) |
| **Connessione** | Orientato alla connessione | Non orientato alla connessione |
| **Flusso dati** | Flusso di byte full duplex | Nessun flusso garantito |
| **Ordine pacchetti** | Preservato (con ritrasmissione dei persi) | Non preservato |
| **Controllo errori/ordine** | Presente | Assente |

## I Socket

I **Socket** sono il meccanismo principale di comunicazione su rete tra processi, anche su macchine differenti. Sono caratterizzati da due elementi:

- **IP**
- **Port**

### Socket in Python

Per creare e usare Socket in Python si usa il modulo `socket`, i cui metodi principali sono:

- `socket`
- `bind`
- `listen`
- `accept`
- `connect`
- `close`

#### Comunicazione TCP

Per scambiare dati su socket **TCP** si usano `send` e `recv`.

**Lato Client:**
- Istanzia una Socket
- Chiede una connessione tramite `connect`
- Una volta ricevuta la connessione, scambia dati tramite `send` e `recv`

**Lato Server:**
- Istanzia una Socket
- Effettua il `bind` per legarla ad una porta
- Si mette in `listen` aspettando una connessione
- Quando arriva una richiesta, effettua la `accept` (che restituisce una socket rappresentante il Client e l'indirizzo della Socket lato Client), instaurando così il collegamento con il client
- Scambia dati tramite `send` e `recv`
- Una volta finito, effettua la `close` sull'oggetto Socket

```python
# Esempio semplificato Server TCP
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('localhost', 8080))
server_socket.listen(1)

connection, client_address = server_socket.accept()
data = connection.recv(1024)
connection.send(data)
connection.close()
```

```python
# Esempio semplificato Client TCP
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(('localhost', 8080))

client_socket.send(b"Messaggio di prova")
response = client_socket.recv(1024)
client_socket.close()
```

#### Comunicazione UDP

Per la comunicazione **UDP** con Socket si usano `sendto` e `recvfrom`.

Dato che UDP non è connection-oriented, non bisogna instaurare la connessione con `connect`, `listen` e `accept`:

- Il **Client** effettua direttamente la `sendto`, specificando indirizzo e porta del server, e poi la `recvfrom`
- Il **Server** deve comunque effettuare la `bind` sulla porta desiderata

```python
# Esempio semplificato Server UDP
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('localhost', 8080))

data, client_address = server_socket.recvfrom(1024)
server_socket.sendto(data, client_address)
```

```python
# Esempio semplificato Client UDP
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
client_socket.sendto(b"Messaggio di prova", ('localhost', 8080))

response, server_address = client_socket.recvfrom(1024)
```

## Server Multi-thread e Multi-processo

Per migliorare l'efficienza dei Server, questi si possono rendere **multi-thread** o **multi-processo**.

È importante ricordare che ogni flusso di controllo (thread o processo) deve sempre avere come parametro:

- La **connessione** restituita dal metodo `accept`, nel caso di Server **TCP**
- **L'indirizzo, la porta del mittente e il messaggio** ricevuto, nel caso di Server **UDP**

## Il Pattern Proxy

Il pattern **Proxy** introduce nel modello di interazione Client-Server un Proxy, che rappresenta un'implementazione del servizio remoto, ma locale al Client. I meccanismi necessari per la comunicazione (TCP o UDP) sono quindi **incapsulati** all'interno del codice del Proxy.

### Pattern Proxy-Skeleton

Un'implementazione del Proxy è il pattern **Proxy-Skeleton**, che prevede:

- Un **Proxy** lato client
- Uno **Skeleton** lato server

Saranno questi due proxy a comunicare tra loro, gestendo tutta la logica di comunicazione, lasciando la **logica applicativa** esclusivamente a client e server.

## Sistemi Distribuiti

Un **sistema distribuito** è un sistema i cui componenti coordinano le loro azioni tramite lo scambio di messaggi, dato che si trovano su macchine diverse connesse in rete.

- In generale un sistema distribuito è **eterogeneo**, e per questo la complessità di sviluppo di software di qualità è molto alta, dato che occorre adoperare tecniche e linguaggi diversi in base all'hardware/software su cui girano i programmi del sistema.
- Anche i **dati** sono distribuiti: memorizzati su più archivi e con **DBMS** diversi.
- Di norma, è raro trovare sistemi complessi sviluppati da zero: la maggior parte vengono creati a partire da sistemi già funzionanti, facendo uso di **COTS** (Commercial Off-The-Shelf) e riutilizzo di **sistemi legacy**.

A differenza di come può sembrare, l'**integrazione** di un sistema esistente è più complessa di uno sviluppo ex-novo, dato che la difficoltà non è la semplice programmazione, ma anche la capacità di adattare componenti già esistenti che magari non erano ideati per interoperare: da qui nascono le tecnologie **middleware**.

## Middleware

Per **middleware** si intende uno strato software tra sistema operativo e applicazioni, che fornisce astrazioni e servizi per lo sviluppo di applicazioni distribuite.

### Proprietà di trasparenza

Un sistema middleware è trasparente rispetto a diversi aspetti:

- **Sistema operativo**: opera al di sopra di esso.
- **Linguaggio di programmazione**: fa da intermediario tra linguaggi diversi, definendo regole non ambigue di corrispondenza con i vari linguaggi.
- **Locazione**: fornisce una localizzazione di tutte le risorse del sistema distribuito.
- **Migrazione**: consente l'accesso a risorse che magari sono state rilocate durante il loro ciclo di vita.
- **Guasti**: offre al programmatore meccanismi per mascherare i guasti, dato che anche in caso di un singolo fallimento l'intero sistema distribuito può cedere.
- **Replicazione**: caratteristica fondamentale per la tolleranza ai guasti e il miglioramento delle prestazioni.

### Modelli di middleware

| Modello | Descrizione | Esempi |
|---|---|---|
| **RPC** (Remote Procedure Call) | Comunicazione sincrona basata su chiamate a procedure remote | SunRPC, gRPC |
| **MOM** (Message Oriented Middleware) | Comunicazione asincrona basata su code di messaggi | JMS, STOMP |
| **DOM** (Distributed Object Model) | Invocazione di metodi su oggetti remoti | Java RMI |
| **Web Services** | Comunicazione basata su servizi web | Flask |

## Modello RPC (Remote Procedure Call)

Nei sistemi **RPC**, la comunicazione è **sincrona e bloccante**, quindi i processi restano in attesa finché la procedura remota non è completata.

Funziona tramite dei metodi di interfaccia chiamati **stub** (lato client e server) che implementano lo scambio di parametri e la comunicazione su rete:

- **Stub client**: preleva i parametri, li impacchetta e li affida al software di rete, attende la risposta, la spacchetta e infine restituisce i parametri al client.
- **Stub server**: attende che qualcuno invochi la procedura, spacchetta il messaggio ricevuto e lo trasmette alla procedura, impacchetta la risposta della procedura e la invia, rimettendosi poi in attesa.

### Marshalling e unmarshalling

L'impacchettamento e lo spacchettamento sono detti **marshalling** e **unmarshalling**:

- Tramite il **marshalling** si converte il formato dei dati in un formato compreso da entrambe le parti della comunicazione.
- Prevede anche una **serializzazione** dei dati, ad esempio per linearizzare dati strutturati come array o record.

### Semantiche RPC

L'RPC supporta quattro semantiche:

- **exactly once**
- **at least once**
- **at most once**
- **zero or more**

### Sun RPC

Un'implementazione dell'RPC è **Sun RPC**, che prevede un programma in esecuzione sul server che contiene una o più procedure remote; vivendo sullo stesso programma, queste possono condividere variabili.

- Più **versioni** dello stesso programma possono essere in esecuzione, quindi per invocare una procedura il client avrà bisogno di una **tripla** (programma, versione, procedura).
- In base alla semantica, Sun RPC può usare **TCP** (semantica *at most once*) o **UDP** (semantiche *at least once*, *zero or more*).

Di base con socket TCP/UDP bisogna concordare la porta su cui comunicare, ma Sun RPC usa il **binding dinamico** per il numero di porta:

1. Il client deve prima contattare l'**RPC port mapper**, che ha una lista dei programmi attivi con rispettivo numero di porta.
2. Il port mapper restituisce la porta a cui comunicare.
3. Il port mapper è registrato sulla porta predefinita **111**.

## Modello MOM (Message Oriented Middleware)

I middleware **MOM** sono basati su **code di messaggi** tra processi su rete, implementando il concetto di **mailbox**.

- In questo modello, la comunicazione non è client-server, ma **peer-to-peer** di tipo produttore-consumatore **asincrono**.
- Tramite un modello **publish-subscribe**, i produttori pubblicano i messaggi differenziati per tipo e i consumatori si iscrivono alla mailbox per riceverli.

## Modello DOM (Distributed Object Model)

Il **modello a oggetti distribuiti (DOM)** rappresenta la programmazione orientata agli oggetti applicata alla comunicazione client/server, cioè permette di invocare metodi su oggetti remoti come fossero locali, tramite un meccanismo analogo a quello RPC (**procedure stub** e un **Interface Definition Language**).

- Le piattaforme DOM si presentano spesso sotto forma di **ORB** (Object Request Broker), ovvero una sorta di bus per la comunicazione tra oggetti su rete.
- Un esempio di DOM è **Java RMI**, specifico per Java (quindi ambiente distribuito omogeneo), dove i server creano oggetti e li registrano nell'**RMI registry** e i client li ricercano in questo catalogo e ne invocano i metodi.

## gRPC

Il **gRPC** è un middleware universale, multilinguaggio e multipiattaforma con le seguenti caratteristiche:

- Usa **HTTP/2** per il trasporto
- Usa un **protobuf** come *Interface Definition Language* (IDL), per definire il contratto a cui le parti della comunicazione dovranno attenersi
- Supporta lo **streaming bidirezionale**
- Supporta il **multiplexing**
- Supporta l'**autenticazione**
- Supporta le comunicazioni sia **bloccanti** che **non bloccanti**

L'idea di base è quella di trattare le **procedure remote** come fossero dei riferimenti a oggetti HTTP.

### HTTP/2 e Binary Framing Layer

L'**HTTP/2**, rispetto all'1.1, presenta le seguenti differenze:

- Fa uso di un **Binary Framing Layer** per suddividere le richieste/risposte HTTP in messaggi di piccola dimensione
- Questi messaggi vengono trasmessi in dei flussi (**stream**) bidirezionali che consentono di trasportare anche più di un messaggio alla volta
- Implementa così il **multiplexing**, dato che un client non dovrà usare più di una connessione alla volta

### Protocol Buffer (protobuf)

I **protocol buffer (protobuf)** di gRPC sono degli IDL con le seguenti funzionalità:

- Generano automaticamente **stub client** e **classi server astratte** da implementare
- I messaggi vengono **serializzati** e trasmessi tramite HTTP/2
- I tipi di dati definiti nei protobuf sono strutturati come **messaggi**, dove per messaggio si intende un record logico di informazioni contenute in una serie di campi (coppie nome-valore)

### Thread-safety

- Ogni **canale** di comunicazione gRPC è **thread-safe**
- Anche gli **stub lato client** sono thread-safe
- Solo il **server** è sempre soggetto al **GIL** (*Global Interpreter Lock*)

### Tipi di RPC call

gRPC supporta **4 tipi** di RPC call:

| Tipo di RPC | Descrizione |
|---|---|
| **Semplice** | Il client invia una richiesta e riceve una risposta |
| **Streaming lato server** | Il client invia una richiesta e riceve uno stream di risposte |
| **Streaming lato client** | Il client invia uno stream di richieste e riceve una risposta |
| **Streaming bidirezionale** | Il client invia uno stream di richieste e riceve uno stream di risposte |

### Restituzione di uno stream

Affinché una funzione sia in grado di restituire uno stream, una classica `return` non va bene: bisogna usare una funzione **Generator**, che al posto di `return` usa `yield`, che ritorna un **iteratore** contenente *n* elementi su cui iterare.

```python
def stream_generator():
    for elemento in dati:
        yield elemento
```

## Message-Oriented Middleware (MOM)

I **message-oriented middleware** sono utilizzati al posto di quelli basati su **RPC**, e sono ottimi per sistemi che richiedono **scalabilità** e **disponibilità**.

- La comunicazione è **indiretta**, quindi avviene tramite un intermediario che può essere, tra quelli visti:
  - una **coda di messaggi** (queue)
  - un **publish-subscribe** (topic)

### Vantaggi dell'uso di un intermediario

Il vantaggio dell'uso di un intermediario è che non c'è bisogno che tutte le parti della comunicazione siano in esecuzione contemporaneamente: un'applicazione **A** potrebbe inviare un messaggio al **broker**, che sarà poi ritirato da un'applicazione **B** quando questa entrerà in esecuzione.

Questo comporta:
- **comunicazione asincrona**
- **disaccoppiamento**, dato che il sender non deve conoscere nulla del receiver e viceversa

Questo tipo di comunicazione è principalmente usato per la **propagazione di eventi**, tramite il pattern **Observer**.

## Domini di comunicazione

| Dominio | Meccanismo | Consumatori | Persistenza del messaggio |
|---|---|---|---|
| **Point-to-Point (PTP)** | Coda di messaggi | Un unico consumatore | Conservato finché non è consumato (o finché non scade) |
| **Publish-Subscribe** | Topic | Più consumatori possibili | Conservato finché non è rilasciato ai subscriber correnti |

## Protocollo STOMP

Esistono vari protocolli utilizzati dai Broker MOM; quello visto è **STOMP** (Simple Text Oriented Message Protocol), tramite il broker **Apache ActiveMQ**.

STOMP è un protocollo basato sullo scambio di **STOMP Frames**, tramite uno stream di messaggi. In Python si implementa tramite il modulo `stomp`.

### Connessione al provider

Per collegarsi al provider:

1. Si crea un oggetto connessione tramite:

```python
connection = stomp.Connection([('ip', port)])
```

2. Si effettua la `connect()` sull'oggetto appena creato:

```python
connection.connect(wait=True)
```

Specificando `wait=True`, si impedisce al programma di procedere finché la connessione non è stabilita.

### Invio di messaggi

Per inviare messaggi si usa la `send()` sull'oggetto connessione, specificando:
- la **destinazione**:
  - `/queue/...` per il dominio **PTP**
  - `/topic/...` per il dominio **Publish-Subscribe**
- il **messaggio** da immettere nella mailbox

```python
connection.send(destination='/queue/example', body='messaggio')
```

### Ricezione di messaggi

Per ricevere messaggi si chiama sulla connessione:

- **`set_listener()`**, specificando il nome del Listener e una classe che estende `stomp.ConnectionListener` e ridefinisce il metodo `on_message(self, frame)`, cioè il comportamento del Listener ogni volta che riceve un messaggio.

```python
class MyListener(stomp.ConnectionListener):
    def on_message(self, frame):
        print(frame.body)

connection.set_listener('listener_name', MyListener())
```

- **`subscribe()`**, specificando sempre:
  - la **destinazione** (Queue o Topic)
  - un **Id**
  - il **tipo di acknowledge** (ad esempio, `auto`)

```python
connection.subscribe(destination='/queue/example', id=1, ack='auto')
```

### Transazioni

Tramite STOMP si possono trasmettere i messaggi all'interno di una **transazione**: i messaggi non verranno inviati alla mailbox finché la transazione non riceverà una `commit()` (oppure verranno scartati in caso di `abort()`).

## Web Services, REST e Flask

### Web Service e architettura REST

Un **Web Service** è un sistema identificato da un **URI** le cui interfacce pubbliche sono definite tramite **XML**.

Un **WS RESTful** (REST sta per **Representational State Transfer** ed indica un'architettura in cui il sistema è aperto, a basso accoppiamento e distribuito massivamente) si differenzia dai meccanismi **RPC** perché:

- si basa sulle **risorse** (e non sulle procedure): il WS espone dati e funzioni tramite risorse identificate da **URI**, interagibili — nel caso di HTTP — tramite **GET**, **POST**, **PUT** e **DELETE**
- è **stateless**, ovvero non esiste il concetto di sessione: ogni ciclo di richiesta-risposta è a sé stante

Lo scenario più comune di un'architettura REST è sul protocollo **HTTP**, in cui ogni pagina è identificata da un **URL** e prevede i metodi visti prima per implementare le operazioni **CRUD** (Create, Retrieve, Update, Delete).

**XML** e **JSON** sono i formati usati per la rappresentazione testuale dei dati esterna.

A differenza di **RPC**, in cui ogni servizio espone un proprio "vocabolario", in **REST** il vocabolario è fisso: quello di **HTTP**.

Per progettare un sistema REST bisogna:

1. Identificare quali **risorse** esporre
2. Definire, per ognuna, le **URI**
3. Esporre un subset adeguato (definire il comportamento delle operazioni HTTP e quali sono ammesse su di essa)
4. Progettarne le **rappresentazioni** (ad esempio in XML o JSON)

### HTML e DOM

**HTML** è un linguaggio di markup ipertestuale che permette di strutturare visivamente gli elementi di una webapp. È composto da **HTML tags**, caratterizzati da *opening* e *closing*, al cui interno sono presenti testi o altri tag.

- Per i titoli o i contenuti importanti si usano i tag **Header**
- Per creare link si usa il tag **Anchor**
- Per specificare liste (ordinate e non) si usa il tag **List**

L'architettura definita dai tag HTML è detta **Document Object Model (DOM)** (da non confondere con il **Distributed Object Model**): ogni tag rappresenta un **DOM element** e sono organizzabili come **tree**.

### Flask: introduzione

Un **Web Framework** è un insieme di tecnologie per lo sviluppo di Web App. Quello trattato è **Flask**, un **microframework** progettato per la realizzazione rapida e semplice di web app.

Flask nasce come wrapper di:

- **Werkzeug**, che mette a disposizione routing, debugging e WSGI
- **Jinja2**, che mette a disposizione il supporto per i template

### Routing in Flask

I browser inviano richieste al web server che li instrada verso l'applicazione Flask; è quindi necessario specificare quale codice eseguire in base all'URL richiesta. Questo si fa tramite il decoratore `@app.route()`, sotto cui si specifica la funzione `index()`: di conseguenza la web app eseguirà quel codice alla richiesta di quell'URL.

```python
@app.route('/')
def index():
    return "Hello, World!"
```

In Flask si possono definire anche **dynamic routes**, ovvero route che contengono porzioni variabili all'interno dell'URL (indicate con `<variabile>` e passate alla funzione come parametro).

```python
@app.route('/user/<variabile>')
def index(variabile):
    return f"Utente: {variabile}"
```

### Template e rendering con Jinja2

Di norma le **view function** (`index()`) ritornano stringhe generate direttamente al loro interno, ma per una migliore comprensione e gestione del codice conviene fare uso dei **Template**, forniti a Flask da **Jinja2**.

**Jinja2** è un template engine che fornisce:

- **Template**: la parte statica della risposta e i placeholder per la parte dinamica
- **Rendering**: sostituisce i placeholder con i valori dinamici effettivi, che possono essere qualsiasi tipo di variabile, anche tipi complessi come liste, dizionari o oggetti

Di default i template vanno messi nella sottocartella `./templates` e sono file `.html`. Jinja2 offre anche delle **strutture di controllo** (condizioni e cicli).

### curl e richieste HTTP da terminale

È possibile generare richieste HTTP da terminale tramite il comando `curl`:

| Parametro | Funzione |
|---|---|
| `-X` | specifica il metodo HTTP |
| `-I` | ottiene solo gli header della risorsa |
| `-d` | specifica il campo data per le richieste POST |
| `--json` | invia dati in formato JSON per le richieste POST |

### Request e Response Object in Flask

Un **Request Object** è ciò che mette a disposizione tutte le informazioni delle richieste HTTP inviate dai client. Attraverso il **Request Object** è possibile accedere al **body** della richiesta e ai **parametri** passati nella URL.

Di default una route corrisponde a una richiesta **GET**, ma si può usare l'argomento `methods` di `route` per definire altri metodi HTTP. Flask mette comunque a disposizione dei metodi shortcut per queste funzionalità, come `@app.post()`.

```python
@app.route('/dati', methods=['POST'])
def crea_dato():
    ...

@app.post('/dati')  # shortcut equivalente
def crea_dato():
    ...
```

Il valore di ritorno di una view function è convertito automaticamente in un **Response Object**:

- se è una **stringa**, conterrà la stringa come body, `200` come status code e `text/html` come mimetype
- se è un **dizionario** o una **lista**, viene usato automaticamente il metodo `jsonify()` per serializzare la risposta in JSON

Flask permette inoltre di costruire un **Response Object** all'interno della view function tramite la funzione `make_response()`, i cui argomenti vengono considerati come una tupla di valori per la generazione del Response Object.

```python
from flask import make_response

@app.route('/custom')
def custom_response():
    return make_response("Contenuto", 201, {"X-Header": "valore"})
```

### La libreria requests in Python

In Python esiste una libreria HTTP per creare facilmente richieste HTTP, ovvero **requests**. I metodi principali sono:

- `head`
- `get`
- `post`
- `put`
- `delete`

Il modulo `requests` consente di utilizzare le **query string**, cioè di passare parametri all'interno dell'URL come *keyword arguments*.

```python
import requests

response = requests.get('https://api.example.com/dati', params={'chiave': 'valore'})
```

È possibile popolare il **payload** del metodo `post` attraverso l'argomento `data` o `json`.

```python
requests.post('https://api.example.com/dati', json={'campo': 'valore'})
```

Anche nel caso del modulo `requests`, il riferimento ottenuto dall'invio di una richiesta è un **Response Object**.

## NoSQL Databases e MongoDB

### Database: concetti generali

In generale i **Database** sono utilizzati sulla parte back-end delle Web App, permettendo di salvare persistentemente dati in memoria (riducendo così l'onere sulla memoria centrale).

Un **Database** non è altro che una **Collection** di dati organizzati, accessibili tramite un **DBMS**, che fornisce la possibilità di creare, modificare e aggiornare database e dati.

I Database possono essere di due categorie:

- **Relazionali (SQL)**
- **Non-Relazionali (NoSQL)**

### Database SQL

Un Database **SQL** è composto di **schemi**, che forniscono un template su come vanno strutturate le tabelle dei dati.

- Ad ogni tabella è associata una **chiave primaria** che consente l'identificazione univoca di un'entità.
- In quanto ben strutturati, i DB SQL hanno pochi errori (anche grazie alle proprietà **ACID**).
- Questo però è anche uno svantaggio: risultano **inutilizzabili per i big data**.
- È inoltre difficile gestire tabelle presenti su **più server diversi**.

### Database NoSQL

I DB **NoSQL** sono invece più flessibili: al posto delle tabelle fanno uso di **collections** di categorie diverse, in cui troviamo dei **documents** (le "righe della tabella").

Possono essere di più tipi:

- **Key-value**
- **Document stores**
- **Search engines**

MongoDB, in particolare, è un **Document Stores DB**.

### Struttura di MongoDB

In questo tipo di DB sono presenti delle **collection** che contengono **document**.

- Ogni document può essere diverso, con un numero di campi variabile e senza necessità di uno schema predefinito.
- L'unica cosa richiesta è un campo **_id**, che dev'esserci in ogni document e rappresenta la **chiave primaria** del document.
- Ogni document è composto da **field**, ovvero coppie **nome-valore**.

## MongoDB in Python: PyMongo

Per usare MongoDB in Python si fa uso di **PyMongo**.

- Si inizia effettuando la connessione all'istanza di MongoDB tramite il metodo:

```python
MongoClient(ip, port)
```

- Da questo è poi possibile accedere ai **database** trattando la connessione come una collection.
- In seguito è possibile accedere alle singole **collection** trattando il database stesso come collection.
- I dati in Python, in quanto **JSON-style**, vengono trattati come **dizionari**.

### Operazioni CRUD in PyMongo

| Operazione        | Metodi                                                                  | Descrizione                                                                                     |
| ----------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Inserimento**   | `insert_one(dict)`, `insert_many(lista di dict)`                        | Inserisce uno o più document in una collection                                                  |
| **Ricerca**       | `find_one(filtro)`, `find(filtro)`                                      | `find()` ritorna un **cursor** su cui iterare, contenente gli elementi che rispettano il filtro |
| **Rimozione**     | `delete_one(filtro)`, `delete_many(filtro)`                             | Rimuove uno o più document che rispettano il filtro                                             |
| **Aggiornamento** | `update_one(filtro, modificatore)`, `update_many(filtro, modificatore)` | Aggiorna uno o più document secondo il modificatore specificato                                 |

Il **filtro** rappresenta dei parametri, sempre sotto forma di dizionario, come ad esempio:

```python
{"autore": "Eliot"}
```

È possibile anche usare degli **operatori** per le query più complesse.

Il **modificatore**, usato nelle operazioni di aggiornamento, rappresenta la modifica da realizzare. Esistono vari modificatori, come ad esempio `$set` e  `$rename`