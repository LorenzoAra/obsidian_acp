# 08 - Programmazione Concorrente in Python

#python #advanced-computer-programming #threading #multiprocessing #GIL #concorrenza #sincronizzazione

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Python threading docs](https://docs.python.org/3/library/threading.html)
- [Real Python – GIL](https://realpython.com/python-gil/)
- [Python multiprocessing docs](https://docs.python.org/3/library/multiprocessing.html)

---

## 📌 Concetti Chiave

- Due moduli principali: `threading` e `multiprocessing`
- Classe `Thread`: creazione, ciclo di vita, daemon vs non-daemon
- Thread-local data con `threading.local()`
- Primitive di sincronizzazione: `Lock`, `RLock`, `Condition`, `Semaphore`, `Event`
- Global Interpreter Lock (GIL): limiti e implicazioni
- Modulo `multiprocessing`: start method (spawn, fork, forkserver)
- Classe `Process`; sincronizzazione tra processi
- Comunicazione tra processi: `Pipe` e `Queue`
- Shared Memory: `Value` e `Array` con `ctypes`
- Modulo `multiprocess` (fork di multiprocessing con dill)

---

## 🧵 Il Modulo `threading`

Python mette a disposizione due moduli principali per la programmazione concorrente:

| Modulo | Tipo | Uso principale |
|---|---|---|
| `threading` | Multi-thread | Task I/O-bound, concorrenza leggera |
| `multiprocessing` | Multi-process | Task CPU-bound, parallelismo reale |

Il modulo `threading` è un'interfaccia **ad alto livello** costruita su `_thread` (deprecato). Classi disponibili: `Thread`, `local`, `Lock`, `RLock`, `Condition`, `Semaphore`, `Event`.

---

## 🔧 Classe `Thread`

- Rappresenta un'attività che esegue in un **flusso di controllo separato**
- Esiste un oggetto **main thread** (`_MainThread`) corrispondente al flusso principale del programma
- Tutti i thread in Python **condividono lo spazio di memoria** del processo padre

### Due modi per creare un Thread

**Metodo 1 — Passare un callable al costruttore:**
```python
import threading

def func():
    print("Thread running")

if __name__ == "__main__":
    t1 = threading.Thread(target=func)
    t1.start()
    t1.join()
```

**Metodo 2 — Estendere la classe `Thread` (override di `run()`):**
```python
import threading

class MyThread(threading.Thread):
    def run(self):
        print("Thread running")

if __name__ == "__main__":
    t1 = MyThread()
    t1.start()
    t1.join()
```

### Parametri del Costruttore

| Parametro | Descrizione |
|---|---|
| `target` | Callable object invocato da `run()` |
| `name` | Nome del thread |
| `args` | Tupla di argomenti per `target` |
| `kwargs` | Dizionario di keyword args per `target` |
| `daemon` | `True` se il thread deve essere un demone |
| `group` | Riservato (deve essere `None`) |

### Metodi Principali

| Metodo | Descrizione |
|---|---|
| `start()` | Avvia il thread (eseguirà `run()` quando schedulato) |
| `run()` | Corpo del thread — da overridare se si estende `Thread` |
| `join(timeout=None)` | **Bloccante**: attende la terminazione del thread |
| `is_alive()` | Ritorna `True` se il thread è ancora in esecuzione |

> [!warning]
> `join()` con timeout ritorna **sempre `None`** anche se il timeout è scaduto. Usare `is_alive()` per verificare se il thread è ancora vivo dopo il join.

### Ciclo di Vita di un Oggetto Thread

```python
from threading import Thread

class MyThread(Thread):
    def run(self):
        print("Running MyThread...")
        print("is_alive?:", self.is_alive())   # → True (dentro run)
    def method1(self):
        print("method1 invocato...")

m = MyThread()
m.start()
m.join()
m.method1()                                    # si può chiamare anche dopo join!
print("is_alive?:", m.is_alive())             # → False
```

> [!tip]
> `join()` attende la fine dell'esecuzione ma **non distrugge** l'oggetto thread. I metodi dell'oggetto rimangono accessibili.

---

## 👻 Daemon vs Non-Daemon Thread

| | **Non-Daemon** | **Daemon** |
|---|---|---|
| Terminazione programma | Il programma attende la loro fine | Terminano con il programma |
| Risorse | Rilasciate correttamente | Potrebbero non essere rilasciate |
| Uso tipico | Operazioni critiche (file, rete, backup) | Logging, manutenzione, monitoraggio |

```python
# Non-daemon (default)
T = Thread(target=thread_1)

# Daemon
T = Thread(target=thread_1, daemon=True)
# oppure:
T.daemon = True
```

> [!warning]
> I thread daemon vengono **interrotti bruscamente** alla terminazione del processo. File aperti, transazioni DB, ecc. potrebbero non essere rilasciati correttamente. Per arresti graduali: usare thread non-daemon + meccanismi di segnalazione (es. `Event`).

---

## 🗄️ Thread-Local Data: Classe `local`

Permette di definire **variabili locali ad ogni thread**, evitando effetti collaterali da condivisione della memoria:

```python
import threading

d = threading.local()     # oggetto thread-local condiviso
d.val = 999               # valore nel main thread

def f(d):
    # d.val NON esiste ancora in questo thread (AttributeError se acceduto)
    d.val = random.randint(1, 100)   # valore locale a questo thread

# Output: ogni thread ha il proprio valore di d.val
# (MainThread) value=999
# (Thread-1)   No value yet → value=51
# (Thread-2)   No value yet → value=19
```

---

## 🔐 Primitive di Sincronizzazione

### `Lock` — Mutual Exclusion

Un lock può essere **locked** o **unlocked**. Un thread che tenta di acquisire un lock già locked viene **bloccato**.

```python
from threading import Thread, Lock

counter = 0

def increase(by, lock):
    global counter
    lock.acquire()
    local_counter = counter
    local_counter += by
    counter = local_counter
    print(f'counter={counter}')
    lock.release()

lock = Lock()
t1 = Thread(target=increase, args=(10, lock))
t2 = Thread(target=increase, args=(20, lock))
t1.start(); t2.start()
t1.join(); t2.join()
# Output: counter=10, counter=30, The final counter is 30
```

**Metodi principali:**

| Metodo | Descrizione |
|---|---|
| `acquire(blocking=True, timeout=-1)` | Acquisisce il lock; ritorna `True`/`False` |
| `release()` | Rilascia il lock |
| `locked()` | Ritorna `True` se il lock è in stato locked |

**Uso con `with` (consigliato):**
```python
lock = Lock()
with lock:
    # sezione critica (lock acquisito e rilasciato automaticamente)
    ...
```

### `RLock` — Reentrant Lock

Permette allo **stesso thread** di acquisire il lock più volte (utile in funzioni ricorsive o metodi che chiamano altri metodi che usano lo stesso lock).

- Tiene traccia di **owning thread** e **recursion level**
- `acquire()` dallo stesso thread → non bloccante, incrementa recursion level
- `release()` → decrementa recursion level; solo a 0 il lock diventa unlocked

```python
class SharedResource:
    def __init__(self):
        self.lock  = threading.RLock()   # ← RLock permette rientro
        self.value = 0

    def increment(self):
        with self.lock:
            self.value += 1
            self._log()           # chiama un altro metodo che usa lo stesso lock

    def _log(self):
        with self.lock:           # ← stessa istanza: non si blocca (RLock)
            print(f"Value is now {self.value}")
```

> [!warning]
> Sostituire `RLock` con `Lock` nell'esempio sopra causerebbe un **deadlock**.

### `Condition` — Variabile di Condizione

Permette ad uno o più thread di attendere che una **condizione** si verifichi, notificati da un altro thread. È sempre associata a un lock (default: `RLock`).

```python
from threading import Condition

cv = Condition()

# Consumer
with cv:
    while not an_item_is_available(queue):
        cv.wait()          # rilascia il lock e si sospende
    get_an_available_item(queue)

# Producer
with cv:
    make_an_item_available(queue)
    cv.notify()            # risveglia un thread in attesa
```

**Metodi principali:**

| Metodo | Descrizione |
|---|---|
| `wait(timeout=None)` | Rilascia il lock, sospende il thread; risvegliato da `notify` o timeout |
| `wait_for(predicate, timeout)` | Attende finché il callable `predicate` ritorna `True` |
| `notify(n=1)` | Risveglia `n` thread in attesa (default 1) |
| `notify_all()` | Risveglia tutti i thread in attesa |

> [!important]
> Il ciclo `while` nella verifica della condizione è **necessario**: al risveglio dalla `notify`, la condizione potrebbe non essere più verificata (spurious wakeup o altri thread).

### `Semaphore` — Semaforo

Gestisce un **contatore interno** (inizializzato al valore passato al costruttore):

```python
from threading import Semaphore, Thread
import time, random

obj = Semaphore(3)   # al massimo 3 thread contemporaneamente

def display(name):
    obj.acquire()         # decrementa il contatore (blocca se == 0)
    value = random()
    time.sleep(value)
    print(f'Thread {name} got {value}')
    obj.release()         # incrementa il contatore, risveglia eventuale thread
```

**Metodi principali:**

| Metodo | Descrizione |
|---|---|
| `Semaphore(value=1)` | Costruttore; `value` = valore iniziale del contatore |
| `acquire(blocking=True, timeout=None)` | Decrementa; blocca se contatore == 0 |
| `release(n=1)` | Incrementa di `n`; risveglia `n` thread |

### `Event` — Segnalazione tra Thread

Meccanismo semplice: un thread **segnala un evento** e altri thread **attendono** tale segnale.

```python
import threading, time

e = threading.Event()

def task(event, timeout):
    print("Waiting for event...")
    event_set = event.wait(timeout)    # blocca fino all'evento o al timeout
    if event_set:
        print("Event received!")
    else:
        print("Timeout scaduto!")

thread = threading.Thread(target=task, args=(e, 4))
thread.start()
time.sleep(3)
e.set()          # segnala l'evento → risveglia il thread
thread.join()
```

**Metodi principali:**

| Metodo | Descrizione |
|---|---|
| `is_set()` | Ritorna `True` se il flag è settato |
| `set()` | Setta il flag a `True`; risveglia tutti i thread in attesa |
| `clear()` | Resetta il flag a `False` |
| `wait(timeout=None)` | Blocca fino a `set()` o timeout; ritorna `True`/`False` |

---

## 🔒 Global Interpreter Lock (GIL)

> [!important] Definizione
> Il **GIL** è un meccanismo dell'interprete CPython che garantisce che **un solo thread alla volta** possa eseguire bytecode Python.

**Motivazione originale**: thread-safe memory management, richiesto dalle librerie C usate da Python.

| Vantaggi | Svantaggi |
|---|---|
| Semplifica l'implementazione dell'interprete | Riduce il parallelismo su macchine multi-core |
| Migliora le performance di applicativi single-thread | Impatta principalmente i task **CPU-bound** |
| Integrazione facile di librerie C non thread-safe | Non applicabile a task I/O-bound (GIL rilasciato) |

> [!tip]
> Il GIL viene **rilasciato** automaticamente durante operazioni di I/O bloccanti (es. `time.sleep()`, lettura da file, richieste di rete). Il multithreading Python è quindi utile per task **I/O-bound**, non per CPU-bound.

> [!info] Rimozione del GIL?
> Tentativi passati di rimozione non hanno avuto successo: riduzione delle performance single-thread, aumento della complessità dell'interprete, necessità di modificare tutte le estensioni C.

---

## ⚙️ Il Modulo `multiprocessing`

Supera i limiti del GIL usando **processi multipli** anziché thread:
- Ogni processo ha un **proprio GIL** e un proprio spazio di memoria
- Sfrutta appieno le architetture **multi-core**
- Consigliato per task principalmente **CPU-bound**

### Start Method

| Metodo | Disponibilità | Descrizione |
|---|---|---|
| **`spawn`** | Unix + Windows (default su Windows/macOS) | Avvia un nuovo Python interpreter; eredita solo il necessario; più lento |
| **`fork`** | Solo Unix (default su Unix) | Usa `fork()`: eredita **tutto** dal padre; unsafe (deadlock potenziali) |
| **`forkserver`** | Unix con Unix pipe | Processo server dedicato; fork sicura (single-threaded) |

```python
import multiprocessing
multiprocessing.set_start_method('spawn')
```

### Classe `Process`

API identica alla classe `Thread`:

```python
import multiprocess as mp

# Metodo 1 — callable
def func():
    print("Process running")

p = mp.Process(target=func)
p.start()
p.join()

# Metodo 2 — subclass
class MyProcess(mp.Process):
    def run(self):
        print("Process running")

p = MyProcess()
p.start()
p.join()
```

### Sincronizzazione tra Processi

Il modulo `multiprocessing` fornisce le stesse primitive del modulo `threading`: `Lock`, `RLock`, `Semaphore`, `Condition`, `Event`.

```python
from multiprocess import Process, Lock

def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()

lock = Lock()
for num in range(10):
    Process(target=f, args=(lock, num)).start()
```

---

## 📡 Comunicazione tra Processi

### `Pipe` — Comunicazione a Due Endpoint

```python
from multiprocess import Process, Pipe

parent_conn, child_conn = Pipe()     # duplex=True (default) → bidirezionale

def parentData(parent):
    parent.send(['Hello'])
    parent.close()

def childData(child):
    child.send(['Bye'])
    child.close()

p1 = Process(target=parentData, args=(parent_conn,))
p2 = Process(target=childData,  args=(child_conn,))
p1.start(); p2.start()
print(parent_conn.recv())   # → ['Bye']
print(child_conn.recv())    # → ['Hello']
```

> [!warning]
> I dati nella pipe possono essere **corrotti** se due processi usano lo stesso endpoint contemporaneamente per leggere o scrivere.

| Parametro `duplex` | Comportamento |
|---|---|
| `True` (default) | Pipe bidirezionale |
| `False` | `conn1` solo riceve; `conn2` solo invia |

### `Queue` — Coda FIFO Multi-Processo

Implementata tramite pipe + locks/semaphores. Thread/process-safe.

```python
from multiprocess import Process, Queue

def f(q):
    q.put([42, None, 'hello'])

q = Queue()
p = Process(target=f, args=(q,))
p.start()
print(q.get())    # → [42, None, 'hello']
p.join()
```

**Metodi principali:**

| Metodo | Descrizione |
|---|---|
| `Queue(maxsize=0)` | Costruttore; `maxsize=0` → illimitata |
| `put(item, block=True, timeout=None)` | Inserisce; blocca se piena e `block=True` |
| `put_nowait(item)` | Equivalente a `put(item, block=False)` |
| `get(block=True, timeout=None)` | Preleva; blocca se vuota e `block=True` |
| `get_nowait()` | Equivalente a `get(False)` |
| `empty()` / `full()` | Controlla stato della coda |
| `qsize()` | Dimensione corrente della coda |

> [!info] Varianti
> Esistono anche `SimpleQueue` e `JoinableQueue` nel modulo `multiprocessing`.

---

## 🧠 Shared Memory tra Processi

Permette la **condivisione di dati** tra processi senza comunicazione esplicita. Usa il modulo `ctypes`.

### `Value` — Singolo Valore Condiviso

```python
from multiprocess import Process, Value

num = Value('d', 0.0)   # 'd' = double precision float
```

### `Array` — Array Condiviso

```python
arr = Array('i', range(10))   # 'i' = signed integer
```

**Esempio:**
```python
from multiprocess import Process, Value, Array

def f(n, a):
    n.value = 3.1415927
    for i in range(len(a)):
        a[i] = -a[i]

num = Value('d', 0.0)
arr = Array('i', range(10))
p = Process(target=f, args=(num, arr))
p.start()
p.join()
print(num.value)   # → 3.1415927
print(arr[:])      # → [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
```

**Parametro `lock`** (sia per `Value` che `Array`):
- `True` (default) → crea un `RLock` automaticamente per la sincronizzazione
- `Lock` o `RLock` → usa il lock passato
- `False` → nessuna protezione (accessi non sicuri)

### Tipi ctypes Comuni

| Codice | Tipo C | Tipo Python |
|---|---|---|
| `'b'` | `signed char` | `int` |
| `'i'` | `signed int` | `int` |
| `'f'` | `float` | `float` |
| `'d'` | `double` | `float` |
| `'u'` | `Py_UNICODE` | `str` |

---

## 📦 Il Modulo `multiprocess`

> [!info]
> `multiprocess` è un **fork** di `multiprocessing` che usa **dill** invece di pickle per la serializzazione, permettendo il trasferimento di oggetti più complessi tra processi.

**Differenze chiave:**
- Serializzazione migliorata con `dill` (supporta lambda, closures, etc.)
- Permette il trasferimento di oggetti tra processi tramite `Pipe` o `Queue`
- Permette la condivisione tramite process server o shared memory
- API identica a `multiprocessing`

---

## ⚖️ threading vs multiprocessing

| Aspetto | `threading` | `multiprocessing` |
|---|---|---|
| GIL | ❌ Limitato dal GIL | ✅ Ogni processo ha il proprio GIL |
| Memoria | Condivisa (stesso processo) | Separata (IPC necessaria) |
| Overhead | Basso | Alto (creazione processo) |
| Task ottimale | **I/O-bound** | **CPU-bound** |
| Comunicazione | Variabili globali / Lock | Pipe, Queue, Shared Memory |
| Parallelismo reale | ❌ No (GIL) | ✅ Sì |

---

## 🔗 Collegamenti

- [[07 - Programmazione Concorrente Richiami]]
- [[06 - Python OOP]]
- [[Semafori e Monitor]]
- [[Produttore Consumatore Python]]

---

## 📚 Risorse Utili

- [Python threading docs](https://docs.python.org/3/library/threading.html)
- [Python multiprocessing docs](https://docs.python.org/3/library/multiprocessing.html)
- [Real Python – GIL spiegato](https://realpython.com/python-gil/)
