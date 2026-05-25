# 02 - Moduli e Package Python

#python #advanced-computer-programming #moduli #package

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- [Python Standard Library](http://docs.python.org/library/)
- [Tutorial sui moduli Python](http://docs.python.org/tutorial/modules.html)

---

## 📌 Concetti Chiave

- Cosa sono i moduli e perché usarli
- Come importare un modulo (`import`, `from ... import`)
- Moduli come script eseguibili e uso di `__name__`
- Search path e `sys.path` / `PYTHONPATH`
- Moduli standard e il modulo `sys`
- La funzione `dir()`
- Cosa sono i package e la struttura con `__init__.py`
- Importazione da package: `import *`, `__all__`
- Riferimenti intra-package: importazioni assolute e relative

---

## 📦 Cosa sono i Moduli?

I moduli risolvono un problema fondamentale: le definizioni di funzioni e variabili **non vengono salvate** all'uscita dall'interprete. Un modulo permette di:

- **Salvare** definizioni (funzioni, variabili, classi) in un file riutilizzabile
- **Eseguire** istruzioni come script eseguibili
- **Organizzare** il codice in unità logiche separate

> [!info] Definizione
> Un **modulo** è semplicemente un file con estensione `.py` contenente definizioni e istruzioni Python.

---

## 📥 Importare un Modulo

### Sintassi base

```python
import example            # importa l'intero modulo (senza estensione .py)
from function import func1, func2   # importa singole definizioni
```

> [!warning]
> Quando un modulo viene importato, **tutte** le sue dichiarazioni e definizioni vengono eseguite.

### Accedere ai componenti

```python
# Con import semplice → si usa la notazione punto
import example
example.func(3)

# Con from ... import → il nome del modulo può essere omesso
from example import func
func(3)
```

### Alias con `as`

```python
import numpy as np          # rinomina il modulo
from math import sqrt as sq # rinomina la funzione importata
```

---

## 🖥️ Moduli come Script

Uno script Python può essere **eseguito da riga di comando**:

```bash
python example.py <arguments>
```

Per distinguere se un modulo è eseguito direttamente o importato si usa la variabile `__name__`:

```python
if __name__ == "__main__":
    # questo codice viene eseguito SOLO se il file è avviato direttamente
    # NON viene eseguito se il modulo è importato
    main()
```

| Situazione | Valore di `__name__` |
|---|---|
| File eseguito direttamente | `"__main__"` |
| File importato come modulo | Nome del modulo (es. `"example"`) |

> [!tip] Collegamento
> Questo meccanismo è già stato visto in [[01 - Introduzione a Python]] nella sezione sulla funzione `main()`.

---

## 🔍 Search Path dei Moduli

Quando si esegue `import nomemodulo`, l'interprete cerca il modulo **nell'ordine seguente**:

1. **Directory corrente** del progetto
2. **`PYTHONPATH`** — variabile d'ambiente contenente un elenco di directory
3. **Percorso predefinito** dipendente dall'installazione di Python

> [!info] `sys.path`
> Tutte e tre le sorgenti (directory corrente + PYTHONPATH + default path) sono memorizzate nella variabile `sys.path`, che può essere ispezionta e modificata a runtime:
> ```python
> import sys
> print(sys.path)
> ```

---

## 📚 Moduli Standard

Python include una vasta **libreria standard** di moduli con operazioni integrate che non fanno parte del nucleo del linguaggio. Alcuni moduli dipendono dal sistema operativo.

### Il modulo `sys`

```python
import sys

sys.path          # lista dei percorsi di ricerca dei moduli
sys.ps1           # stringa del prompt primario (modificabile)
sys.ps2           # stringa del prompt secondario (modificabile)
```

```python
>>> import sys
>>> sys.ps1 = 'plurt> '
plurt> print('plurt')
plurt
```

### La funzione `dir()`

Restituisce un elenco di nomi (variabili e funzioni) definiti in un modulo:

```python
import math
dir(math)         # → lista di tutti i nomi definiti in math

dir()             # → tutti i nomi attualmente definiti nel namespace corrente
                  #   (esclude i nomi built-in)
```

---

## 📁 Package

### Definizione

> [!info]
> Un **package** è una raccolta organizzata di moduli — ovvero, **moduli all'interno di moduli**, organizzati in directory.

### Struttura

```
mypackage/
│
├── __init__.py          ← obbligatorio! rende la directory un package
├── modulo1.py
├── modulo2.py
│
└── subpackage/
    ├── __init__.py      ← ogni sottodirectory deve averne uno
    └── modulo3.py
```

### Il file `__init__.py`

- Istruisce l'interprete Python a **trattare la directory come un package**
- Può essere **vuoto** oppure contenere:
  - Codice di **inizializzazione** del package
  - La variabile `__all__` (lista dei moduli esportati con `import *`)

### Importare un Package

```python
import currency.conversion.euro        # importa il modulo euro

# accesso solo tramite notazione punto completa:
euro.convertfromdollar(10)
```

---

## 🔄 Importazione da un Package

### `import *` e `__all__`

```python
from example.subexample import *      # importa tutto dal sotto-package
```

Questo carica tutti i nomi contenuti nella variabile `__all__` definita in `__init__.py`.

> [!warning] Comportamento senza `__all__`
> Se `__all__` **non è definita**, `import *` da un package:
> - ❌ **Non importa** tutti i sottomoduli
> - ✅ **Importa** il package stesso (eseguendo `__init__.py`)
> - ✅ **Importa** tutti i nomi definiti in `__init__.py`
> - ❌ I sottomoduli **non vengono importati** a meno che non siano esplicitamente importati in `__init__.py`

---

## 🔗 Riferimenti Intra-Package

### Importazioni Assolute

Usate per fare riferimento a sottomoduli di package "fratelli":

```python
# Dal modulo sound.filters.vocoder, importare echo da sound.effects
from sound.effects import echo
```

### Importazioni Relative

Usano punti iniziali per indicare il package corrente e quello padre:

```python
from . import echo            # importa dal package CORRENTE
from .. import formats        # risale di un livello (package padre)
from ..filters import equalizer  # risale e accede a un sottopackage fratello
```

| Sintassi | Significato |
|---|---|
| `from . import x` | Dal package corrente |
| `from .. import x` | Dal package padre |
| `from ..pkg import x` | Da un package fratello |

---

## 🗂️ Package in Directory Multiple

La variabile `__path__` memorizza il percorso della cartella in cui si trova `__init__.py`. Modificarla consente a un package di cercare i propri moduli in **directory multiple**:

```python
# Dentro __init__.py
__path__ = ["/percorso/alternativo", "/altro/percorso"]
```

---

## 🔗 Collegamenti

- [[01 - Introduzione a Python]]
- [[03 - Oggetti e Classi in Python]]
- [[Namespace e Scope]]
- [[Libreria Standard Python]]

---

## 📚 Risorse Utili

- [Python Standard Library Docs](http://docs.python.org/library/)
- [Tutorial Moduli Python](http://docs.python.org/tutorial/modules.html)
