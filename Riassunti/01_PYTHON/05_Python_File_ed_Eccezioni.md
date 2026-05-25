# 05 - Gestione dei File e delle Eccezioni

#python #advanced-computer-programming #file #eccezioni #debugging #asserzioni

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- Tony Gaddis. *Introduzione a Python*. 5° ed. – Pearson, 2021
- Paul J. Deitel, Harvey M. Deitel. *Introduzione a Python. Per l'informatica e la data science.* Pearson, 2021
- [Think Like a Computer Scientist](https://runestone.academy/runestone/books/published/thinkcspy/index.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)

---

## 📌 Concetti Chiave

- File standard Python: `sys.stdin`, `sys.stdout`, `sys.stderr`
- Apertura file con `open()` e modalità (`mode`)
- Istruzione `with` come context manager
- Lettura: iterazione riga per riga, `readlines()`, `seek()`
- Modulo `os` per operazioni su file (rimozione, rinomina)
- Debugging: strumenti e metodo della bisezione
- Tipi di eccezione più comuni
- Gestione eccezioni: `try / except / else / finally`
- Sollevare eccezioni: `raise`
- Asserzioni: `assert` e programmazione difensiva

---

## 🗂️ Gestione dei File

### File Standard Python

Quando si esegue un programma Python, vengono creati automaticamente 3 oggetti file:

| Oggetto | Descrizione |
|---|---|
| `sys.stdin` | Standard input |
| `sys.stdout` | Standard output |
| `sys.stderr` | Standard error |

### Apertura e Scrittura: `open()` e `write()`

```python
with open('accounts.txt', mode='w') as accounts:
    accounts.write('100 Jones 24.98\n')
    accounts.write('200 Doe 345.67\n')
    accounts.write('300 White 0.00\n')
```

**Modalità di apertura (`mode`):**

| Mode | Descrizione | Note |
|---|---|---|
| `'r'` | Lettura (default) | Errore se il file non esiste |
| `'w'` | Scrittura | Crea il file se non esiste; **cancella** il contenuto esistente |
| `'a'` | Append | Aggiunge in fondo, crea se non esiste |
| `'r+'` | Lettura + scrittura | Il file deve esistere |

> [!warning]
> Aprire un file in modalità `'w'` **cancella** tutti i dati esistenti nel file.

### L'istruzione `with` (Context Manager)

> [!info] Perché usare `with`?
> Le applicazioni acquisiscono risorse (file, connessioni di rete, DB) e devono rilasciarle quando non servono più. L'istruzione `with`:
> 1. **Acquisisce** la risorsa e la assegna a una variabile
> 2. **Consente** l'uso della risorsa nel blocco
> 3. **Chiama automaticamente** `close()` alla fine del blocco, anche in caso di errore

```python
with open('accounts.txt', mode='w') as accounts:
    accounts.write('...')
# il file viene chiuso automaticamente qui
```

### Lettura da File

#### Iterazione riga per riga (consigliata)

```python
with open('accounts.txt', mode='r') as accounts:
    print(f'{"Account":<10}{"Name":<10}{"Balance":>10}')
    for record in accounts:                     # una riga alla volta
        account, name, balance = record.split()
        print(f'{account:<10}{name:<10}{balance:>10}')
```

#### `readlines()` — legge tutto il file in una lista

```python
with open('accounts.txt', mode='r') as f:
    lines = f.readlines()   # ritorna una lista di stringhe (una per riga)
```

> [!warning]
> `readlines()` carica **l'intero file in memoria**. Per file di grandi dimensioni può essere lento e bloccare l'utilizzo della lista fino al completamento.

#### `seek()` — riposizionare il puntatore

```python
file_object.seek(0)    # riporta il puntatore all'inizio del file
```

### Modulo `os` per Operazioni su File

```python
import os

os.remove('accounts.txt')                    # cancella un file
os.rename('temp_file.txt', 'accounts.txt')   # rinomina un file
```

---

## 🐛 Debugging

> [!tip] Obiettivo
> Avere programmi **bug-free**. La curva di apprendimento del debugging è lenta: bisogna essere sistematici.

### Strumenti

- IDE built-in debugger
- Python Tutor (`http://www.pythontutor.com/`)
- Istruzione `print` strategica

### Debugging con `print` — Metodo della Bisezione

Usare `print` per ispezionare lo stato del programma:
- All'**ingresso** di una funzione
- Sui **parametri** ricevuti
- Sui **risultati** restituiti

**Strategia di bisezione:**
1. Inserire una `print` nel **punto mediano** del codice
2. Decidere in quale metà si trova il bug in base ai valori ottenuti
3. Ripetere nella metà sospetta

### Passi per il Debugging (Metodo Scientifico)

1. **Studiare** i dati disponibili (output inatteso)
2. **Non chiederti** "cosa è sbagliato?" → chiediti **"come ho ottenuto questo output inatteso?"**
3. **Formulare ipotesi** verificabili
4. **Esperimenti ripetibili** con l'input più semplice possibile

---

## ⚠️ Messaggi di Errore Comuni

| Errore | Causa tipica |
|---|---|
| `SyntaxError` | Parentesi/apici non chiusi, sintassi errata |
| `NameError` | Variabile non definita |
| `TypeError` | Tipo non corretto per l'operazione (es. `'3'/4`) |
| `ValueError` | Tipo ok, ma valore non valido (es. `int("abc")`) |
| `IndexError` | Accesso fuori dai limiti di una lista |
| `AttributeError` | Riferimento ad attributo non esistente |
| `IOError` | Errore di I/O (es. file non trovato) |
| `ZeroDivisionError` | Divisione per zero |

---

## 🚨 Eccezioni

Quando una funzione esegue codice che produce condizioni non attese, viene sollevata un'**eccezione**.

### Strategie per gestire gli errori

| Strategia | Pro | Contro |
|---|---|---|
| **Fallire silenziosamente** (usare valori default) | Semplice | L'utente non viene avvisato |
| **Ritornare un valore di errore** | Esplicito | Complica il codice, quale valore scegliere? |
| **Sollevare un'eccezione** (`raise`) | ✅ Approccio corretto | Richiede gestione esplicita |

### Struttura `try / except`

```python
try:
    a = int(input("Dimmi un numero: "))
    b = int(input("Dimmi un altro numero: "))
    print(a / b)
except:
    print("Errore nell'input utente.")
```

- Le eccezioni sono **sollevate** da qualsiasi istruzione nel blocco `try`
- Vengono **gestite** nel blocco `except`
- L'esecuzione **continua** nel blocco `except`

### Gestire Eccezioni Specifiche

> [!important]
> I blocchi `except` vanno ordinati **dal più specifico al più generico** (seguendo la gerarchia delle eccezioni).

```python
try:
    a = int(input("Dimmi un numero: "))
    b = int(input("Dimmi un altro numero: "))
    print("a/b =", a/b)
    print("a+b =", a+b)
except ValueError:
    print("Impossibile convertire in numero.")
except ZeroDivisionError:
    print("Non si può dividere per zero.")
except:
    print("Qualcosa è andato molto storto.")
```

### `else` e `finally`

```python
try:
    <istruzioni rischiose>
except SomeError:
    <gestione errore>
else:
    # eseguito SOLO se il blocco try NON ha sollevato eccezioni
    <istruzioni se tutto ok>
finally:
    # eseguito SEMPRE (anche se ci sono errori, break, continue, return)
    # ideale per operazioni di clean-up (es. chiudere un file)
    <clean-up>
```

| Blocco | Quando viene eseguito |
|---|---|
| `except` | Se il `try` solleva un'eccezione |
| `else` | Se il `try` completa **senza** eccezioni |
| `finally` | **Sempre**, a prescindere da tutto |

---

## 🔺 L'Istruzione `raise`

Permette di **sollevare esplicitamente** un'eccezione quando non è possibile fornire un risultato coerente:

```python
raise ValueError("something is wrong")
raise Exception("descriptive string")
```

> [!tip] Regola d'uso
> Non ritornare valori speciali per segnalare errori. Usa `raise` quando la funzione non può produrre un risultato valido secondo la sua specifica.

### Esempio Completo: `get_ratios`

```python
def get_ratios(L1, L2):
    """
    Assumes: L1 e L2 sono liste di numeri di uguale lunghezza.
    Returns: lista contenente L1[i]/L2[i]
    """
    ratios = []
    for index in range(len(L1)):
        try:
            ratios.append(L1[index] / L2[index])
        except ZeroDivisionError:
            ratios.append(float('nan'))    # nan = not a number
        except:
            raise ValueError('get_ratios chiamata con argomenti errati')
    return ratios
```

### Esempio: Gestione Voti Studenti

```python
def avg(grades):
    return sum(grades) / len(grades)   # ZeroDivisionError se grades è vuota!
```

**Opzione 1 — Notificare e restituire `None`:**
```python
def avg(grades):
    try:
        return sum(grades) / len(grades)
    except ZeroDivisionError:
        print('warning: no grades data')
```

**Opzione 2 — Restituire 0.0 come policy:**
```python
def avg(grades):
    try:
        return sum(grades) / len(grades)
    except ZeroDivisionError:
        print('warning: no grades data')
        return 0.0
```

---

## ✅ Asserzioni (`assert`)

Le asserzioni sono uno strumento di **programmazione difensiva**: garantiscono che certe condizioni siano vere durante l'esecuzione. Se la condizione fallisce, viene sollevato un `AssertionError`.

```python
def avg(grades):
    assert len(grades) != 0, 'no grades data'
    return sum(grades) / len(grades)
```

> [!info] Comportamento
> - Condizione **vera** → esecuzione prosegue normalmente
> - Condizione **falsa** → `AssertionError` con il messaggio specificato

### Quando Usare le Asserzioni

> [!tip] Obiettivo: trovare i bug il prima possibile

- Controllare i **tipi** dei parametri/valori in ingresso
- Verificare **invarianti** su strutture dati
- Controllare il rispetto di **vincoli sui valori di ritorno**
- Rilevare **violazioni di vincoli** sulle funzioni (es. nessun duplicato in una lista)

### `assert` vs `raise`

| | `assert` | `raise` |
|---|---|---|
| Scopo | Debugging, programmazione difensiva | Gestione di errori attesi a runtime |
| Quando | Condizioni che non dovrebbero mai verificarsi | Condizioni di errore prevedibili |
| Disabilitabile | Sì (con flag `-O`) | No |
| Eccezione sollevata | `AssertionError` | Qualunque eccezione specificata |

---

## 🔗 Collegamenti

- [[01 - Introduzione a Python]]
- [[04 - Passaggio Parametri, args, kwargs]]
- [[06 - Oggetti e Classi in Python]]
- [[Modulo os]]
- [[Debugging Python]]

---

## 📚 Risorse Utili

- [Python Tutor – visualizzazione esecuzione](http://www.pythontutor.com/)
- [Python Exceptions Docs](https://docs.python.org/3/library/exceptions.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)
