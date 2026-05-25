# 07 - Programmazione Concorrente: Richiami

#python #advanced-computer-programming #concorrenza #thread #processi #semafori #sincronizzazione

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- P. Ancilotti, M. Boari. *Programmazione concorrente e distribuita*, McGraw-Hill (Cap. 2 e Cap. 3)
- Dispensa su problemi di programmazione concorrente (Ancilotti – Boari, *Principi e Tecniche di Programmazione Concorrente*)

---

## 📌 Concetti Chiave

- Programma vs Processo; Process Control Block (PCB)
- Context switch e scheduling della CPU
- Thread: differenze con i processi, vantaggi
- Concorrenza vs Parallelismo; Speed-up e Legge di Amdahl
- Problematiche della programmazione concorrente: safety e liveness
- Sincronizzazione: competizione e cooperazione
- Semaforo: `wait` e `signal`
- Mutua esclusione con semaforo (`mutex`)
- Costrutto Monitor e variabili condition
- Problema Produttore/Consumatore
- Problema Lettori/Scrittori

---

## 🖥️ Programma vs Processo

| | **Programma** | **Processo** |
|---|---|---|
| Definizione | Codifica statica di un algoritmo | Programma **in esecuzione** |
| Natura | Descrizione delle operazioni | Unità di esecuzione del Sistema Operativo |
| Memoria | — | Area codice + Heap + Stack |

### Process Control Block (PCB)

Il SO gestisce ogni processo tramite un **descrittore** (PCB) contenente:
- Identificazione del processo (**PID**, proprietario)
- **Contesto del processore** (registri CPU, stack pointer, Program Counter)
- Informazioni di controllo del processo
- **Risorse possedute** (memoria centrale, file aperti)
- Parametri di scheduling
- **Stato del processo**

---

## 🔄 Context Switch

Il SO alterna l'utilizzo della CPU tra i vari processi per sfruttare i tempi morti o rispettare criteri di scheduling.

```
Procedura Context_Switch:
  begin
    Salvataggio_stato       ← salva il PCB del processo corrente
    Scheduling_CPU          ← sceglie il prossimo processo
    Ripristino_stato        ← carica il PCB del nuovo processo
  end
```

**Cause di un context switch:**
- **Timeout**: il tempo assegnato al processo è scaduto
- **Interruzioni di I/O**
- **System call**: il processo richiede un servizio al SO

---

## 🧵 Thread

> [!info] Definizione
> Un **thread** è un flusso di controllo sequenziale **all'interno** di un processo. Separa il concetto di *esecuzione* da quello di *possesso di risorse*.

### Cosa ha un thread (e cosa non ha)

| Risorsa | Thread | Processo |
|---|---|---|
| Registri CPU | ✅ Propri | ✅ Propri |
| Stack | ✅ Proprio | ✅ Proprio |
| Area heap | ❌ Condivisa con il processo | ✅ Propria |
| Area dati statici | ❌ Condivisa con il processo | ✅ Propria |
| Spazio di indirizzamento | ❌ Condiviso (stesso processo) | ✅ Proprio |

> [!tip] Processo pesante vs Processo leggero
> - **Processo** = "processo pesante": porta dietro un contesto completo
> - **Thread** = "processo leggero": contesto più semplice, contenuto in un processo

### Vantaggi dei Thread

- **Creazione/terminazione** molto più efficiente di un processo
- **Comunicazione** tra thread più semplice ed efficiente (nessun coinvolgimento del kernel)
- **Context switch** con overhead decisamente minore
- Guadagno in **performance**; bloccare un thread non blocca l'intero processo
- Rispecchia la natura intrinseca di molti programmi (task indipendenti, server, calcoli numerici)

### Thread a Livello Utente (ULT) vs Kernel (KLT)

| | **ULT** | **KLT** |
|---|---|---|
| Gestione | Libreria utente | Sistema Operativo |
| Context switch | Rapido (no kernel) | Più lento (coinvolge il kernel) |
| Blocco I/O | Blocca l'intero processo | Solo il thread corrente |
| Parallelismo reale | ❌ No (su mono-CPU) | ✅ Sì (su multi-CPU) |

> [!warning]
> Per ottenere speed-up su sistemi monoprocessore ottimizzando i tempi di attesa, **occorrono KLT** (non ULT).

---

## ⚡ Concorrenza vs Parallelismo

```
Concorrenza:                     Parallelismo:
Time ─────────────────           Time ─────────────────
Thread1: ██░░██░░3██               Thread1: ████████████
Thread2: ░░██░░██░░               Thread2: ████████████
(interleaving su 1 CPU)          (esecuzione su CPU separate)
```

| | **Concorrenza** | **Parallelismo** |
|---|---|---|
| Definizione | Più processi si sovrappongono nel tempo | Più processi eseguono *davvero* contemporaneamente |
| Richiede | 1+ CPU | 2+ CPU/core |
| Tecnica | Multitasking, Multithreading | Architetture multiprocessore/multicomputer |

### Speed-up

\[S = rac{T_S}{T_P}\]

- \(T_S\) = tempo di esecuzione **sequenziale**
- \(T_P\) = tempo di esecuzione **parallela/concorrente**
- Lo speed-up è vantaggioso quando \(S > 1\)

### Legge di Amdahl

Con \(f\) = frazione di codice **sequenziale** e \((1-f)\) = frazione **parallelizzabile**, su \(n\) processori:

\[S = rac{1}{f + rac{1-f}{n}}\]

> [!warning] Limiti del parallelismo
> - Lo speed-up non è mai **lineare** con il numero di CPU
> - C'è sempre una parte di lavoro che **non può** essere svolta in parallelo
> - I programmi paralleli devono prima o poi **sincronizzarsi**
> - Esiste una fase comune di **inizializzazione** prima del parallelismo

**Esempio (compilazione parallela):**
- 3 file: 3s + 2s + 1s di compilazione; 1s di linking
- Con 3 CPU: i 3 file si compilano in parallelo in 3s, poi 1s di linking → totale **4s**
- Speed-up = 7/4 = **1.75** (pur usando il triplo dei processori)

### Limiti del Multithreading

Il multithreading è vantaggioso **quando**:
- Il tempo di esecuzione di ciascun thread è **largamente superiore** al tempo di context switch
- L'esecuzione concorrente **non è troppo vincolata** da punti di sincronizzazione

---

## 🔗 Programmazione Concorrente

> [!info] Definizione
> Insieme delle tecniche, metodologie e strumenti per supportare l'esecuzione di applicazioni come un insieme di attività svolte **simultaneamente**.

**Primitive necessarie:**
1. Per definire attività indipendenti (processi, thread)
2. Per la **comunicazione** e **sincronizzazione** tra attività concorrenti

### Problematiche Fondamentali

> [!important] Due proprietà essenziali

**Safety** — assicurare la consistenza:
- **Mutua esclusione**: le risorse condivise sono aggiornate in modo atomico
- **Condition synchronization**: alcune operazioni devono essere differite se la risorsa non è in uno stato appropriato (es. lettura da buffer vuoto)

**Liveness** — assicurare l'avanzamento:
- **No Deadlock**: alcuni processi possono sempre accedere a una risorsa condivisa
- **No Starvation**: tutti i processi, *eventually*, possono accedere alle risorse

### Tipologie di Interazione tra Processi

| Tipo | Descrizione |
|---|---|
| **Indipendenti** | L'esecuzione di P1 non influenza P2 e viceversa → comportamento riproducibile |
| **Interagenti** | L'esecuzione dipende dalla velocità relativa dei processi → comportamento non riproducibile |

**Forme di interazione:**
- **Competizione**: accesso a risorse comuni non utilizzabili contemporaneamente → mutua esclusione
- **Cooperazione**: esecuzione di un'attività comune mediante scambio di informazioni
- **Interferenza**: accesso non autorizzato o erronea gestione → comportamento non deterministico

---

## 🚦 Semaforo

Un semaforo è un **tipo di dato astratto** con:
- Una variabile intera `s.value`
- Una coda `s.queue` dei processi sospesi

### Operazioni

```
void wait(semaphore s) {
    s.value--;
    if (s.value < 0) {
        s.queue.insert(Process);
        suspend(Process);          // il processo viene bloccato
    }
}

void signal(semaphore s) {
    s.value++;
    if (s.value <= 0) {
        s.queue.remove(Process);
        wake-up(Process);          // un processo viene risvegliato
    }
}
```

**Inizializzazione**: il valore del semaforo deve essere inizializzato a un valore **non negativo**.

### Evoluzione del Semaforo (Esempio Step-by-Step)

| Step | Evento | s.value | Coda sospesi |
|---|---|---|---|
| 1 | Iniziale, A in esecuzione | 1 | — |
| 2 | A esegue `wait` | 0 | — (A non sospeso) |
| 3 | B esegue `wait` | -1 | B sospeso |
| 4 | D esegue `signal` | 0 | B risvegliato e messo in coda ready |

---

## 🔒 Mutua Esclusione

> [!info] Concetti chiave
> - **Risorsa Critica**: risorsa ad uso esclusivo (es. stampante)
> - **Sezione Critica**: porzione di codice che utilizza la risorsa critica
> - **Un solo processo** alla volta può accedere alla sezione critica

### Implementazione con Semaforo (`mutex`)

```c
semaphore mutex;

void P(int i) {
    ...
    wait(mutex);      // ← ingresso sezione critica
    /* sezione critica */
    signal(mutex);    // ← uscita sezione critica
    ...
}

int main() {
    mutex.value = 1;  // inizializzazione
    /* esecuzione concorrente di P(1), ..., P(n) */
}
```

---

## 🖥️ Costrutto Monitor

Un **monitor** è un costrutto sintattico che associa un insieme di operazioni a una risorsa condivisa, **garantendo automaticamente la mutua esclusione**:

- Un solo thread per volta può eseguire una procedura del monitor
- Un thread che tenta di entrare mentre un altro è nel monitor viene **bloccato**
- Quando il thread nel monitor si blocca, un altro thread può accedere

### Variabili Condition

```
var_cond x;
x.wait    → sospende il thread corrente (libera il monitor)
x.signal  → risveglia un thread in attesa sulla condizione x
```

### Strategie dopo `signal`

| Strategia | Comportamento |
|---|---|
| **Signal and Wait** | Il thread segnalato (P) riprende immediatamente; il segnalante (Q) viene sospeso |
| **Signal and Continue** (wait & notify) | Q prosegue dopo aver risvegliato P; P va nella coda del monitor e può essere preceduto da altri |

> [!warning] Problema con `notify`
> Dopo una `notify`, entrambi i thread potrebbero concettualmente proseguire violando le proprietà del monitor. Uno dei due **deve essere sospeso**.

---

## 🔁 Problema Produttore/Consumatore

**Due categorie di processi:**
- **Produttori**: depositano un messaggio su una risorsa condivisa (buffer)
- **Consumatori**: prelevano il messaggio dalla risorsa condivisa

**Vincoli:**
- Il produttore **non può produrre** un messaggio prima che il consumatore abbia prelevato il precedente
- Il consumatore **non può prelevare** alcun messaggio finché un produttore non l'ha depositato

> [!tip] Differenza dalla mutua esclusione
> Non basta la mutua esclusione: occorre imporre un **ordinamento** nelle operazioni. Produttori e consumatori devono **coordinarsi** per segnalare l'avvenuto deposito/prelievo.

---

## 📖 Problema Lettori/Scrittori

**Due categorie di processi:**
- **Lettori**: leggono da una risorsa condivisa
- **Scrittori**: scrivono sulla risorsa condivisa

**Vincoli:**

| | Lettore | Scrittore |
|---|---|---|
| **Lettore** | ✅ Accesso contemporaneo consentito | ❌ Esclusione mutua |
| **Scrittore** | ❌ Esclusione mutua | ❌ Accesso esclusivo |

1. I **lettori** possono accedere contemporaneamente alla risorsa
2. Gli **scrittori** hanno **accesso esclusivo** alla risorsa
3. Lettori e scrittori si **escludono mutuamente**

---

## 🔗 Collegamenti

- [[06 - Python OOP]]
- [[08 - Thread in Python]]
- [[09 - Multiprocessing in Python]]
- [[Semafori e Monitor]]
- [[Legge di Amdahl]]

---

## 📚 Risorse Utili

- P. Ancilotti, M. Boari. *Programmazione concorrente e distribuita*, McGraw-Hill
- [Python threading module](https://docs.python.org/3/library/threading.html)
- [Python multiprocessing module](https://docs.python.org/3/library/multiprocessing.html)
