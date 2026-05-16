# 14 - NoSQL Database for Web Applications (MongoDB + PyMongo)

#python #advanced-computer-programming #NoSQL #MongoDB #PyMongo #database #DBMS #CRUD

**Corso:** Advanced Computer Programming  
**Docente:** Prof. Raffaele Della Corte  
**Riferimenti:**
- N. O'Higgins. *MongoDB & Python*, O'Reilly
- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [PyMongo Driver docs](https://www.mongodb.com/docs/languages/python/pymongo-driver/current/)
- [PyMongo readthedocs](https://pymongo.readthedocs.io/en/stable/)

---

## Concetti Chiave

- Database e Web Server: back-end, persistenza, riduzione carico RAM
- Database e DBMS: definizioni, esempi, features
- DB Relazionali vs Non-Relazionali: SQL vs NoSQL
- Vantaggi/Svantaggi dei DB relazionali: ACID-compliance, scalabilità, schema
- Tipologie NoSQL: Key-Value, Document Stores, Search Engines
- MongoDB: document-oriented, JSON-like, schema-free, componenti chiave
- MongoDB shell: `save`, `insertMany`, `find`, `remove`, `dropDatabase`
- PyMongo: `MongoClient`, accesso a DB e collection, formato dizionario
- CRUD in PyMongo: `insert_one/many`, `find_one/find`, `delete_one/many`, `update_one/many`
- Filtri con operatori: `$gt`, `$lt`, `$in`, `$nin`, `$or`, `$exists`, ecc.
- Modificatori di aggiornamento: `$set`, `$inc`, `$push`, `$pop`, `$rename`, ecc.
- Esempio completo con `get_database()` e `add_items()`

---

## Database e Web Server

I **Database** sono comunemente usati nella parte **back-end** delle web application:
- Forniscono un modo per salvare i dati in modo **persistente** in memoria
- Riducono il **carico sulla memoria centrale** del server
- Permettono di **recuperare i dati** anche in caso di crash del server
- Molte richieste HTTP richiedono query su un database (lettura o scrittura)

---

## Database e DBMS

- **Database**: collection di dati organizzati per essere facilmente acceduti, gestiti e aggiornati
- **DBMS** (Database Management System): software per la creazione, definizione e manipolazione di un database

Un DBMS fornisce: interfaccia per creare/modificare DB, protezione e sicurezza, consistenza dei dati in caso di utenti multipli.

**Esempi di DBMS:** MySQL, Oracle, SQL Server, IBM DB2, PostgreSQL, Amazon SimpleDB (cloud)

### Features di un DBMS

- Data stored into Tables
- Reduced Redundancy via Normalization
- Data Consistency
- Support Multiple user and Concurrent Access
- Query Language
- Security
- Support for transactions

---

## DB Relazionali vs Non-Relazionali

I database si dividono in due categorie principali: **Relazionali (SQL)** e **Non-Relazionali (NoSQL)**. Si possono usare singolarmente o in combinazione.

### Database Relazionali

Usano uno **schema** (template) per definire la struttura dei dati. Esempio principale: **MySQL**.

- Piu tabelle (Entita) in relazione tra loro tramite **chiavi**
- Ogni modifica a una tabella ha effetti sulle altre in modo predicibile

**Vantaggi:**

| Vantaggio | Dettaglio |
|---|---|
| **Schema ben definito** | Dati prevedibili e facilmente valutabili |
| **ACID-compliance** | Atomicity, Consistency, Isolation, Durability garantiti |
| **Ben strutturati** | Poco spazio all'occorrenza di errori |

**Svantaggi:**

| Svantaggio | Dettaglio |
|---|---|
| **Big data** | Schema rigido rende quasi impossibile l'uso in scenari big data |
| **Scalabilita** | Scalare su server multipli e costoso e difficoltoso |
| **Migrazione** | I vincoli dello schema ostacolano la migrazione tra RDBMS diversi |

### Database Non-Relazionali (NoSQL)

Piu **indulgenti** nella struttura. Usano **collections** di documenti invece di tabelle con righe e colonne.

**Tipologie:**

| Tipo | Descrizione | Esempi |
|---|---|---|
| **Key-Value** | Dati come coppie key-value | Amazon DynamoDB, Redis |
| **Document Stores** | Dizionario di tipi/valori annidati; dati in JSON documents | **MongoDB**, Couchbase |
| **Search Engines** | Accesso ai dati tramite text-based searches | Solr, Splunk, Elasticsearch |

**Pros e Cons:**

| | Dettaglio |
|---|---|
| Pro | Schema-free: gestione di grandi volumi; scalabilita orizzontale; dati distribuibili su piu nodi |
| Con | Non si puo fare affidamento sulla presenza di un campo; aggiornamento dati piu difficoltoso |

---

## MongoDB

**MongoDB** e un **document-oriented database** costruito su **JSON-like documents**, senza schema predefinito (campi creabili *on-the-fly*).

### Componenti Chiave

| Componente | Descrizione | Equivalente Relazionale |
|---|---|---|
| `_id` | Campo obbligatorio; valore unico (chiave primaria) | Primary Key |
| **Collection** | Raggruppamento di documents | Tabella |
| **Document** | Un record in una collection | Riga/Record |
| **Field** | Coppia name-value in un document | Colonna |
| **Database** | Contenitore per le collection | Database |
| **Cursor** | Puntatore al risultato di una query su cui iterare | Result set |

### Confronto Relazionale vs MongoDB

```
// Relazionale: 4 tabelle separate (users, professions, apps, cars)
// MongoDB: un unico document innestato
{
  first_name: "Mary", last_name: "Jones",
  location: { type: "Point", coordinates: [-73.9876, 40.7574] },
  profession: ["Developer", "Engineer"],
  apps: [ { name: "MyApp", version: 1.0.4 }, { name: "DocFinder", version: 2.5.7 } ],
  cars: [ { make: "Bentley", year: 1973 }, { make: "Rolls Royce", year: 1965 } ]
}
```

### Installazione e Avvio

```bash
sudo systemctl start mongod   # avvia il server MongoDB
mongo                          # avvia la mongo shell
```

### MongoDB Shell

```javascript
show dbs                        // mostra i database
show collections                // mostra le collection
db.cards.save(cards)            // inserisce array di oggetti
db.cards.insertMany(cards)      // alternativa a save
db.cards.find()                 // mostra tutti i documenti

// DELETE
db.collectionName.remove({QUERY});  // rimuove documenti che soddisfano la query
db.collectionName.remove({});       // rimuove TUTTI i documenti
use databaseName;
db.dropDatabase();                  // elimina l'intero database
```

---

## PyMongo

**PyMongo** è il **driver ufficiale** per l'utilizzo di MongoDB in Python.

```bash
pip install pymongo
```

### Connessione con MongoClient

```python
from pymongo import MongoClient

# Metodo 1: host + porta
client = MongoClient("localhost", 27017)

# Metodo 2: connection string
client = MongoClient("mongodb://localhost:27017/")

# Accedere a un database e a una collection
db         = client['test-database']
collection = db['test_collection']
```

> Database e collection vengono creati automaticamente al primo inserimento se non esistono gia.

### Formato Dati: Dizionari Python

```python
post = {
    "author": "Mike",
    "text":   "My first blog post!",
    "tags":   ["mongodb", "python", "pymongo"]
}
```

---

## CRUD in PyMongo

### Create

```python
collection.insert_one(post)           # un solo documento (_id aggiunto automaticamente se assente)

posts = [
    {"author": "Mike",  "text": "Another post!", "tags": ["bulk", "insert"]},
    {"author": "Eliot", "title": "MongoDB is fun", "text": "and easy too!"},
]
collection.insert_many(posts)         # documenti multipli
```

### Read

```python
result  = collection.find_one({"author": "Eliot"})    # primo documento che fa match (o None)
results = collection.find({"author": "Eliot"})         # cursor su tutti i matching

for post in results:
    print("author:", post['author'])

all_docs = collection.find({})        # tutti i documenti
```

### Delete

```python
collection.delete_one({"author": "Eliot"})    # cancella il primo che fa match
collection.delete_many({"author": "Eliot"})   # cancella tutti quelli che fanno match
```

### Update

```python
# Struttura: update_one(filtro, modificatore)
collection.update_one(
    {"author": "Eliot"},
    {"$set": {"author": "eliot"}}
)

# Tutti i matching
collection.update_many(
    {"author": "Eliot"},
    {"$set": {"author": "eliot"}}
)
```

---

## Filtri

```python
res = collection.find({"items": {"$in": ["item-A", "item-B"]}})
```

| Operatore | Significato | SQL Equivalent |
|---|---|---|
| `$gt` | Greater Than | `>` |
| `$lt` | Less Than | `<` |
| `$gte` | Greater Than or Equal | `>=` |
| `$lte` | Less Than or Equal | `<=` |
| `$ne` | Not Equals | `!=` |
| `$in` | In (lista) | `IN` |
| `$nin` | Not In | `NOT IN` |
| `$all` | Array Must Contain All | N/A |
| `$exists` | Property Must Exist | N/A |
| `$mod` | Modulo X Equals Y | `MOD()` |
| `$or` | Or | `OR` |
| `$nor` | Nor | N/A |
| `$size` | Array Must Be Of Size | N/A |

---

## Modificatori di Aggiornamento

| Modificatore | Significato | Esempio |
|---|---|---|
| `$inc` | Atomic Increment | `{"$inc": {"score": 1}}` |
| `$set` | Set Property Value | `{"$set": {"username": "niall"}}` |
| `$unset` | Unset (delete) Property | `{"$unset": {"username": 1}}` |
| `$push` | Atomic Array Append (atom) | `{"$push": {"emails": "foo@example.com"}}` |
| `$pushAll` | Atomic Array Append (list) | `{"$pushAll": {"emails": ["a@ex.com","b@ex.com"]}}` |
| `$addToSet` | Append-If-Not-Present | `{"$addToSet": {"emails": "foo@example.com"}}` |
| `$pop` | Atomic Array Tail Remove | `{"$pop": {"emails": 1}}` |
| `$pull` | Conditional Array Item Removal | `{"$pull": {"emails": "foo@example.com"}}` |
| `$pullAll` | Array Multi Item Removal | `{"$pullAll": {"emails": ["a@ex.com","b@ex.com"]}}` |
| `$rename` | Atomic Property Rename | `{"$rename": {"emails": "old_emails"}}` |

---

## Esempio Completo

```python
from pymongo import MongoClient

def get_database():
    CONNECTION_STRING = "mongodb://127.0.0.1:27017"
    client = MongoClient(CONNECTION_STRING)
    return client['test']

def add_items(collection):
    item_1 = {
        "_id":          "U1IT00001",
        "item_name":    "Blender",
        "max_discount": "10%",
        "batch_number": "RR450020FRG",
        "price":        340,
        "category":     "kitchen appliance"
    }
    item_2 = {
        "_id":              "U1IT00002",
        "item_name":        "Egg",
        "category":         "food",
        "quantity":         12,
        "price":            36,
        "item_description": "brown country eggs"
    }
    collection.insert_many([item_1, item_2])

if __name__ == "__main__":
    dbname          = get_database()
    collection_name = dbname["user_1_items"]
    add_items(collection_name)
```

I due item hanno strutture diverse: valido in MongoDB grazie alla natura schema-free.

---

## Risorse Utili

- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [PyMongo Driver](https://www.mongodb.com/docs/languages/python/pymongo-driver/current/)
- [PyMongo API Reference](https://pymongo.readthedocs.io/en/stable/)
- [Install MongoDB on Ubuntu](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)
