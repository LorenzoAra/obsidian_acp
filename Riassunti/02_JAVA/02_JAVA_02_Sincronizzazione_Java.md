# 02_JAVA_02 - Sincronizzazione in Java

#java #advanced-computer-programming #sincronizzazione #synchronized #monitor #wait #notify #semaforo #lock #concorrenza

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- B. Eckel, *Thinking in Java*, capitolo 13
- [Java Tutorial – Concorrenza e Sincronizzazione](http://java.sun.com/docs/books/tutorial/essential/concurrency/sync.html)
- [Java 1.5 Concurrency Utilities Overview](http://java.sun.com/j2se/1.5.0/docs/guide/concurrency/overview.html)

---

## Concetti Chiave

- Sincronizzazione in Java: mutex per ogni oggetto, accesso tramite `synchronized`
- Metodi sincronizzati vs blocchi sincronizzati
- Sincronizzazione di metodi statici: mutex della classe
- Vantaggio per il programmatore: rilascio automatico del mutex
- Avvertenze sui blocchi sincronizzati (dispersione dei vincoli)
- Esempio completo check/write/read con Wrapper
- Sincronizzazione implicita con oggetto ausiliario
- Monitor in Java: classe con metodi `synchronized`, unica condition variable
- `wait()`, `notify()`, `notifyAll()`: primitive di cooperazione da Object
- Monitor JVM: politica "signal and continue"
- Modello grafico: Entry Set, The Owner, Wait Set
- Considerazione fondamentale: ricontrollare la condizione dopo il risveglio (`while` non `if`)
- Pattern cooperazione: `while(<not condition>) { wait(); }`
- Java 1.5: `Semaphore`, interfaccia `Lock`, `ReentrantLock`, `Condition`

---

## Sincronizzazione in Java

Java fornisce il meccanismo di sincronizzazione basato sui **mutex** per l'accesso a sezioni critiche:
- **Ogni oggetto** ha associato un mutex (lock)
- Il mutex non e acceduto direttamente dall'applicazione, ma tramite:
  - **Metodi sincronizzati** (`synchronized` nella firma del metodo)
  - **Blocchi sincronizzati** (`synchronized(oggetto) { ... }`)

Quando un thread esegue un metodo/blocco `synchronized`:
1. Entra in possesso del mutex associato all'istanza
2. Acquisisce l'accesso **esclusivo** alla sezione critica (mutex lock)
3. Gli altri thread che vogliono accedere vengono posti in **attesa**

---

## Metodi Sincronizzati

```java
synchronized void Method() { ... }
```

L'accesso al metodo avviene solo quando il **lock associato all'oggetto** e stato acquisito. Al termine del metodo (normale o per eccezione), il lock e **rilasciato automaticamente**.

> **Vantaggio per il programmatore:** non occorre preoccuparsi di rilasciare il mutex manualmente, viene eseguito automaticamente.

### Esempio: SharedCounter

```java
class SharedCounter {
    private int theData;

    public SharedCounter(int initialValue) {
        theData = initialValue;
    }

    public synchronized int read() {   // acquisisce il lock sull'oggetto
        return theData;
    }                                  // rilascia il lock sull'oggetto
}
```

### Note importanti

> [!warning]
> - I metodi **non sincronizzati** non richiedono il lock e possono essere eseguiti in ogni istante **senza garanzie di mutua esclusione**.
> - I metodi sincronizzati garantiscono la ME **solo se si accede ai dati tramite metodi dichiarati `synchronized`**.

---

## Sincronizzazione di Metodi Statici

I metodi statici possono essere dichiarati `synchronized`. Poiche non sono legati ad alcuna istanza, viene acquisito il **mutex associato alla classe** (non all'istanza).

```java
class StaticSharedVariable {
    public synchronized int Read() { ... }            // mutex dell'ISTANZA
    public synchronized static void Write(int i) { ... } // mutex della CLASSE
}
```

| Scenario | Esecuzione |
|---|---|
| Due metodi statici `synchronized` della stessa classe, due thread | **In sequenza** (stesso mutex di classe) |
| Un metodo statico + un metodo di istanza, entrambi `synchronized` | **In concorrenza** (mutex diversi) |

> Ottenere il lock della classe **non influenza** i lock di qualsiasi istanza della classe.

---

## Blocchi Synchronized

Permettono di definire sezioni critiche anche quando non coincidono con il corpo di un metodo. `synchronized` prende come parametro il **riferimento all'oggetto** di cui acquisire il lock:

```java
public int read() {
    synchronized(this) {   // acquisisce il lock sull'oggetto corrente
        return theData;
    }
}

// Con oggetto ausiliario (sincronizzazione implicita)
private Object obj = new Object();
synchronized(obj) {
    comando1;
    ...
    comandoN;
}
```

### Avvertenze

- Maggiore espressivita nell'implementare vincoli di sincronizzazione
- L'eccessivo uso puo rendere il codice **disordinato**
- I vincoli non sono piu incapsulati in un singolo posto (definizione del metodo)
- Non e possibile capire i vincoli di un oggetto `O` guardando solo `O`: bisogna controllare **tutti gli oggetti** che accedono ad `O` in un blocco `synchronized`

---

## Esempio Completo: check/write/read (Wrapper)

**Scenario:** thread in lettura/scrittura concorrente; `check()` + `read()`/`write()` devono essere atomici.

### Soluzione SBAGLIATA (solo metodi synchronized)

```java
class Wrapper {
    private int buffer;
    synchronized void write() { ... }
    synchronized void read()  { ... }
    synchronized bool check() { ... }
}
```

**Perche e sbagliata:** un thread puo acquisire il monitor rilasciato da `check()` prima che lo stesso thread faccia `read()`/`write()`, violando l'atomicita della coppia.

### Soluzione CORRETTA (blocchi sincronizzati)

```java
class Wrapper {
    private int buffer;
    void write() { ... }   // nessun synchronized qui
    void read()  { ... }
    bool check() { ... }
}

class MyWriter extends Thread {
    private Wrapper wrapper;

    public MyWriter(Wrapper _wrapper) {
        wrapper = _wrapper;
    }

    void run() {
        synchronized(wrapper) {          // lock sull'istanza di Wrapper
            if (wrapper.check())
                wrapper.write();
        }
    }
}
```

### TestApp (main)

```java
class TestApp {
    public static void main() {
        Wrapper buf = new Wrapper();         // unica istanza Wrapper
        boolean is_reader = false;

        for (int i = 0; i < N; i++) {
            Thread th;
            if (is_reader) {
                th = new MyReader(buf);      // passa sempre lo STESSO Wrapper
                is_reader = false;
            } else {
                th = new MyWriter(buf);      // cosi tutti i thread usano lo stesso monitor
                is_reader = true;
            }
            th.start();
        }
    }
}
```

> Passare sempre lo **stesso oggetto `Wrapper`** garantisce che tutti i thread usino il **medesimo monitor** nei loro blocchi `synchronized`.

---

## Monitor in Java

Una classe con metodi `synchronized` e un **monitor in Java**.

- Un monitor Java ha una sola (ed implicita) **variabile condition**
- Il thread attivo nel monitor puo sospendersi invocando `wait()`:
  - Rilascia il monitor
  - Inserisce il thread nel **wait set**
  - Rimane sospeso fino a una `notify()` da un altro thread attivo nel monitor

### Monitor della JVM: "Signal and Continue"

La soluzione adottata in Java e quella **"signal and continue"**: il thread che invoca `notify()` rimane in possesso del monitor e **continua** la propria esecuzione all'interno della monitor region (a differenza della soluzione di Hoare).

---

## Modello di Sincronizzazione

```
+--Entry Set--|------The Owner (sezione critica)------+
|    T2  T3   |              T1                       |
|             |                                  [Dato]|
+--Wait Set---+                                        |
|    T1       |--5 (wait set / entry set boundary)     |
+-------------+---------------------------------------+
```

**Transizioni principali:**

| Transizione | Condizione |
|---|---|
| Esterno → Entry Set (1) | Thread vuole entrare nella sezione critica |
| Entry Set → The Owner (2) | Thread acquisisce il monitor |
| The Owner → Exit (caso 1) | Thread completa l'esecuzione in sezione critica |
| The Owner → Wait Set (caso 2) | Thread esegue `wait()` → rilascia il monitor |
| Wait Set → Entry Set (3) | Un altro thread invoca `notify()` o `notifyAll()` |
| Wait Set rimane fermo | Se il thread attivo termina senza `notify()`, solo l'Entry Set compete |

---

## Wait() e Notify(): Primitive di Cooperazione

Le primitive `wait()` e `notify()` sono ereditate da **`java.lang.Object`** e possono essere invocate **solo all'interno di sezioni critiche**.

> [!warning]
> `wait()` e `notify()` sono primitive di **cooperazione**, non risolvono la mutua esclusione!

### Varianti di `wait()`

| Firma | Comportamento |
|---|---|
| `public final void wait()` | Si sospende indefinitamente finche non riceve `notify`/`notifyAll` o `interrupt()` |
| `public final void wait(long millis)` | Come sopra, ma si risveglia anche dopo `millis` millisecondi |
| `public final void wait(long millis, int nanos)` | Come sopra, ma con risoluzione a nanosecondi |

### `notify()` e `notifyAll()`

| Metodo | Comportamento |
|---|---|
| `public final void notify()` | Risveglia **un** thread nel wait set; il thread risvegliato deve attendere il rilascio del mutex e competere per riacquisirlo |
| `public final void notifyAll()` | Risveglia **tutti** i thread nel wait set; competono per il mutex; se ne esiste uno con priorita piu alta, viene eseguito subito |

---

## Considerazione Fondamentale: Ricontrollare la Condizione

Al risveglio, un thread **non puo assumere** che la sua condizione sia vera: altri thread potrebbero essere stati risvegliati indipendentemente.

> [!important]
> Java non garantisce che un thread risvegliato dopo una `wait()` acquisisca immediatamente il lock.
> **E essenziale che un thread verifichi nuovamente la sua condizione appena risvegliato!**
> Usare sempre `while` (non `if`) per controllare la condizione.

### Pattern Cooperazione

```java
// Thread 1 (attende la condizione)
synchronized(obj) {
    while (<not condition>) {   // WHILE, non IF
        obj.wait();
    }
    do_something();
}

// Thread 3 (abilita la condizione)
synchronized(obj) {
    <condition goes true>;
    obj.notify();
}
```

### Esempio: Gestione Deposito

```java
public synchronized void deposita() {
    while (disponibilita == 0) {       // attende se pieno
        try {
            System.out.println("Deposita: nessuno spazio!");
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    disponibilita = disponibilita - 1;
    System.out.println("Deposita disp= " + disponibilita);
    notifyAll();
}

public synchronized void preleva() {
    while (disponibilita == dimensione) {   // attende se vuoto
        try {
            System.out.println("Preleva: nessun oggetto!");
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    disponibilita = disponibilita + 1;
    System.out.println("Preleva disp= " + disponibilita);
    notifyAll();
}
```

---

## Meccanismi di Sincronizzazione in Java 1.5

I monitor built-in Java sono semplici ma hanno limitazioni:
- **Unica wait-queue** (non si possono avere piu condition variable)
- **Non esiste la possibilita di tentare** semplicemente di acquisire un lock (`tryLock`)

A partire da Java 1.5 sono stati introdotti tre package:

| Package | Contenuto |
|---|---|
| `java.util.concurrent` | Classi per paradigmi comuni di programmazione concorrente |
| `java.util.concurrent.atomic` | Programmazione lock-free e thread-safe su variabili semplici (AtomicInteger, AtomicBoolean, ...) |
| `java.util.concurrent.locks` | Framework per diversi algoritmi di locking (read-write locks, condition variables) |

**Vantaggi:**
- Riuso: minore sforzo di programmazione
- Maggiori prestazioni e affidabilita
- Package standard e testati
- Debugging piu semplice

---

## Semafori (`java.util.concurrent.Semaphore`)

La classe `Semaphore` gestisce un insieme di **permessi**:

| Metodo | Comportamento |
|---|---|
| `acquire()` | Blocca il thread corrente se non c'e almeno un permesso disponibile |
| `release()` | Aggiunge un permesso, potenzialmente sbloccando un thread |
| `boolean tryAcquire()` | Acquisisce un permesso **solo se disponibile immediatamente** (non bloccante) |

**Uso tipico:** limitare il numero di thread che accedono a una risorsa condivisa.

> [!note]
> Il semaforo incapsula la sincronizzazione sull'accesso, ma la **consistenza della risorsa** va comunque gestita a parte.

### Costruttori

```java
Semaphore(int permits)                   // permessi iniziali, nessun ordinamento garantito
Semaphore(int permits, boolean fair)     // fair=true → ordinamento FIFO
```

### Semaforo Binario

Un semaforo inizializzato a `1` funge da **lock per la mutua esclusione**. A differenza dei `Lock` di `java.util.concurrent.locks`, il lock del semaforo binario puo essere rilasciato da **qualsiasi altro thread**.

### Esempio: Pool di Risorse

```java
class Pool {
    private static final int MAX_AVAIL = 100;
    private Semaphore available = new Semaphore(MAX_AVAIL, true); // FIFO

    public Object getItem() throws InterruptedException {
        available.acquire();                    // acquire: puo sollevare InterruptedException
        return getNextAvailableItem();
    }

    public void putItem(Object x) {
        if (markAsUnused(x))
            available.release();                // release: NON solleva eccezioni
    }

    protected Object[] items = ...;
    protected boolean[] used = new boolean[MAX_AVAIL];

    protected synchronized Object getNextAvailableItem() { ... }
    protected synchronized boolean markAsUnused(Object item) { ... }
}
```

---

## Lock (`java.util.concurrent.locks`)

`Lock` e un'**interfaccia** del package `java.util.concurrent` con implementazioni piu flessibili rispetto a `synchronized`:

```java
Lock l = ...;
l.lock();
try {
    // accesso alla risorsa protetta
} finally {
    l.unlock();   // garantire sempre il rilascio
}
```

### Funzionalita Addizionali rispetto a `synchronized`

| Metodo | Descrizione |
|---|---|
| `tryLock()` | Tentativo **non bloccante** di acquisire il lock |
| `tryLock(long, TimeUnit)` | Tentativo con **timeout** |
| `lockInterruptibly()` | Acquisizione interrompibile da `interrupt()` |

### `ReentrantLock`

La classe `ReentrantLock` e un `Lock` con lo stesso comportamento e semantica dei lock associati a metodi e blocchi `synchronized`.

### Condition Variable

I `Lock` supportano piu condition variable tramite l'interfaccia `Condition`:

| Metodo `Condition` | Equivalente Monitor | Descrizione |
|---|---|---|
| `await()` | `wait()` | Sospende il thread e rilascia il lock |
| `signal()` | `notify()` | Risveglia un thread in attesa su questa condition |
| `signalAll()` | `notifyAll()` | Risveglia tutti i thread in attesa su questa condition |

Un'istanza `Condition` e legata a un `Lock` e si ottiene con `lock.newCondition()`.

### Esempio: BoundedBuffer con ReentrantLock

```java
class BoundedBuffer {
    final Lock lock       = new ReentrantLock();
    final Condition notFull  = lock.newCondition();  // condition "buffer non pieno"
    final Condition notEmpty = lock.newCondition();  // condition "buffer non vuoto"
    final Object[] items  = new Object[100];
    int putptr, takeptr, count;

    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();          // attende se pieno
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0;
            ++count;
            notEmpty.signal();            // segnala che c'e almeno un elemento
        } finally {
            lock.unlock();                // rilascio garantito nel finally
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();         // attende se vuoto
            Object x = items[takeptr];
            if (++takeptr == items.length) takeptr = 0;
            --count;
            notFull.signal();             // segnala che c'e almeno un posto
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

> **Vantaggio chiave rispetto al monitor built-in:** due condition variable distinte (`notFull` e `notEmpty`) evitano risvegli spurii di thread non interessati alla condizione che e cambiata.

---

## Confronto: synchronized vs Semaphore vs Lock

| Aspetto | `synchronized` | `Semaphore` | `Lock` / `ReentrantLock` |
|---|---|---|---|
| Semplicita d'uso | Alta | Media | Media |
| Rilascio automatico | Si | No (manuale) | No (manuale, ma con `finally`) |
| Condition variable | 1 implicita | No | N esplicite (`newCondition()`) |
| `tryLock` | No | `tryAcquire()` | `tryLock()` |
| Timeout | No | No | `tryLock(long, TimeUnit)` |
| Interrompibile | No | `acquire()` | `lockInterruptibly()` |
| Rilascio da altro thread | No | Si (semaforo binario) | No (deve rilasciare il thread che ha acquisito) |

---

## Collegamento con Slide Precedenti

- 02_JAVA_01 - Multithreading Java: ciclo di vita, scheduling, stati Blocked synch
- 01_PYTHON_07 - Programmazione Concorrente Richiami: mutex, monitor, semafori (concetti generali)
- 01_PYTHON_08 - Python Concurrency: Lock, Semaphore, Condition in Python
