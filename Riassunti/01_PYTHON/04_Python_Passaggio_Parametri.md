# 04 - Passaggio Parametri, *args, **kwargs

#python #advanced-computer-programming #funzioni #parametri #args #kwargs

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte

---

## 📌 Concetti Chiave

- Variabili come riferimenti a oggetti in memoria (`id()`)
- Passaggio per riferimento: comportamento con oggetti mutabili e immutabili
- Differenza tra modifica in-place e riassegnazione del riferimento
- Argomenti a lunghezza variabile: `*args`
- Keyword arguments e `**kwargs`
- Argomenti con valori di default
- Unpacking di liste/tuple come argomenti

---

## 🔗 Riferimenti e Oggetti in Memoria

> [!info] Tutto in Python è un oggetto
> Quando si dichiara una variabile, essa **non contiene il valore** direttamente, ma un **riferimento** (puntatore) alla posizione in memoria dove il valore è memorizzato.

```python
x = 7
y = 10
print("x=", x, "ref_x=", id(x))   # id() restituisce l'indirizzo in memoria
print("y=", y, "ref_y=", id(y))

# Output:
# x= 7  ref_x= 4308052464
# y= 10 ref_y= 4308052560
```

```
x  ──→  [ 7  ]   (oggetto in memoria)
y  ──→  [ 10 ]   (oggetto in memoria)
```

---

## 📨 Passaggio di Parametri

> [!important] Regola fondamentale
> In Python **tutti i parametri sono passati per riferimento**: il riferimento del parametro attuale viene **copiato** nel parametro formale. La funzione ha quindi accesso diretto all'oggetto in memoria.

```python
def fun(x):
    print("Inside fun: x=", x, "ref_x=", id(x))

x = 7
fun(x)
print("Outside fun: x=", x, "ref_x=", id(x))

# Output:
# Inside fun:  x= 7  ref_x= 4308052464
# Outside fun: x= 7  ref_x= 4308052464  ← stesso riferimento!
```

Il comportamento **concreto** dipende dal tipo dell'oggetto passato: **mutabile** o **immutabile**.

---

## 🔄 Oggetti Mutabili (es. Liste)

Con oggetti mutabili, le modifiche **in-place** fatte dentro la funzione si riflettono sul chiamante (stesso oggetto in memoria).

```python
def fun(mylist):
    mylist.append([1, 2, 3, 4])    # modifica in-place
    print("Inside fun: val=", mylist, "ref=", id(mylist))

mylist = [10, 20, 30]
fun(mylist)
print("Outside fun: val=", mylist, "ref=", id(mylist))

# Output:
# Inside fun:  val= [10, 20, 30, [1, 2, 3, 4]]  ref= 4353638528
# Outside fun: val= [10, 20, 30, [1, 2, 3, 4]]  ref= 4353638528  ← modificata!
```

### ⚠️ Eccezione: riassegnazione del riferimento

Se dentro la funzione il parametro viene **riassegnato** (non modificato in-place), il parametro diventa **locale** e le modifiche **non** si riflettono sul chiamante:

```python
def fun(mylist):
    mylist = [1, 2, 3, 4]          # riassegnazione → nuovo oggetto locale
    print("Inside fun: val=", mylist, "ref=", id(mylist))

mylist = [10, 20, 30]
fun(mylist)
print("Outside fun: val=", mylist, "ref=", id(mylist))

# Output:
# Inside fun:  val= [1, 2, 3, 4]   ref= 4353638464  ← riferimento DIVERSO
# Outside fun: val= [10, 20, 30]   ref= 4352610752  ← invariato!
```

---

## 🔒 Oggetti Immutabili (es. int, str, tuple)

Con oggetti immutabili, anche se la funzione ha il riferimento, **non può modificare l'oggetto**. Qualsiasi operazione crea un **nuovo oggetto** in memoria, lasciando l'originale intatto.

```python
def fun(x):
    print("Inside fun before: val=", x, "ref=", id(x))
    x **= 2                        # crea un nuovo oggetto (non modifica l'originale)
    print("Inside fun after:  val=", x, "ref=", id(x))

x = 3
fun(x)
print("Outside fun: val=", x, "ref=", id(x))

# Output:
# Inside fun before: val= 3  ref= 4308052336
# Inside fun after:  val= 9  ref= 4308052528  ← nuovo oggetto!
# Outside fun:       val= 3  ref= 4308052336  ← invariato
```

### Riepilogo: Mutabile vs Immutabile

| Tipo | Modifica in-place | Effetto sul chiamante |
|---|---|---|
| **Mutabile** (lista, dict, set) | `append`, `remove`, etc. | ✅ Sì, il chiamante vede la modifica |
| **Mutabile** — riassegnazione | `mylist = [...]` | ❌ No, il parametro diventa locale |
| **Immutabile** (int, str, tuple) | Qualsiasi operazione | ❌ No, si crea sempre un nuovo oggetto |

---

## 🎯 Argomenti a Lunghezza Variabile: `*args`

La sintassi `*args` permette di passare un **numero variabile di argomenti posizionali**. La funzione riceve una **tupla** chiamata `args`.

```python
def fun(*args):
    for arg in args:
        print(arg)

fun("first", "second", "third")
# Output:
# first
# second
# third
```

### Combinare con parametri obbligatori

```python
def fun(x, *args):
    print(x)
    for arg in args:
        print(arg)

fun(100, "first", "second", "third")
# Output:
# 100
# first
# second
# third
```

> [!tip]
> `x` riceve il primo argomento, `*args` raccoglie **tutti i rimanenti** in una tupla.

---

## 🔑 Keyword Arguments e `**kwargs`

### Keyword Arguments

Permettono di passare argomenti **per nome** anziché per posizione:

```python
def fun(name, surname, country):
    print(name, surname, country)

fun(surname="Rossi", country="Italy", name="Mario")
# Output: Mario Rossi Italy
```

### `**kwargs` — numero variabile di keyword arguments

La sintassi `**kwargs` raccoglie un numero variabile di keyword arguments in un **dizionario**:

```python
def fun(**kwargs):
    for key, value in kwargs.items():
        print(key, "=", value)

fun(day=8, month="March", year=2024)
# Output:
# day = 8
# month = March
# year = 2024
```

### Combinare con parametri obbligatori

```python
def fun(x, **kwargs):
    print(x)
    for key, value in kwargs.items():
        print(key, "=", value)

fun("Date:", day=8, month="March", year=2024)
# Output:
# Date:
# day = 8
# month = March
# year = 2024
```

### Riepilogo `*args` vs `**kwargs`

| Sintassi | Tipo ricevuto | Accesso | Uso |
|---|---|---|---|
| `*args` | `tuple` | `for arg in args` | N argomenti posizionali |
| `**kwargs` | `dict` | `for k, v in kwargs.items()` | N argomenti per nome |

---

## ⚙️ Argomenti di Default

Python permette di definire valori di default per gli argomenti. Se un valore non viene passato, viene usato il default.

```python
def fun(username, password="changeme", days=30):
    print(username, password, days)

fun("admin", "admin", 120)    # → admin admin 120
fun("user", "password")       # → user password 30   (days usa il default)
fun("operator")               # → operator changeme 30  (entrambi usano il default)
```

> [!warning] Regola sintattica obbligatoria
> Gli argomenti **con default** devono sempre essere dichiarati **dopo** quelli **senza default**.

```python
# ❌ ERRORE — argomento obbligatorio dopo argomento con default
def fun(username, password="changeme", days=30, email):
    ...
# SyntaxError: non-default argument follows default argument

# ✅ CORRETTO
def fun(username, email, password="changeme", days=30):
    ...
```

---

## 📦 Unpacking di Argomenti

L'operatore `*` può essere usato anche nella **chiamata** di una funzione per "spacchettare" una lista o tupla come argomenti separati:

```python
# Chiamata normale
list(range(3, 6))         # → [3, 4, 5]

# Chiamata con unpacking
args = [3, 6]
list(range(*args))        # equivalente a list(range(3, 6)) → [3, 4, 5]
```

> [!info] Utilità
> Utile quando gli argomenti sono già contenuti in una lista o tupla ma la funzione li richiede come parametri separati.

---

## 🔗 Collegamenti

- [[01 - Introduzione a Python]]
- [[03 - Tuple, Liste e Dizionari]]
- [[05 - Oggetti e Classi in Python]]
- [[Scope e Namespace]]
- [[Mutabilità e Immutabilità]]

---
