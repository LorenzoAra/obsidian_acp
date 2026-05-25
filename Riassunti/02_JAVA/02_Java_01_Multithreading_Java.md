# 02_JAVA_01 - Multithreading in Java

#java #advanced-computer-programming #multithreading #thread #scheduling #concorrenza #JVM

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [The Java Tutorials: Concurrency](http://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
- B. Eckel, *Thinking in Java*, capitolo 13

---

## Concetti Chiave

- Multithreading in Java: supporto a livello di linguaggio, indipendenza dalla piattaforma
- Due tecniche per creare un thread: `extends Thread` vs `implements Runnable`
- Perche due soluzioni: Java non ammette derivazione multipla
- Costruttore `Thread(ThreadGroup, Runnable, String)`
- Stato condiviso vs privato tra thread (heap vs stack)
- Java runtime data areas: method area, heap, stacks, PC register
- Context switch gestito dalla JVM (<100 istruzioni)
- Metodi principali: `start()`, `run()`, `sleep()`, `yield()`
- Ciclo di vita: New → Runnable → Running → Blocked → STOP
- Tabella stati: Running, Runnable, Sleeping, Waiting, Blocked I/O, Blocked synch
- Scheduling: fixed priority scheduling, 10 livelli, MIN/NORM/MAX_PRIORITY
- Selfish thread, time slicing, good practice con `yield()`
- Interruzione: `interrupt()`, `InterruptedException`, `Thread.interrupted()`, `isInterrupted()`
- Metodi deprecati: `suspend()`, `resume()`, `stop()`
- `sleep()` e `join()`

---

## Multithreading in Java

Java supporta la programmazione multithread a **livello di linguaggio** (a differenza di C++, che usa librerie esterne come `pthread`/POSIX per UNIX).

Java fornisce:
- **Primitive** per definire attivita indipendenti
- **Primitive** per la comunicazione e sincronizzazione tra attivita concorrenti
- Programmi multithread in maniera **standardizzata e indipendente dalla piattaforma**

---

## Creare un Thread

Java offre due tecniche (entrambe in `java.lang`):

| Tecnica | Quando usarla |
|---|---|
| **`extends Thread`** | La classe utente **non** e gia coinvolta in un legame di derivazione |
| **`implements Runnable`** | La classe utente **e gia** coinvolta in derivazione (Java non ammette ereditarieta multipla) |

### Tecnica 1: `extends Thread`

```java
public class MyThread extends Thread {
    public void run() {
        doWork();  // codice del thread
    }
}

Thread t = new MyThread();
t.start();
```

### Tecnica 2: `implements Runnable`

```java
public class MyRunnable implements Runnable {
    public void run() {
        doWork();  // codice del thread
    }
}

Runnable r = new MyRunnable();
Thread t = new Thread(r);
t.start();
```

> La chiamata a `start()` crea due flussi di esecuzione distinti: l'oggetto chiamante continua la propria esecuzione **senza aspettare** la fine del thread.

### Costruttore della classe Thread

```java
Thread(ThreadGroup group, Runnable target, String name)
```

| Parametro | Descrizione |
|---|---|
| `group` | Il gruppo a cui appartiene il thread |
| `target` | Un oggetto di tipo `Runnable` |
| `name` | Il nome del thread |

---

## Thread e Applicazioni: Stato Condiviso

Thread diversi all'interno della stessa applicazione **condividono** la maggior parte dello stato:

| Risorsa | Condivisa? | Note |
|---|---|---|
| **Method area** | Si | Ambiente delle classi |
| **Heap** | Si | Variabili statiche e di istanza |
| **Stack delle attivazioni** | No | Ogni thread ha il proprio stack |
| **Variabili locali dei metodi** | No | Sullo stack del thread |
| **PC register** | No | Ogni thread ha il proprio program counter |

### Java Runtime Data Areas (visione semplificata)

```
+-----------------------------------+
|  Method area  |  heap  |  stacks  |  native method stacks
|   (condivisi da tutti i thread)   |  (per-thread)
+-----------------------------------+
           |                |
    execution engine   native interface
     (PC, FP, SP)
```

- Method area e heap sono condivisi da **tutti** i thread nella JVM
- Ogni thread ottiene un proprio **PC** e **stack**
- Gli stack sono composti da **frame** (uno per ogni invocazione di metodo)

### Context Switch

- Il cambio di contesto tra thread Java e effettuato dalla **JVM**
- Richiede tipicamente meno di **100 istruzioni**

---

## Metodi Principali della Classe Thread

| Metodo | Descrizione |
|---|---|
| `start()` | Alloca il thread nella JVM e determina l'invocazione di `run()` |
| `run()` | Definisce il **comportamento** del thread; il thread viene deallocato alla sua conclusione |
| `sleep(long millisec)` | Sospende il thread per il periodo specificato (interrompibile) |
| `yield()` | Sospende il thread a favore di un altro thread (stesso livello di priorita) |

> **Nota:** I thread in Java non hanno una funzione di `exit`. Il thread viene deallocato **solo** alla fine del metodo `run()`.

---

## Ciclo di Vita di un Thread

```
New ──start()──> Runnable <──────────── unblocks ────────────── Blocked
                    |                                               |
           chosen by scheduler                          thread blocks
                    |                               (sleep, wait, sync, I/O)
                    v
                Running ──yield()/scheduler swaps──> Runnable
                    |
              stop()/run exit
                    |
                   STOP
```

### Tabella degli Stati

| Stato | Blocked | Interruptible | Descrizione |
|---|---|---|---|
| **Running** | | | Il thread e in esecuzione sul processore |
| **Runnable** (ready-to-run) | | | In attesa del proprio turno per il processore |
| **Sleeping** | SI | SI | Diventa ready-to-run allo scadere del tempo di sleep o a seguito di interruzione |
| **Waiting** | SI | SI | Diventa ready-to-run allo scadere di un timeout, dopo una `notify`, o a seguito di interruzione |
| **Blocked I/O** | SI | | Diventa ready-to-run a seguito di una variazione della condizione di I/O attesa |
| **Blocked synch** | SI | | Diventa ready-to-run quando e acquisito il lock su uno statement `synchronized` |

---

## Scheduling dei Thread

Nei sistemi monoprocessore **un solo thread alla volta** puo essere eseguito.

### Priorita

- 10 livelli: da `MIN_PRIORITY` (1) a `MAX_PRIORITY` (10)
- Valore intermedio: `NORM_PRIORITY` (5)
- Un thread **eredita** la priorita del thread che lo crea
- Modificabile con `setPriority(int)`

### Algoritmo: Fixed Priority Scheduling

- Se c'e piu di un thread Runnable, il sistema sceglie quello a **priorita piu alta**
- Se piu thread hanno la **stessa priorita**: algoritmo **First Come-First Served**
- Un thread continua l'esecuzione finche:
  - Un thread a priorita piu alta diventa Runnable
  - Il thread invoca `yield()` o `run()` termina
  - In OS time-slicing: il periodo di CPU e terminato
  - Passa da Running a Blocked (`sleep()`, `sync`, `wait()`, I/O)

### JVM e Thread

La JVM usa una libreria nativa (POSIX, Win32) o propria ("green" threads):
- La maggior parte delle JVM usa i thread del **sistema operativo sottostante**
- Le priorita sono un **suggerimento** per lo scheduler, non garantiscono la correttezza

> **Good practice:** La specifica Java per lo scheduling e molto "lasca". Per codice portabile, **non fare troppe assunzioni** sullo scheduling sottostante. Scrivere programmi che funzionino bene a prescindere dal livello di priorita (o non usarle affatto).

---

## Thread Selfish (Egoisti) e Time Slicing

Un thread selfish occupa la CPU senza mai cederla finche non termina (o arriva un thread a priorita maggiore):

```java
public class SelfishRunner extends Thread {
    public int tick = 1;
    public void run() {
        while (tick < 4000000) {
            tick++;
        }
    }
}
```

### Sistema NON Time-Slicing vs Time-Slicing

| Sistema NON Time-Slicing | Sistema Time-Slicing |
|---|---|
| Thread #0 esegue fino al completamento, poi Thread #1 | Thread #0 e Thread #1 si alternano ogni quanto di tempo |
| Il primo thread occupa la CPU fino alla fine di `run()` | La frequenza dei cambi dipende dalla larghezza degli slot temporali |

> **Nota:** La specifica Java **non impone** il time slicing. Non si puo fare affidamento su di esso.

### Soluzione: `yield()`

`yield()` cede la CPU a un altro thread eseguibile con la stessa priorita, rendendo l'esecuzione **deterministica** (round-robin):

```
Thread #0, tick 500000
Thread #1, tick 500000
Thread #2, tick 500000
Thread #0, tick 1000000
...
```

> **Good practice:** Nel caso peggiore assumere che il sistema sia NON time-slicing. Se l'elaborazione e fortemente CPU-intensive, usare occasionalmente `yield()` o `sleep()`.

---

## Interruzione di un Thread

Un thread solleva un'interruzione su un altro invocando `interrupt()`:

### Caso 1: Metodi che sollevano `InterruptedException`

```java
try {
    // metodo che puo sollevare InterruptedException (es. sleep, wait)
} catch (InterruptedException ex) {
    // codice di gestione
}
```

### Caso 2: Polling con `Thread.interrupted()`

Se il thread non invoca metodi bloccanti, puo controllare periodicamente:

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // thread interrotto...
        return;
        // oppure: throw new InterruptedException();
    }
}
```

### `interrupted()` vs `isInterrupted()`

| Metodo | Tipo | Effetto sul flag |
|---|---|---|
| `Thread.interrupted()` | `static boolean` | **Resetta** il flag `interrupted status` dopo la lettura |
| `isInterrupted()` | `boolean` | **Non modifica** il flag `interrupted status` |

### Metodi Deprecati

| Metodo | Comportamento | Perche deprecato |
|---|---|---|
| `suspend()` | Sospende il thread | Rischio deadlock: il thread viene sospeso senza rilasciare le risorse |
| `resume()` | Riattiva un thread sospeso | Legato a `suspend()` |
| `stop()` | Ferma e uccide il thread | Rischio deadlock: il thread puo essere interrotto prima di rilasciare risorse |

> Un thread si interrompe e muore **solo** alla conclusione del suo `run()`.

---

## `sleep()` e `join()`

### `sleep()`

Pone il thread in stato dormiente per `n` millisecondi:

```java
try {
    Thread.sleep(4000);
} catch (InterruptedException ex) {
    System.out.println(ex);
}
```

### `join()`

Il thread padre attende la conclusione dei thread figli prima di procedere:

```java
Thread[] threads = new Thread[numOfThreads];

// Crea e avvia i thread figli
for (int i = 0; i < threads.length; i++) {
    threads[i] = new MyThread();
    threads[i].start();
}

// Attende la conclusione di tutti i thread figli
for (int i = 0; i < threads.length; i++) {
    try {
        threads[i].join();  // il padre si blocca finche threads[i] non termina
    } catch (InterruptedException ignore) {}
}
```

---

## Collegamento con Python

| Concetto | Python | Java |
|---|---|---|
| Classe base thread | `threading.Thread` | `Thread` (oppure `implements Runnable`) |
| Avvio thread | `t.start()` | `t.start()` |
| Codice del thread | Override di `run()` | Override di `run()` |
| Sleep | `time.sleep(s)` | `Thread.sleep(ms)` |
| Join | `t.join()` | `t.join()` |
| GIL | Si (limita parallelismo reale) | No GIL (parallelismo reale con thread) |
| Scheduling | Delegato all'OS + GIL | Fixed priority scheduling via JVM |

---

## Risorse Utili

- [Java Tutorials: Concurrency](http://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
- [Thread Javadoc](https://docs.oracle.com/en/java/docs/api/java.base/java/lang/Thread.html)
- B. Eckel, *Thinking in Java*, capitolo 13
