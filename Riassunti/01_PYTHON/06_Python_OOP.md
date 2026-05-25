# 06 - Object-Oriented Programming (OOP) in Python

#python #advanced-computer-programming #OOP #classi #ereditarietà #polimorfismo #interfacce

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- Tony Gaddis. *Introduzione a Python*. 5° ed. – Pearson, 2021
- Paul J. Deitel, Harvey M. Deitel. *Introduzione a Python. Per l'informatica e la data science.* Pearson, 2021
- [Think Like a Computer Scientist](https://runestone.academy/runestone/books/published/thinkcspy/index.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)

---

## 📌 Concetti Chiave

- Oggetti come astrazioni di dati (attributi + metodi)
- Definizione di classi con `class`, `__init__`, `self`
- Metodi getter/setter e information hiding
- Variabili di istanza vs variabili di classe (statiche)
- Metodi statici e decoratore `@staticmethod`
- Metodi speciali (dunder): `__str__`, `__add__`, `__eq__`, etc.
- Ereditarietà singola e multipla, problema del diamante
- Polimorfismo e Duck Typing
- Eccezioni come oggetti e gerarchia `BaseException`
- Classi astratte (`ABC`, `@abstractmethod`) e Interfacce

---

## 🧱 Oggetti e Classi in Python

> [!info] Cos'è un oggetto?
> Un oggetto è un'**astrazione di dati** che include:
> 1. Una **rappresentazione interna** → attributi membro
> 2. Un'**interfaccia** per l'interazione → metodi membro (information hiding)

Ogni valore in Python è già un oggetto istanza di una classe:
- `1234` → istanza di `int`
- `"hello"` → istanza di `str`
- `[1,5,7]` → istanza di `list`

Gli oggetti possono essere **creati**, **manipolati** e **distrutti**. Python gestisce la memoria tramite **garbage collection**.

---

## 🏗️ Definizione di una Classe

```python
class Coordinate(object):    # Coordinate eredita da object (classe base)
    def __init__(self, x, y):
        self.x = x           # instance variable
        self.y = y           # instance variable
```

- `class` definisce un nuovo tipo
- `object` come argomento indica che la classe eredita dalla classe base Python
- `__init__` è il **costruttore**, invocato alla creazione di ogni istanza
- `self` rappresenta l'istanza corrente (analogo a `this` in Java/C++)

### Creare un'istanza

```python
c      = Coordinate(3, 4)
origin = Coordinate(0, 0)
print(c.x)       # → 3
print(origin.x)  # → 0
```

---

## ⚙️ Metodi di una Classe

I metodi sono **attributi procedurali**: funzioni definite nel contesto di una classe. Python passa sempre l'oggetto come **primo parametro** (`self`).

```python
class Coordinate(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def distance(self, other):
        x_diff_sq = (self.x - other.x) ** 2
        y_diff_sq = (self.y - other.y) ** 2
        return (x_diff_sq + y_diff_sq) ** 0.5
```

**Due modi equivalenti per chiamare un metodo:**
```python
c    = Coordinate(3, 4)
zero = Coordinate(0, 0)

c.distance(zero)               # modo convenzionale ✅
Coordinate.distance(c, zero)   # modo equivalente esplicito
```

---

## ✨ Metodi Speciali (Dunder Methods)

I metodi con doppio underscore (`__nome__`) permettono di ridefinire il comportamento degli operatori built-in:

| Metodo | Operatore / Funzione | Descrizione |
|---|---|---|
| `__init__(self, ...)` | Costruttore | Inizializza l'oggetto |
| `__str__(self)` | `print(obj)` | Rappresentazione leggibile |
| `__add__(self, other)` | `self + other` | Addizione |
| `__sub__(self, other)` | `self - other` | Sottrazione |
| `__eq__(self, other)` | `self == other` | Uguaglianza |
| `__lt__(self, other)` | `self < other` | Minore di |
| `__len__(self)` | `len(self)` | Lunghezza |

### `__str__` — Rappresentazione Testuale

```python
# Senza __str__:
>>> print(c)
<__main__.Coordinate object at 0x7fa918510488>   # ← non informativo!

# Con __str__:
def __str__(self):
    return "<" + str(self.x) + "," + str(self.y) + ">"

>>> print(c)
<3,4>   # ✅
```

### `type()` e `isinstance()`

```python
c = Coordinate(3, 4)
type(c)                    # → <class '__main__.Coordinate'>
isinstance(c, Coordinate)  # → True
```

---

## 🔐 Getter, Setter e Information Hiding

```python
class Animal(object):
    def __init__(self, age):
        self.age  = age
        self.name = None

    def get_age(self):   return self.age
    def get_name(self):  return self.name
    def set_age(self, newage):     self.age  = newage
    def set_name(self, newname=""): self.name = newname

    def __str__(self):
        return "animal:" + str(self.name) + ":" + str(self.age)
```

> [!important] Information Hiding
> Usare sempre getter/setter **al di fuori della classe** per accedere ai dati membro. Se la rappresentazione interna cambia (es. `self.age` → `self.years`), solo i metodi interni devono essere aggiornati.

> [!warning] Python e l'information hiding
> Python **non applica** l'information hiding in modo rigoroso:
> - ✅ `a.get_age()` → uso corretto
> - ⚠️ `a.age` → accesso diretto (consentito, ma sconsigliato)
> - ⚠️ `a.age = 'infinite'` → scrittura diretta dall'esterno (consentito, ma sconsigliato)
> - ⚠️ `a.size = "tiny"` → crea un nuovo attributo dall'esterno (consentito, ma sconsigliato)

---

## 📊 Variabili di Classe vs Variabili di Istanza

| | **Variabile di Istanza** | **Variabile di Classe (Statica)** |
|---|---|---|
| Dove definita | In `__init__` con `self.x` | Nel corpo della classe, fuori dai metodi |
| Condivisa | ❌ Ogni oggetto ha la sua copia | ✅ Condivisa da tutte le istanze |
| Accesso | `self.x` o `obj.x` | `ClassName.x` (preferito) |

```python
class Rabbit(Animal):
    tag = 1                                        # variabile di classe

    def __init__(self, age, parent1=None, parent2=None):
        Animal.__init__(self, age)
        self.parent1 = parent1                     # variabile di istanza
        self.parent2 = parent2
        self.rid     = Rabbit.tag                  # assegna l'id univoco
        Rabbit.tag  += 1                           # incrementa il contatore globale
```

---

## 🔧 Metodi Statici (`@staticmethod`)

I metodi statici appartengono alla classe ma **non ricevono `self`** come parametro. Si usano per operazioni non legate a singole istanze.

```python
class Rabbit(Animal):
    tag = 1

    @staticmethod
    def get_tag():
        return Rabbit.tag    # accede alla variabile di classe

# Possono essere chiamati sia sulla classe che su un'istanza:
Rabbit.get_tag()    # → valore corrente del tag
r1.get_tag()        # → stesso risultato
```

> [!info] Nota sui Decoratori
> I **decoratori** sono uno strumento per modificare il comportamento di una funzione/classe senza alterarla permanentemente, facendo il *wrap* della funzione originale.

---

## 🌳 Ereditarietà

```
object
  └── Animal
        ├── Cat
        ├── Rabbit
        └── Person
              └── Student
```

Una **sottoclasse** (classe figlia):
- **Eredita** tutti i dati e i metodi della superclasse
- Può **aggiungere** nuovi attributi e metodi
- Può **sovrascrivere** (override) metodi esistenti

### Ricerca dei Metodi nella Gerarchia

1. Si cerca il metodo nella **definizione della classe corrente**
2. Se non trovato, si risale la gerarchia
3. Si usa il **primo metodo trovato**

### Esempio: Gerarchia Animal → Person → Student

```python
class Animal(object):
    def __init__(self, age):
        self.age  = age
        self.name = None
    def __str__(self):
        return "animal:" + str(self.name) + ":" + str(self.age)

class Person(Animal):
    def __init__(self, name, age):
        Animal.__init__(self, age)      # chiama esplicitamente il costruttore padre
        self.set_name(name)
        self.friends = []

    def speak(self):     print("hello")
    def add_friend(self, fname):
        if fname not in self.friends: self.friends.append(fname)
    def age_diff(self, other):
        print(abs(self.age - other.age), "year difference")
    def __str__(self):
        return "person:" + str(self.name) + ":" + str(self.age)

class Student(Person):
    def __init__(self, name, age, major=None):
        Person.__init__(self, name, age)
        self.major = major

    def speak(self):
        import random
        r = random.random()
        if r < 0.25:         print("i have homework")
        elif r < 0.5:        print("i need sleep")
        elif r < 0.75:       print("i should eat")
        else:                print("i am watching tv")
    def __str__(self):
        return "student:" + str(self.name) + ":" + str(self.age) + ":" + str(self.major)
```

### Ereditarietà Multipla

```python
class Derived(Base1, Base2):
    pass
```

> [!warning] Problema del Diamante (Diamond Problem)
> Si verifica quando:
> - `B` e `C` ereditano da `A`
> - `D` eredita da **entrambe** `B` e `C`
> - `B` e/o `C` fanno override di un metodo `m` definito in `A`
> - Quale versione di `m` eredita `D`?
>
> Python risolve con il **MRO (Method Resolution Order)**: usa l'algoritmo C3 linearization, che segue l'ordine `D → B → C → A`.
> ```python
> class D(B, C): pass
> d = D()
> d.m()   # → chiama "m of B called" (B viene prima di C nell'ordine)
> ```

---

## 🔄 Overloading degli Operatori

### Esempio: `__add__` per la classe Rabbit

```python
def __add__(self, other):
    return Rabbit(0, self, other)   # crea un nuovo Rabbit con self e other come genitori
# r4 = r1 + r2  ←  equivalente a Rabbit(0, r1, r2)
```

### Esempio: `__eq__` per la classe Rabbit

```python
def __eq__(self, other):
    parents_same     = self.parent1.rid == other.parent1.rid \
                    and self.parent2.rid == other.parent2.rid
    parents_opposite = self.parent2.rid == other.parent1.rid \
                    and self.parent1.rid == other.parent2.rid
    return parents_same or parents_opposite
# Due Rabbit sono uguali se hanno gli stessi due genitori
```

> [!warning]
> Non comparare gli oggetti direttamente (`self.parent1 == other.parent1`) in `__eq__`: causerebbe ricorsione infinita fino a `None.parent1` → `AttributeError`.

---

## 🚨 Eccezioni ed Ereditarietà

Ogni eccezione è un oggetto che eredita dalla classe base `BaseException`.

```
BaseException
├── SystemExit            ← termina il programma senza traceback
├── KeyboardInterrupt     ← Ctrl+C dall'utente
├── GeneratorExit         ← chiusura di un generatore
└── Exception             ← classe base per le eccezioni comuni
      ├── ZeroDivisionError
      ├── NameError
      ├── ValueError
      ├── TypeError
      ├── StatisticsError
      └── ... (molte altre)
```

### Definire Eccezioni Personalizzate

```python
class Error(Exception):
    """Classe base per eccezioni personalizzate."""
    pass

class ValueTooSmallError(Error):
    """Sollevata quando il valore è troppo piccolo."""
    pass

class ValueTooLargeError(Error):
    """Sollevata quando il valore è troppo grande."""
    pass
```

---

## 🦆 Polimorfismo e Duck Typing

> [!info] Polimorfismo
> Le sottoclassi possono avere metodi con lo **stesso nome** della superclasse. Python invoca il metodo corretto a **runtime** (late binding / dynamic dispatch).

```python
animals = [a, c, p1, p2, s1, s2, r1, r2]
for animal in animals:
    print(animal)   # chiama __str__ della classe CORRENTE di ogni oggetto
```

Output:
```
animal::4
cat:fluffy:5
person:jack:30
student:alice:20:CS
rabbit:001
...
```

> [!quote] Duck Typing
> *"When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck."* — James Whitcomb Riley
>
> In Python, l'idoneità di un oggetto è determinata dalla **presenza di metodi e proprietà** (non dal tipo effettivo). Se un oggetto ha i metodi giusti, può essere usato nel contesto richiesto.

---

## 🔲 Classi Astratte e Interfacce

### Classi Astratte con `abc`

Una **classe astratta** definisce un insieme di metodi che le sottoclassi **devono** implementare. Non può essere istanziata direttamente.

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def doAction(self):
        pass

    def status(self):             # metodo concreto (non astratto)
        print("I am alive")

class Human(Animal):
    def doAction(self): print("I can walk and run")

class Snake(Animal):
    def doAction(self): print("I can crawl")

# Animal()  ← TypeError: Can't instantiate abstract class Animal
Human().doAction()    # → "I can walk and run"
Human().status()      # → "I am alive"
```

> [!important] Regola `@abstractmethod`
> Se una classe ha `@abstractmethod` ma **non** il decoratore, il metodo è concreto e la classe può essere istanziata senza errori. Il decoratore è necessario per forzare l'implementazione.

### Interfacce in Python

| Tipo | Descrizione |
|---|---|
| **Interfaccia Informale** | Classe normale con metodi non implementati (`pass`); sfrutta il duck typing |
| **Interfaccia Formale** | Usa `abc.ABC` + `@abstractmethod` su **tutti** i metodi; solleva `NotImplementedError` |

```python
# Interfaccia Formale
import abc

class MyInterface(abc.ABC):
    @abc.abstractmethod
    def metodo1(self): raise NotImplementedError
    @abc.abstractmethod
    def metodo2(self): raise NotImplementedError

class MyInterfaceImpl(MyInterface):
    def metodo1(self): print("metodo1")
    def metodo2(self): print("metodo2")

# MyInterface()      ← TypeError
c = MyInterfaceImpl()  # ✅ OK
c.metodo1()
```

---

## 🔗 Collegamenti

- [[05 - Gestione dei File e delle Eccezioni]]
- [[04 - Passaggio Parametri, args, kwargs]]
- [[07 - Iteratori e Generatori]]
- [[Design Patterns Python]]
- [[Ereditarietà e Polimorfismo]]

---

## 📚 Risorse Utili

- [Python Data Model – Dunder Methods](https://docs.python.org/3/reference/datamodel.html#basic-customization)
- [Python abc Module](https://docs.python.org/3/library/abc.html)
- [Think Python – Allen Downey](https://greenteapress.com/thinkpython2/thinkpython2.pdf)
