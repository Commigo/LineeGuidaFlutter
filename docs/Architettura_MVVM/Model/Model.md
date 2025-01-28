## Il Model: Gestione dei Dati Attivi dell'Applicazione

Il **Model** è responsabile di gestire i dati "attivi" dell'applicazione, ovvero lo stato corrente dell'applicazione in
esecuzione.  
Questi dati si differenziano dai dati "passivi", che sono persistenti e rappresentano lo stato generale del sistema,
memorizzati ad esempio in un database. I dati attivi, invece, riflettono lo stato dinamico e temporaneo di una specifica
istanza dell'applicazione.
(Queste definizioni me le sono appena inventate, mi sento un professore di matematica)

### Notifiche e Reattività nei Model

Il Model non è solo un contenitore di dati, ma ha la caratteristica fondamentale di poter notificare dei **listener**
ogni volta che i dati al suo interno cambiano. Questo è reso possibile avvolgendo i dati in strutture che emettono
eventi quando avvengono modifiche.

- **Strumenti predefiniti**: Alcuni strumenti sono già forniti da Flutter, come ad esempio `ValueNotifier`, che permette
  di notificare i listener ogni volta che il valore contenuto cambia.
- **Notificatori personalizzati**: In altri casi, è necessario creare notificatori personalizzati per soddisfare
  esigenze specifiche. Ad esempio, un `SetNotifier` potrebbe notificare quando un elemento viene aggiunto o rimosso da
  un insieme.

### Linee Guida per Progettare un Model

Per progettare un Model efficace, è utile seguire questi passaggi:

1. **Identificare i dati necessari**: Chiedersi quali sono i dati che l'applicazione deve utilizzare e rappresentare
   come stato attivo.
2. **Creare la struttura dati**: Definire una classe contenente questi dati.
3. **Rendere i dati reattivi**: Avvolgere i dati in strutture che consentano di notificare i listener quando i dati
   cambiano, utilizzando strumenti predefiniti o personalizzati.

Un Model ben progettato garantisce che i cambiamenti dello stato siano facilmente tracciabili e reattivi, semplificando
la comunicazione tra il Model e i livelli superiori dell'architettura, come il ViewModel.

## Esempi

### Model per contenere una lista di oggetti di un Ecommerce

``` dart
//model per racchiudere un insieme di Elementi dello store
//invia una notifica quando l'insieme di elementi cambia
class ShoppingItemsModel{
  ValueNotifier<List<StoreItem>> items = ValueNotifier(List.empty());
}

``` 

#### Spiegazione del Model per la Lista di Oggetti di un Ecommerce:

Questo **Model** è progettato per gestire una lista di oggetti disponibili in un ecommerce. Rappresenta uno stato attivo
dell'applicazione, ovvero i dati che descrivono gli oggetti attualmente caricati o disponibili.

#### Applicazione delle Linee Guida:

1. **Identificazione dei dati necessari**:  
   La lista di oggetti dello store è un elemento centrale dello stato dell'applicazione, poiché rappresenta i prodotti
   da visualizzare all'utente.

2. **Creazione della struttura dati**:  
   È stato creato un Model contenente una lista, che funge da contenitore per i prodotti dello store.

3. **Rendere i dati reattivi**:  
   La lista è stata avvolta in una struttura reattiva (`ValueNotifier`) che consente di notificare automaticamente i
   listener ogni volta che i dati cambiano. Questo garantisce che eventuali modifiche alla lista degli oggetti (come
   aggiunte, rimozioni o aggiornamenti) possano essere tracciate e comunicate in tempo reale agli altri livelli
   dell'applicazione, come i ViewModel o le View.

### Model per rappresentare gli elementi nel carrello

``` dart
//model per racchiudere un insieme di Elementi dello store
//invia una notifica quando l'insieme di elementi cambia
class ShoppingItemsModel{
  ValueNotifier<List<StoreItem>> items = ValueNotifier(List.empty());
}

``` 

#### Spiegazione del Model per gli Elementi nel Carrello

Questo **Model** gestisce lo stato degli elementi inseriti nel carrello di un ecommerce. È progettato per tracciare
dinamicamente le modifiche al carrello e notificare tali cambiamenti agli osservatori.

#### Applicazione delle Linee Guida

1. **Identificazione dei dati necessari**:  
   Il carrello è rappresentato come un insieme di identificatori univoci degli oggetti (ad esempio, ID dei prodotti).
   Questo riflette lo stato attuale del carrello e consente di identificare facilmente quali oggetti sono stati aggiunti
   o rimossi.

2. **Creazione della struttura dati**:  
   Per gestire il carrello, è stato utilizzato un insieme (`Set`) per evitare duplicati, dato che un prodotto può essere
   presente una sola volta nel carrello.

3. **Rendere i dati reattivi**:  
   L'insieme è stato avvolto in una classe reattiva personalizzata (`SetNotifier`), che invia notifiche ogni volta che
   un elemento viene aggiunto o rimosso. Questa classe consente di gestire eventi granulari legati alle modifiche del
   carrello, come l'aggiunta o la rimozione di un prodotto.

### Model per rappresentare i filtri attivi

``` dart
//Modello per rappresentare i filtri attivi per gli oggetti del negozio
class ItemsFilterModel {
  ValueNotifier<String> prefix = ValueNotifier("");
  ValueNotifier<double> maxPrice = ValueNotifier(double.infinity);
}

``` 

#### Spiegazione del Model per i Filtri degli Oggetti del Negozio

Questo **Model** rappresenta lo stato corrente dei filtri applicati agli oggetti di un negozio online. È progettato per rendere i filtri dinamici e reattivi, notificando eventuali cambiamenti ai componenti interessati.

#### Applicazione delle Linee Guida

1. **Identificazione dei dati necessari**:  
   Il Model include due filtri principali:
   - **`prefix`**: Una stringa che rappresenta il prefisso del nome degli oggetti da filtrare.
   - **`maxPrice`**: Un valore numerico che indica il prezzo massimo consentito per gli oggetti filtrati.

2. **Creazione della struttura dati**:  
   Entrambi i dati sono rappresentati da tipi semplici (`String` e `double`) per riflettere in modo chiaro i requisiti dei filtri.

3. **Rendere i dati reattivi**:  
   I dati sono avvolti in oggetti `ValueNotifier`, che consentono di notificare i listener ogni volta che il valore cambia. Questo approccio rende il Model reattivo e permette agli osservatori di essere aggiornati in tempo reale quando un filtro viene modificato.

