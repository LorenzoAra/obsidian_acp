# 01 - Introduzione a Python

#python #advanced-computer-programming #introduzione

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- Tony Gaddis. *Introduzione a Python*. 5° ed. – Pearson, 2021
- Paul J. Deitel, Harvey M. Deitel. *Introduzione a Python. Per l'informatica e la data science.* Pearson, 2021
- [Think Like a Computer Scientist](https://runestone.academy/runestone/books/published/thinkcspy/index.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)

---

## 📌 Concetti Chiave

- Differenza tra Compilatori e Interpreti
- Funzionamento dell'interprete CPython e della PVM
- Modalità di utilizzo di Python (Interactive vs Script)
- Oggetti scalari e non-scalari, Type Casting
- Stringhe: operazioni, indicizzazione, slicing, immutabilità
- I/O: `print()` e `input()`
- Operatori di confronto e logici
- Flusso di controllo: `if/elif/else`, `while`, `for`, `break`
- Funzioni: definizione, scope, funzioni come argomenti
- La variabile speciale `__name__`

---

## 🔄 Compilatori vs Interpreti

> [!info] Definizione
> - **Compilatori**: traducono il codice sorgente in un *programma separato* in linguaggio macchina, eseguibile in qualunque momento.
> - **Interpreti**: traducono **ed eseguono** le istruzioni una alla volta. Python usa questo approccio.

### CPython e la Python Virtual Machine (PVM)

- L'implementazione di riferimento è **CPython** (scritto in C e Python).
- Il codice sorgente **non** viene tradotto direttamente in linguaggio macchina, ma in **byte code**.
- Il byte code viene eseguito dalla **Python Virtual Machine (PVM)**.

```
Codice sorgente (.py)
        ↓  [Compilatore Python]
    Byte code
        ↓  [PVM]
  Istruzioni macchina
```

> [!warning] Differenza importante
> Il **byte code** ≠ linguaggio assembly. Il byte code è per una *virtual machine*, l'assembly è per una specifica CPU fisica.

### File `.pyc`

- I file `.pyc` sono **byte code pre-compilati**, generati automaticamente all'importazione/esecuzione di uno script.
- Permettono esecuzioni successive più rapide (non serve ricompilare).
- Per generarli manualmente:
```bash
python -m compileall filename.py
```

---

## 🖥️ Modalità di Utilizzo

| Modalità | Descrizione | Uso |
|---|---|---|
| **Interactive mode** | Prompt interattivo, istruzioni eseguite una alla volta | Test rapidi, esplorazione |
| **Script mode** | Istruzioni salvate in un file `.py`, eseguite tutte insieme | Programmi completi |

```bash
# Esecuzione script
python filename.py
```

### IDEs Consigliati

- **VSCodium** (con estensioni: Code Runner, Jupyter, Python, Python Debugger)
- **Anaconda Navigator** / **Jupyter Notebook**
- **PyCharm**
- Terminale

> [!warning] Nota VSCodium
> Le estensioni **non sostituiscono** l'interprete Python: è necessario installarlo separatamente sul PC.

---

## 🧱 Oggetti Python

I programmi Python manipolano **data objects**. Ogni oggetto ha un **tipo** che ne definisce le operazioni possibili.

| Categoria | Descrizione |
|---|---|
| **Scalari** | Non divisibili (int, float, bool, NoneType) |
| **Non-scalari** | Hanno struttura interna accessibile (stringhe, liste, tuple, dizionari) |

### Tipi Scalari

| Tipo | Descrizione | Esempio |
|---|---|---|
| `int` | Interi | `5` |
| `float` | Numeri reali | `3.27` |
| `bool` | Booleani | `True`, `False` |
| `NoneType` | Assenza di valore | `None` |

```python
type(5)    # → int
type(3.0)  # → float
```

### Type Casting

```python
float(3)   # → 3.0    (int → float)
int(3.9)   # → 3      (float → int, tronca)
str(42)    # → "42"   (int → string)
```

---

## 🔢 Operatori Aritmetici

| Operatore | Operazione | Note |
|---|---|---|
| `i + j` | Somma | |
| `i - j` | Differenza | |
| `i * j` | Prodotto | |
| `i / j` | Divisione | Risultato sempre `float` |
| `i % j` | Modulo | Resto della divisione |
| `i ** j` | Potenza | |

> [!tip]
> Se **entrambi** gli operandi sono `int` → risultato `int` (eccetto `/`).  
> Se almeno uno è `float` → risultato `float`.

---

## 📋 Operatori di Confronto e Logici

### Operatori di confronto (restituiscono `bool`)

```python
i > j   # maggiore
i >= j  # maggiore o uguale
i < j   # minore
i <= j  # minore o uguale
i == j  # uguaglianza
i != j  # disuguaglianza
```

### Operatori logici

| Espressione | Risultato |
|---|---|
| `not a` | `True` se `a` è `False`, viceversa |
| `a and b` | `True` solo se entrambi `True` |
| `a or b` | `True` se almeno uno è `True` |

---

## 📝 Stringhe

Le stringhe sono **sequenze case-sensitive di caratteri**, definite con `"..."` o `'...'`.

### Operazioni Base

```python
s = "abc"
len(s)         # → 3
s + "def"      # concatenazione → "abcdef"
"ha" * 3       # ripetizione → "hahaha"
```

### Indicizzazione

```python
s = "abc"
s[0]   # → "a"   (primo elemento)
s[-1]  # → "c"   (ultimo elemento)
s[3]   # → IndexError (fuori range)
```

> [!info] Indici
> - Gli indici **positivi** partono da `0` (sinistra)
> - Gli indici **negativi** partono da `-1` (destra)

### Slicing `[start:stop:step]`

```python
s = "abcdefgh"
s[3:6]    # → "def"         (start=3, stop=6, step=1 default)
s[3:6:2]  # → "df"          (step=2)
s[::]     # → "abcdefgh"    (tutta la stringa)
s[::-1]   # → "hgfedcba"    (stringa invertita)
s[4:1:-2] # → "ec"
```

### Immutabilità

> [!warning] Le stringhe sono IMMUTABILI
> ```python
> s = "hello"
> s[0] = 'y'           # ❌ ERRORE
> s = 'y' + s[1:]      # ✅ OK → "yello" (si crea un nuovo oggetto)
> ```

---

## 💬 Input / Output

### `print()`

```python
x = 1
print(x)                                      # → 1
print("il mio numero preferito è", x, ".")   # con virgole
print("il mio numero preferito è" + str(x))  # con concatenazione
```

### `input()`

```python
text = input("Scrivi qualcosa: ")        # restituisce sempre una stringa
num  = int(input("Scrivi un numero: "))  # cast necessario per numeri
```

> [!warning]
> `input()` restituisce **sempre una stringa**. Usare `int()` o `float()` per convertire se si vuole operare numericamente.

---

## 🔀 Flusso di Controllo

### `if / elif / else`

```python
if <condizione>:
    <istruzioni>
elif <condizione>:
    <istruzioni>
else:
    <istruzioni>
```

> [!important] L'indentazione in Python è OBBLIGATORIA
> Definisce i blocchi di istruzioni. Un'indentazione errata causa errori di sintassi.

> [!tip] `=` vs `==`
> - `=` → **assegnazione**
> - `==` → **confronto di uguaglianza**

### Ciclo `while`

```python
while <condizione>:
    <istruzioni>
```

1. Valuta la condizione come booleano
2. Se `True`, esegue il blocco
3. Rivaluta la condizione
4. Ripete finché la condizione è `False`

### Ciclo `for` e `range()`

```python
for <variabile> in range(<num>):
    <istruzioni>
```

**`range(start, stop, step)`** — default: `start=0`, `step=1`, esegue fino a `stop - 1`

```python
for i in range(7, 10):     # i = 7, 8, 9
    mysum += i

for i in range(5, 11, 2):  # i = 5, 7, 9
    mysum += i
```

### Istruzione `break`

- Esce **immediatamente** dal ciclo corrente (solo il più interno).
- Salta le istruzioni rimanenti nel blocco.

```python
for i in range(5, 11, 2):
    mysum += i
    if mysum == 5:
        break
    mysum += 1
```

### Iterazione su Stringhe (Pythonic)

```python
# Metodo classico (con indice)
for index in range(len(s)):
    if s[index] == 'i' or s[index] == 'u':
        print("c'è una i o una u")

# Metodo Pythonic ✅ (preferibile)
for char in s:
    if char == 'i' or char == 'u':
        print("c'è una i o una u")
```

---

## ⚙️ Funzioni

### Struttura

```python
def nome_funzione(parametro1, parametro2):
    """Docstring: descrive la funzione (opzionale ma consigliata)."""
    # corpo della funzione
    return valore
```

**Caratteristiche:**
- **Nome** identificativo
- **Parametri** (0 o più)
- **Docstring** (opzionale ma raccomandata — prima istruzione della funzione)
- **Corpo** con le istruzioni
- **`return`** per restituire un valore

> [!warning]
> Se non si usa `return`, Python restituisce automaticamente `None`.

```python
def is_even(i):
    """Input: i, intero positivo. Ritorna True se i è pari."""
    return i % 2 == 0

is_even(3)  # → False
is_even(4)  # → True
```

### Scope delle Variabili

- Ogni chiamata a funzione crea un **nuovo scope/frame/environment**.
- I parametri formali sono legati ai parametri effettivi all'invocazione.
- Dentro una funzione si può **leggere** una variabile dello scope esterno, ma **non modificarla**.

```python
def f(x):
    x = x + 1      # x locale alla funzione
    return x

x = 3
z = f(x)
# x rimane 3 nello scope globale
# z = 4
```

> [!warning] Variabili globali
> È tecnicamente possibile usare la keyword `global`, ma **non è una buona pratica**.

### Funzioni Annidate e come Argomenti

Le funzioni possono essere **definite dentro altre funzioni** e **passate come argomenti**.

```python
def func_a():
    print("inside func_a")

def func_b(y):
    print("inside func_b")
    return y

def func_c(z):
    print("inside func_c")
    return z()          # z è una funzione passata come argomento

print(func_a())         # None
print(5 + func_b(2))    # 7
print(func_c(func_a))   # chiama func_a → None
```

---

## 🚀 La Funzione `main()` e `__name__`

In Python **non esiste** una funzione `main()` obbligatoria. L'interprete esegue il codice dal livello 0 di indentazione.

La variabile built-in `__name__`:
- Vale `"__main__"` → il file viene **eseguito direttamente**
- Vale il **nome del modulo** → il file viene **importato** da un altro script

```python
print("Hello")

def main():
    print("hey there")

if __name__ == "__main__":
    main()
```

> [!tip] Best Practice
> Racchiudere il codice principale in `if __name__ == "__main__":` permette al file di funzionare sia come **script eseguibile** che come **modulo importabile**.

---

## 🔗 Collegamenti

- [[02 - Tuple, Liste e Dizionari]]
- [[03 - Oggetti e Classi in Python]]
- [[Flusso di Controllo Python]]
- [[Funzioni Avanzate Python]]
- [[Scope e Namespace]]

---

## 📚 Risorse Utili

- [Python Tutor – visualizzazione scope](http://www.pythontutor.com/)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)
- [Think Like a Computer Scientist](https://runestone.academy/runestone/books/published/thinkcspy/index.html)
