## Service

I **Service** sono responsabili della comunicazione con entità esterne all'applicazione. Queste entità possono includere:
- **API di un back-end**: Per recuperare o inviare dati remoti.
- **File locali**: Per leggere o scrivere dati sul dispositivo dell'utente.
- **Socket**: Per stabilire connessioni in tempo reale con altri sistemi.
- **Chiamate al sistema operativo**: Per accedere a funzionalità native del dispositivo.  
  In generale, i Service si occupano di tutte le interazioni che richiedono una comunicazione con entità esterne o richieste specializzate al sistema.

---

## Linee Guida

### 1. **Progettazione dell’interfaccia**
- Identificare i dati o le operazioni di cui l’applicazione ha bisogno.  
  Ad esempio, un servizio potrebbe dover fornire una lista di oggetti per un negozio e un'operazione per salvarne di nuovi.

- Definire l’interfaccia del Service in modo **indipendente dall'implementazione**.  
  Questo significa progettare metodi che descrivano le operazioni senza vincolarsi a come queste sono eseguite (ad esempio, se i dati vengono letti da un file locale o da un’API).

### 2. **Vantaggi di un’interfaccia astratta**
- **Flessibilità**: Se in futuro cambia la sorgente dei dati (da un file locale a un'API, per esempio), la modifica può avvenire senza impattare altre parti dell’applicazione.
- **Manutenzione semplificata**: Un’interfaccia stabile rende più semplice aggiornare o sostituire l'implementazione di un Service.

### 3. **Separazione delle responsabilità**
- I Service devono occuparsi esclusivamente della comunicazione con le entità esterne e non gestire logiche interne dell’applicazione. Questo consente di mantenere chiaro e modulare il design.


## Esempi

### Service per ottenre dati da un api di prodotti di ecommerce

``` dart
//Service che esegue chiamate Api per prendere tutti gli
//oggetti venduti sul sito
class StoreItemService {
  Future<List<StoreItem>> getAllItems() async {
    final response =
        await http.get(Uri.parse('https://fakestoreapi.com/products'));

    if (response.statusCode == 200) {
      Iterable jsonList = jsonDecode(response.body);
      List<StoreItem> items = List.empty(growable: true);
      for (var jsonObject in jsonList) {
        items.add(StoreItem.fromJson(jsonObject));
      }
      return items;
    } else {
      throw Exception('Failed to load items');
    }
  }

  Future<List<StoreItem>> getAllItemsWithIds(Set<String> ids) async {
    List<StoreItem> allItems = await getAllItems();
    List<StoreItem> itemsWithId = List.empty(growable: true);
    for (StoreItem storeItem in allItems) {
      if (ids.contains(storeItem.id)) {
        itemsWithId.add(storeItem);
      }
    }
    return itemsWithId;
  }
}


``` 
### Spiegazione del **Service StoreItemService**

Il **StoreItemService** è un esempio concreto di Service che si occupa di comunicare con un'API esterna per recuperare i dati relativi agli oggetti venduti su un sito di e-commerce. Questo Service segue le linee guida presentate, garantendo modularità e flessibilità.

---

### Funzionalità principali

1. **Recupero di tutti gli oggetti**
    - Il metodo `getAllItems` invia una richiesta HTTP `GET` all'API remota (`https://fakestoreapi.com/products`).
    - Se la risposta ha uno status code di successo (`200`), i dati JSON restituiti vengono decodificati e trasformati in una lista di oggetti `StoreItem`.
    - In caso di errore (status code diverso da `200`), viene lanciata un'eccezione, mantenendo chiara la gestione degli errori.

2. **Recupero di oggetti filtrati per ID**
    - Il metodo `getAllItemsWithIds` consente di filtrare solo gli oggetti che corrispondono a un insieme di ID specificati.
    - Utilizza il metodo `getAllItems` per ottenere prima tutti gli oggetti e successivamente applica un filtro in base agli ID forniti.

---

### Applicazione delle linee guida

1. **Interfaccia chiara e indipendente dall'implementazione**
    - L'interfaccia del Service fornisce due metodi (`getAllItems` e `getAllItemsWithIds`) che definiscono operazioni specifiche senza rivelare dettagli sull'origine dei dati.
    - Questo approccio rende semplice modificare la sorgente dati in futuro (ad esempio, passando da un'API remota a un database locale) senza modificare il contratto dell'interfaccia.

2. **Separazione delle responsabilità**
    - Il Service si occupa esclusivamente di recuperare i dati e non implementa logiche applicative. La trasformazione dei dati JSON in oggetti `StoreItem` è un'operazione minimale necessaria per rappresentare i dati in modo utile all'applicazione.
3. **Modularità**
    - Grazie all'uso di metodi separati per le diverse operazioni, è possibile estendere facilmente le funzionalità del Service, aggiungendo ad esempio filtri più complessi o endpoint diversi.

---
