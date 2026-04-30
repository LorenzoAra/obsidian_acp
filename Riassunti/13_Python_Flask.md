# 13 - Flask e RESTful Web Services in Python

#python #advanced-computer-programming #Flask #REST #HTTP #WebServices #Jinja2 #requests

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- L. Richardson, S. Ruby. *RESTful Web Services*, O'Reilly
- [Flask docs 3.0](https://flask.palletsprojects.com/en/3.0.x/)
- M. Grinberg. *Flask Web Development*, O'Reilly
- [requests library docs](https://requests.readthedocs.io/en/latest/)

---

## 📌 Concetti Chiave

- Web Service: definizione W3C, servizi invocabili via HTTP/URI
- REST: architettura, risorse, URI, interfaccia uniforme, stateless
- HTTP methods: GET, POST, PUT, DELETE e operazioni CRUD
- RPC-style vs REST-style: confronto
- Passi di progettazione REST; note sulle URI
- Cenni HTML: tag, DOM, headings, anchor, list, nav, footer
- Flask: microframework WSGI, Werkzeug, Jinja2
- Route e Dynamic Route con decorator `@app.route`
- View function, `app.run()`, debug mode
- HTML Templating con Jinja2: variabili, `if`, `for`
- `curl` da terminale: parametri utili
- Request Object: `get_data`, `get_json`, `request.args`
- Response Object: stringa, dict/list, tupla, `make_response`, cookie
- Route methods: `methods=`, shortcut `@app.get`, `@app.post`
- `requests` library: GET, POST, PUT, DELETE; query string, payload, Response Object

---

## 🌐 Web Service e REST

### Web Service (WS)

> [!info] Definizione W3C
> *"A Web service is a software system identified by a URI, whose public interfaces and bindings are defined and described using XML."*

Un servizio è un modulo software che:
- Espone funzionalità invocabili dai client
- Può essere **riusato** ed è **componibile**
- È accessibile tramite Internet con protocolli standard (HTTP, URI, XML, SOAP)

> [!warning]
> "Web service" ≠ "web server": il concetto si riferisce alla possibilità di usare richieste HTTP per **avviare l'esecuzione di un programma**.

### REST: Concetti Chiave

**REST** (Representational State Transfer) è uno stile architetturale per sistemi distribuiti. Rispetto a RPC, il focus è sulle **risorse** e non sulle procedure.

| Principio                | Descrizione                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| **Risorsa**              | Entità indirizzabile tramite Web, accessibile e trasferibile tra client e server            |
| **URI**                  | Ogni risorsa è identificata da un URI univoco                                               |
| **Interfaccia uniforme** | I client interagiscono con le risorse tramite un insieme **fisso** di metodi HTTP           |
| **Stateless**            | Ogni ciclo richiesta-risposta è un'interazione completa; non esiste il concetto di sessione |

### HTTP Methods e CRUD

| Metodo HTTP | Operazione CRUD | Request Body | Response Body |
|---|---|---|---|
| **GET** | Read (retrieve) | Vuoto | Rappresentazione della risorsa |
| **POST** | Create | Rappresentazione proposta | Vuoto / stato / copia risorsa |
| **PUT** | Update | Rappresentazione proposta | Vuoto / stato / copia risorsa |
| **DELETE** | Delete | Vuoto | Vuoto / messaggio di stato |

**Esempi di URI RESTful:**
```
POST   http://example.com/Orders/8    → CREATE
GET    http://example.com/Orders/1    → READ
PUT    http://example.com/Orders/12   → UPDATE
DELETE http://example.com/Orders/12   → DELETE
```

### RPC-style vs REST-style

| | **RPC-style** | **RESTful** |
|---|---|---|
| Vocabolario | Proprietario (nomi funzione diversi) | Fisso (metodi HTTP) |
| Esempio orders | `insertOrder(id)`, `getOrder(id)`, `updateOrder(id)`, `deleteOrder(id)` | `POST /Order/{id}`, `GET /Order/{id}`, `PUT /Order/{id}`, `DELETE /Order/{id}` |

### Note sulle URI

- Preferire **nomi a verbi**: `DELETE /book/15` ✅ vs `GET /book?isbn=15&action=delete` ❌
- **GET** è read-only (idempotente); **POST** è read-write e può modificare lo stato
- **POST**: per creare una risorsa *figlia* rispetto ad una padre
- **PUT**: per *modificare* risorse esistenti

### Passi di Progettazione REST

1. Identificare le **risorse** da esporre come servizi
2. Per ogni risorsa: 
	1. Definire le **URI**
	2. Esporre un subset adeguato dell'**interfaccia uniforme** (decidere quali metodi HTTP supportare)
	3. Progettare le **rappresentazioni** (JSON, XML, messaggi di stato...)

### Formati di Rappresentazione: XML e JSON

```json
// JSON — serie non ordinata di coppie nome/valore
{
  "id": "42",
  "name": "Laptop",
  "price": 999.99
}
```

```xml
<!-- XML — markup testuale con tag -->
<product>
  <id>42</id>
  <name>Laptop</name>
  <price>999.99</price>
</product>
```

Entrambi sono rappresentazioni **testuali** (diversamente dalla serializzazione binaria Java).

---

## 🌍 Cenni HTML

**HTML** (HyperText Markup Language) specifica la struttura degli elementi visivi di una web application tramite **tag**: `<name>...</name>`.

### Tag Principali

| Tag | Descrizione |
|---|---|
| `<h1>` … `<h6>` | Headings (titoli) di diversi livelli |
| `<a href="url">` | Anchor — link ipertestuale |
| `<ul>` / `<ol>` | Unordered list (bullet) / Ordered list (numerata) |
| `<li>` | List item |
| `<p>` | Paragrafo |
| `<img src="..." alt="...">` | Immagine |
| `<nav>` | Sezione di navigazione |
| `<header>` / `<footer>` | Intestazione / Piè di pagina |
| `<div class="...">` | Divisione generica (usata per stile/layout) |

### Document Object Model (DOM)

Il **DOM** è una rappresentazione gerarchica (ad albero) del documento HTML:
- I **tag HTML** definiscono i DOM elements
- Con il DOM si può interagire tramite scripting language (es. JavaScript)
- Tutti i tag sono organizzabili come un **tree**

---

## 🔥 Flask

### Cos'è Flask

**Flask** è un **web application framework WSGI leggero** (microframework) per Python:
- Consente la realizzazione **rapida** di web app, scalabile verso applicazioni complesse
- Nato come wrapper di **Werkzeug** e **Jinja2**
- Il core fornisce i servizi base; estendibile tramite **estensioni**

| Componente | Ruolo |
|---|---|
| **Werkzeug** | Routing, debugging, WSGI (Web Server Gateway Interface) |
| **Jinja2** | Template engine |

> [!info] WSGI
> WSGI (pronunciato *whiskey*) è una specifica che descrive come un web server comunica con le applicazioni web.

### Installazione

```bash
pip install Flask
```

Verifica:
```python
import flask   # nessun errore → installazione ok
```

---

## 🗺️ Route e View Function

### Struttura Base di un'App Flask

```python
from flask import Flask

app = Flask(__name__)   # __name__ = nome del modulo/package principale

@app.route("/")
def index():
    return "<h1>Hello, World!</h1>"

@app.route('/user/<name>')
def user(name):
    return '<h1>Hello, %s!</h1>' % name

if __name__ == "__main__":
    app.run(debug=True)
```

### Route e Decorator `@app.route`

- Una **route** è il mapping tra una URL e una funzione Python (**view function**)
- Il decorator `@app.route("/path")` registra la view function come gestore di quella URL
- Ad ogni richiesta sulla URL specificata, Flask invoca la view function
- La view function può ritornare **HTML** o **JSON**

### Dynamic Route

```python
@app.route('/user/<name>')
def user(name):
    return '<h1>Hello, %s!</h1>' % name
```

- `/user/` è la parte **statica**
- `<name>` è la parte **dinamica**: cattura il testo fino al prossimo `/`
- Flask passa il valore dinamico come **parametro** della funzione

### Server Startup

```python
app.run(debug=True)
# default: http://127.0.0.1:5000/ (solo localhost)

# Per renderlo accessibile dall'esterno:
app.run(host='0.0.0.0', port=80)
```

Output di avvio tipico:
```
* Running on http://127.0.0.1:5000
* Debug mode: on
127.0.0.1 -- "GET / HTTP/1.1" 200 -
```

> [!warning]
> Il server di sviluppo Flask **non va usato in produzione**. Usare un server WSGI di produzione (es. Gunicorn, uWSGI).

---

## 🎨 HTML Templating con Jinja2

### Motivazione

Ritornare HTML generato direttamente nella view function mescola **business logic** e **presentation logic**. La soluzione è usare i **template** di Jinja2.

- **Template**: file HTML con parti statiche e **placeholder** per le parti dinamiche
- **Rendering**: processo che sostituisce i placeholder con i valori correnti

### Setup

I template si collocano nella sottocartella `./templates/` (file `.html`).

```
myapp/
├── app.py
└── templates/
    ├── index.html
    └── user.html
```

```html
<!-- templates/index.html -->
<h1>Hello World!</h1>

<!-- templates/user.html -->
<h1>Hello, {{ name }}!</h1>
```

### `render_template`

```python
from flask import Flask, render_template

@app.route("/index")
def index():
    return render_template('index.html')

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)
```

### Variabili nei Template

Jinja2 riconosce qualsiasi tipo Python: stringhe, liste, dizionari, oggetti.

```html
<p>Da un dizionario: {{ mydict['key'] }}</p>
<p>Da una lista: {{ mylist[3] }}</p>
<p>Da un oggetto: {{ myobj.somemethod() }}</p>
```

### Strutture di Controllo

```html
<!-- Condizionale -->
{% if user %}
    Hello, {{ user }}!
{% else %}
    Hello, Stranger!
{% endif %}

<!-- Ciclo for -->
<ul>
{% for comment in comments %}
    <li>{{ comment }}</li>
{% endfor %}
</ul>
```

---

## 📥 Request Object

Flask rende accessibili le informazioni delle richieste HTTP tramite **context** e l'oggetto `request`.

> [!info]
> I context permettono di rendere `request` accessibile come variabile globale a un thread **senza interferire** con gli altri thread.

```python
from flask import Flask, request

@app.route('/')
def index():
    user_agent = request.headers.get('User-Agent')
    return '<p>Your browser is %s</p>' % user_agent
```

### Metodi Principali di `request`

| Metodo / Attributo | Descrizione |
|---|---|
| `request.headers.get('...')` | Recupera un header della richiesta |
| `request.get_data(as_text=False, cache=True, parse_form_data=False)` | Legge il body come `bytes` (o stringa se `as_text=True`) |
| `request.get_json(force=False, silent=False, cache=True)` | Parsing del body JSON (richiede `mimetype: application/json`; `force=True` ignora il mimetype) |
| `request.args` | Parametri nella query string (la parte dopo `?`) |
| `request.method` | Metodo HTTP della richiesta (`'GET'`, `'POST'`, ecc.) |

**Esempi:**
```python
# Accesso a dati JSON nel body
@app.post('/json')
def json():
    data = request.get_json()
    return data

# Accesso ai parametri nella query string
# test: curl http://127.0.0.1:5000/hello?name=pippo&surname=pippozzo
@app.get('/hello')
def hello():
    params = request.args
    print("name:", params['name'])       # pippo
    print("surname:", params['surname']) # pippozzo
    return params
```

---

## 📤 Response Object

Il valore di ritorno di una view function è **automaticamente convertito** in un oggetto `Response`.

### Logica di Conversione

| Valore di ritorno | Comportamento |
|---|---|
| `Response` object | Ritornato direttamente |
| **stringa** | `body=stringa`, `status=200 OK`, `mimetype=text/html` |
| **dizionario** o **lista** | `jsonify()` invocato automaticamente → `mimetype=application/json` |
| **iteratore/generator** (stringhe/byte) | Trattato come *streaming response* |
| **tupla** `(response, status)` | Status code personalizzato |
| **tupla** `(response, headers)` | Header aggiuntivi |
| **tupla** `(response, status, headers)` | Status + header aggiuntivi |

**Esempi:**

```python
# Stringa con status code 400
@app.route('/error')
def error():
    return '<h1>Bad Request</h1>', 400

# Stringa con status + headers personalizzati
@app.route('/errorh')
def errorh():
    headers = {'New-Header': 'FlaskApplication'}
    return '<h1>Bad Request</h1>', 400, headers

# Dizionario → jsonify automatico
@app.route("/me")
def me_api():
    user = {"username": "pippo", "age": 20}
    return user   # → JSON automatico

# Lista di dizionari → jsonify automatico
@app.route("/users")
def users_api():
    users = get_all_users()
    return users
```

### `make_response` — Costruzione Esplicita

```python
from flask import make_response
import time

@app.route('/')
def index():
    response = make_response('<h1>This document carries a cookie!</h1>')
    response.set_cookie('Last-access', str(time.time()))
    return response
```

---

## 🔀 Route Methods

Di default le route gestiscono solo **GET**. Per supportare altri metodi HTTP:

```python
# Metodo 1: argomento methods
@app.route('/print', methods=['GET', 'POST'])
def print_method():
    if request.method == 'POST':
        return '<h1>This is a POST</h1>'
    else:
        return '<h1>This is a GET</h1>'

# Metodo 2: shortcut decorator (consigliato per chiarezza)
@app.post('/print')
def print_post():
    return '<h1>This is a POST</h1>'

@app.get('/print')
def print_get():
    return '<h1>This is a GET</h1>'
```

Test da terminale:
```bash
curl http://127.0.0.1:5000/print          # GET
curl -X POST http://127.0.0.1:5000/print  # POST
```

---

## 🖥️ `curl` da Terminale

`curl` è un tool da riga di comando per generare richieste HTTP.

```bash
# Installazione
sudo apt-get install curl   # Linux
brew install curl           # macOS

# GET (default)
curl https://www.google.com/

# Specificare il metodo HTTP
curl -X DELETE https://example.com

# Recuperare solo gli Header (HEAD)
curl -I https://example.com

# POST con dati form
curl -d "name=curl" https://example.com

# POST con JSON
curl --json '{"name": "pippo", "surname": "pippozzo"}' https://example.com
```

| Parametro | Descrizione |
|---|---|
| `-X METHOD` | Specifica il metodo HTTP |
| `-I` | Solo header (richiesta HEAD) |
| `-d "data"` | Dati per POST (form-encoded) |
| `--json '{...}'` | Dati JSON per POST |

---

## 🐍 `requests` Library

**`requests`** è una libreria HTTP per Python per creare facilmente richieste HTTP/1.1.

**Installazione:**
```bash
pip install urllib3==1.26.6
pip install requests
```

### Metodi Principali

```python
import requests

r = requests.get('https://httpbin.org/get')
r = requests.post('https://httpbin.org/post', data={'key': 'value'})
r = requests.put('https://httpbin.org/put',   data={'key': 'value'})
r = requests.delete('https://httpbin.org/delete')
r = requests.head('https://httpbin.org/head')
```

### Firme delle Funzioni

| Funzione | Parametri chiave |
|---|---|
| `requests.get(url, params=None, **kwargs)` | `params`: query string |
| `requests.post(url, data=None, json=None, **kwargs)` | `data`: form-encoded; `json`: JSON body |
| `requests.put(url, data=None, **kwargs)` | `data`: body |
| `requests.delete(url, **kwargs)` | — |
| `requests.head(url, **kwargs)` | — |

### Query String

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get('https://httpbin.org/get', params=payload)
print(r.url)
# https://httpbin.org/get?key2=value2&key1=value1

# Lista come valore
payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
# → ?key1=value1&key2=value2&key2=value3
```

> [!tip]
> Se una chiave del dizionario ha valore `None`, il parametro **non** viene aggiunto alla query string.

### POST Payload

```python
# Form-encoded (data=)
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post('https://httpbin.org/post', data=payload)
# → "form": {"key1": "value1", "key2": "value2"}

# Multi-valore con lista di tuple
payload_tuples = [('key1', 'value1'), ('key1', 'value2')]
r = requests.post('https://httpbin.org/post', data=payload_tuples)

# JSON body (json=)
r = requests.post('https://api.github.com/some/endpoint', json={'some': 'data'})
```

### Response Object

| Attributo / Metodo | Descrizione |
|---|---|
| `r.text` | Contenuto della risposta in **unicode** |
| `r.content` | Contenuto della risposta in **bytes** |
| `r.encoding` | Encoding usato dal server (modificabile) |
| `r.status_code` | Status code HTTP (es. `200`, `404`) |
| `r.url` | URL finale (dopo eventuali redirect) |
| `r.headers` | Header della risposta (dizionario) |
| `r.headers['Content-Type']` | Header specifico |
| `r.json()` | Decodifica il body come JSON |
| `r.raise_for_status()` | Solleva `HTTPError` se status code è un errore |

> [!warning]
> `r.json()` decodifica solo il contenuto. Un JSON valido **non garantisce** che la richiesta abbia avuto successo: verificare sempre `r.status_code` o usare `raise_for_status()`.

```python
# Verifica errori
bad_r = requests.get('https://httpbin.org/status/404')
bad_r.status_code   # → 404
bad_r.raise_for_status()   # → HTTPError: 404 Client Error
```

---

## 🔗 Collegamenti

- [[12 - Python MOM STOMP]]
- [[11 - Python RPC gRPC]]
- [[10 - Sistemi Middleware]]
- [[09 - Python Networking]]
- [[Pattern Proxy-Skeleton]]

---

## 📚 Risorse Utili

- [Flask 3.0 docs](https://flask.palletsprojects.com/en/3.0.x/)
- [Flask API reference](https://flask.palletsprojects.com/en/3.0.x/api/)
- [requests library docs](https://requests.readthedocs.io/en/latest/)
- [curl manpage](https://curl.se/docs/manpage.html)
