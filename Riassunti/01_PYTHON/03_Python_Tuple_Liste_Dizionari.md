# 03 - Tuple, Liste e Dizionari

#python #advanced-computer-programming #tuple #liste #dizionari #strutture-dati

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- Tony Gaddis. *Introduzione a Python*. 5° ed. – Pearson, 2021
- Paul J. Deitel, Harvey M. Deitel. *Introduzione a Python. Per l'informatica e la data science.* Pearson, 2021
- [Think Like a Computer Scientist](https://runestone.academy/runestone/books/published/thinkcspy/index.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)

---

## 📌 Concetti Chiave

- **Tuple**: sequenza ordinata, immutabile
- **Liste**: sequenza ordinata, mutabile; metodi principali
- Alias, clonazione e mutazione di liste
- Pericoli dell'iterazione su liste che si modificano
- **Dizionari**: struttura chiave-valore, operazioni principali
- Differenze tra Liste e Dizionari
- Caso d'uso: analizzatore di frequenza testo

---

## 🔒 Tuple

Una **tupla** è una sequenza ordinata di elementi (anche di tipi diversi) che **non può essere modificata** (immutable).

```python
t = ()                  # tupla vuota
t = (2, "mit", 3)       # tupla con elementi misti

t[0]          # → 2
t[1:2]        # → ("mit",)       slicing
t[1:3]        # → ("mit", 3)     slicing
len(t)        # → 3

(2,"mit",3) + (5,6)   # → (2,"mit",3,5,6)  concatenazione
t[1] = 4              # ❌ ERRORE: can't modify object
```

> [!warning] Immutabilità
> Le tuple **non possono** essere modificate dopo la creazione. Qualsiasi tentativo di assegnazione ad un indice genera errore.

### Usi Comuni delle Tuple

**1. Swap di variabili** (modo Pythonic):
```python
# Metodo classico (con variabile temporanea)
temp = x
x = y
y = temp

# Metodo Pythonic ✅
(x, y) = (y, x)
```

**2. Ritornare più valori da una funzione:**
```python
def quotient_and_remainder(x, y):
    q = x // y
    r = x % y
    return (q, r)

(quot, rem) = quotient_and_remainder(4, 5)
# quot = 0, rem = 4
```

### Iterazione su Tuple ed Esempio Avanzato

```python
def get_data(aTuple):
    """
    aTuple: tupla di tuple del tipo (int, string)
    Ritorna (min, max degli interi, numero di stringhe uniche)
    """
    nums = ()
    words = ()
    for t in aTuple:
        nums = nums + (t[0],)
        if t[1] not in words:
            words = words + (t[1],)
    min_n = min(nums)
    max_n = max(nums)
    unique_words = len(words)
    return (min_n, max_n, unique_words)
```

---

## 📋 Liste

Una **lista** è una sequenza ordinata di elementi, accessibile tramite indice. A differenza delle tuple, è **mutabile**.

```python
a_list = []                    # lista vuota
L = [2, 'a', 4, [1, 2]]       # lista con tipi misti e lista annidata

len(L)      # → 4
L[0]        # → 2
L[2] + 1    # → 5
L[3]        # → [1, 2]  (lista annidata!)
L[4]        # → IndexError
```

> [!info] Notazione punto (OOP)
> Le liste sono **oggetti Python**. I metodi si invocano con `object_name.do_something()`, esattamente come in Java/C++.

### Cambiare Elementi

```python
L = [2, 1, 3]
L[1] = 5     # → L è ora [2, 5, 3]
```

### Iterare su una Lista

```python
total = 0
for i in range(len(L)):    # con indice
    total += L[i]

# Metodo Pythonic ✅
total = 0
for i in L:                # direttamente sugli elementi
    total += i
```

---

## 🛠️ Metodi delle Liste

### Aggiungere Elementi

| Metodo | Descrizione | Muta la lista? |
|---|---|---|
| `L.append(e)` | Aggiunge `e` alla fine | ✅ Sì |
| `L.extend(L2)` | Aggiunge tutti gli elementi di `L2` alla fine | ✅ Sì |
| `L1 + L2` | Concatenazione → nuova lista | ❌ No |

```python
L = [2, 1, 3]
L.append(5)          # → [2, 1, 3, 5]

L1 = [2, 1, 3]
L2 = [4, 5, 6]
L3 = L1 + L2         # → [2, 1, 3, 4, 5, 6]  (L1, L2 invariate)
L1.extend([0, 6])    # → L1 = [2, 1, 3, 0, 6]  (L1 mutata)
```

### Rimuovere Elementi

| Metodo | Descrizione | Muta? | Ritorna |
|---|---|---|---|
| `del(L[index])` | Rimuove per indice | ✅ | — |
| `L.pop()` | Rimuove l'ultimo elemento | ✅ | L'elemento rimosso |
| `L.remove(e)` | Rimuove la prima occorrenza di `e` | ✅ | — (errore se non trovato) |

```python
L = [2, 1, 3, 6, 3, 7, 0]
L.remove(2)   # → [1, 3, 6, 3, 7, 0]
L.remove(3)   # → [1, 6, 3, 7, 0]  (rimuove la PRIMA occorrenza)
del(L[1])     # → [1, 3, 7, 0]
L.pop()       # ritorna 0, → [1, 3, 7]
```

### Ordinamento

```python
L = [9, 6, 0, 3]
sorted(L)    # → ritorna [0, 3, 6, 9]  ✅ L NON viene mutata
L.sort()     # → muta L = [0, 3, 6, 9]  ⚠️ ritorna None
L.reverse()  # → muta L = [9, 6, 3, 0]
```

> [!warning] `sort()` vs `sorted()`
> - `sorted(L)` → **non muta** la lista, ritorna una nuova lista ordinata
> - `L.sort()` → **muta** la lista, ritorna `None`

### Conversione Lista ↔ Stringa

```python
s = "I<3cs"
list(s)           # → ['I', '<', '3', '', 'c', 's']  (ogni carattere)
s.split('<')      # → ['I', '3cs']                     (split su '<')
s.split()         # split sullo spazio (default)

L = ['a', 'b', 'c']
''.join(L)         # → "abc"
'_'.join(L)        # → "a_b_c"
```

---

## ⚠️ Liste in Memoria: Alias e Clonazione

> [!warning] Effetti collaterali (Side Effects)
> Le liste sono oggetti in memoria. Una variabile **punta** all'oggetto, non lo copia. Più variabili che puntano alla stessa lista vengono tutte modificate da qualsiasi cambiamento.

```python
warm = ['red', 'yellow', 'orange']
hot = warm           # hot è un ALIAS di warm → stesso oggetto in memoria!

hot.append('pink')
print(warm)          # → ['red', 'yellow', 'orange', 'pink']  ← modificata!
```

### Clonazione (copia indipendente)

```python
chill = cool[:]      # crea una NUOVA lista con gli stessi elementi
```

> [!tip]
> Usare `[:]` per creare una copia indipendente ed evitare effetti collaterali.

---

## ⚠️ Mutazione e Iterazione su Liste

> [!danger] Non modificare una lista mentre ci si itera sopra!

```python
# ❌ SBAGLIATO — comportamento inatteso
def remove_dups(L1, L2):
    for e in L1:
        if e in L2:
            L1.remove(e)

L1 = [1, 2, 3, 4]
L2 = [1, 2, 5, 6]
remove_dups(L1, L2)
# Risultato: L1 = [2, 3, 4]  invece di [3, 4]
# Perché: Python usa un contatore interno che non aggiorna dopo remove()
```

```python
# ✅ CORRETTO — iterare su una copia
def remove_dups(L1, L2):
    L1_copy = L1[:]         # clona L1
    for e in L1_copy:
        if e in L2:
            L1.remove(e)
```

---

## 📖 Dizionari

> [!info] Definizione
> Un **dizionario** è una struttura dati che associa **chiavi** a **valori**. A differenza delle liste, l'accesso avviene tramite chiave e non tramite indice intero.

```python
my_dict = {}
grades = {'Ana': 'B', 'John': 'A+', 'Denise': 'A', 'Katy': 'A'}
```

### Lookup

```python
grades['John']    # → 'A+'
grades['Sylvan']  # → KeyError  (chiave non esistente)
```

### Operazioni Principali

```python
# Aggiungere / aggiornare una entry
grades['Sylvan'] = 'A'

# Verificare se una chiave esiste
'John' in grades      # → True
'Daniel' in grades    # → False

# Cancellare una entry
del(grades['Ana'])

# Ottenere tutte le chiavi
grades.keys()          # → dict_keys(['Denise', 'Katy', 'John', 'Ana'])

# Ottenere tutti i valori
grades.values()        # → dict_values(['A', 'A', 'A+', 'B'])
```

### Chiavi e Valori: Regole

| | **Chiavi** | **Valori** |
|---|---|---|
| Tipo | Solo tipi **immutabili** (int, float, str, tuple, bool) | Qualunque tipo (mutabile o immutabile) |
| Unicità | Devono essere **uniche** | Possono essere duplicate |
| Ordinamento | **Nessun ordinamento** garantito | **Nessun ordinamento** garantito |

```python
# Dizionario con tipi misti
d = {4: {1: 0}, (1, 3): "twelve", 'const': [3.14, 2.7, 8.44]}
```

> [!warning]
> Fare attenzione con `float` come chiavi: i problemi di precisione numerica possono generare comportamenti inattesi.

---

## ⚖️ Liste vs Dizionari

| Caratteristica | Lista | Dizionario |
|---|---|---|
| Struttura | Sequenza ordinata | Associazione chiave-valore |
| Accesso | Tramite **indice intero** | Tramite **chiave** (qualunque tipo immutabile) |
| Ordinamento | ✅ Garantito | ❌ Non garantito |
| Uso tipico | Collezioni omogenee ordinate | Mappatura di concetti correlati |

---

## 🎵 Caso d'uso: Analizzatore di Frequenza Testo

Esempio completo che usa dizionari per analizzare la frequenza di parole in un testo (es. testo di una canzone).

### 1. Costruire il dizionario di frequenza

```python
def lyrics_to_frequencies(lyrics):
    myDict = {}
    for word in lyrics:
        if word in myDict:
            myDict[word] += 1
        else:
            myDict[word] = 1
    return myDict
```

### 2. Trovare le parole più frequenti

```python
def most_common_words(freqs):
    values = freqs.values()
    best = max(values)
    words = []
    for k in freqs:
        if freqs[k] == best:
            words.append(k)
    return (words, best)    # ritorna (lista parole, frequenza massima)
```

### 3. Trovare parole con frequenza ≥ X

```python
def words_often(freqs, minTimes):
    result = []
    done = False
    while not done and len(freqs) > 0:
        temp = most_common_words(freqs)
        if temp[1] >= minTimes:
            result.append(temp)
            for w in temp[0]:
                del(freqs[w])   # muta il dizionario per trovare la prossima
        else:
            done = True
    return result

print(words_often(beatles, 5))
```

> [!tip] Idea chiave
> Trovare la parola più frequente, cancellarla dal dizionario, ripetere. Funziona perché stiamo **mutando** il dizionario ad ogni iterazione.

---

## 🔗 Collegamenti

- [[01 - Introduzione a Python]]
- [[02 - Moduli e Package Python]]
- [[04 - Oggetti e Classi in Python]]
- [[Strutture Dati Python]]
- [[Mutabilità e Immutabilità]]

---

## 📚 Risorse Utili

- [Python Data Structures Docs](https://docs.python.org/3/tutorial/datastructures.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)
