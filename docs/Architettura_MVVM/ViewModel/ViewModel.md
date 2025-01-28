## Il ruolo del ViewModel

Il **ViewModel** si occupa di collegare la **View** ai **Model** e ai **Service**, e ha diverse responsabilità fondamentali:

### Responsabilità principali del ViewModel
1. **Fornire dati alla View**:  
   Espone i dati necessari alla **View** attraverso getter, garantendo che la View possa accedere solo a una rappresentazione filtrata e rilevante dei dati del **Model**.

2. **Gestire input della View**:  
   Espone metodi richiamabili dalla **View**, che permettono di gestire input dell’utente e di modificare lo stato dell’applicazione.

3. **Notificare la View**:  
   Espone eventi che permettono alla **View** di sapere quando aggiornarsi, in base ai cambiamenti del **Model**.

### Collegamento con Model e Service
Per adempiere alle sue funzioni, il **ViewModel**:
- **Ottiene i dati dal Model**:  
  Recupera le informazioni di base, le filtra o le trasforma per adattarle alle esigenze della View.

- **Modifica il Model**:  
  Quando riceve input dalla View, può aggiornare direttamente il Model o interagire con un **Service**.

- **Gestisce eventi del Model**:  
  Si registra agli eventi di modifica del Model, reagendo a essi per aggiornare la View di conseguenza.

- **Utilizza i Service**:  
  Interagisce con i Service per ottenere dati esterni o eseguire operazioni che influenzano il Model.

### Stato del ViewModel
Il **ViewModel** non deve avere uno stato interno persistente. Invece:
- Lo stato del ViewModel deve essere derivato direttamente dal **Model**.
- Questo approccio assicura che il ViewModel sia sempre sincronizzato con il Model e mantenga un'architettura pulita e prevedibile.

## Linee Guida

### Progettazione dell’interfaccia di un ViewModel
Per progettare efficacemente un ViewModel, bisogna chiedersi: *Di cosa ha bisogno la View?*

1. **che input mi può fornire la View?**:
    - per ogni input diverso che si può ricevere dalla View creare un metodo per gestire quell'input

2. **di che dati ha bisogno la View?**:
    - I dati devono essere una versione filtrata o trasformata di quelli presenti nel Model.

3. **che eventi interessano alla View?**:
    - creare gli eventi che notificano alla View quando deve aggiornarsi.

### Progettazione dell’implementazione del ViewModel
Per implementare il ViewModel, bisogna considerare quali Model e Service sono necessari per soddisfare i requisiti della View:

1. **da dove derivano i dati forniti alla View?**:  
   I dati esposti dal ViewModel alla View derivano dai dati del Model, ma vengono filtrati o trasformati per essere più utili alla View.

2. **quando lancio gli eventi alla View?**:  
   Gli eventi esposti dal ViewModel alla View corrispondono spesso a eventi di modifica del Model, filtrati o raggruppati.

3. **cosa devo fare con l'input della View?**:  
   Quando il ViewModel riceve input dalla View, tende a:
    - Modificare direttamente i dati del Model.
    - Utilizzare i Service per ottenere dati esterni o eseguire operazioni complesse.

## Esempi:

### Esempio ViewModel per gli oggetti filtrati in vendita di un ecommerce

```dart
//ViewModel per permettere di caricare la lista degli elementi e filtrarli in base ai filtri attivi
//Notifica ogni volta che cambia la lista oppure uno dei filtri
class ShoppingItemFilteredViewModel extends ViewModel{

  //Events
  Event onItemsChanged = Event();

  //Models
  ShoppingItemsModel shoppingItemsModel;
  ItemsFilterModel itemsFilterModel;

  //Services
  StoreItemService storeItemsService;


  ShoppingItemFilteredViewModel(this.shoppingItemsModel, this.storeItemsService,this.itemsFilterModel){
    shoppingItemsModel.items.addListener(onItemsChanged.notifyListeners);
    itemsFilterModel.maxPrice.addListener(onItemsChanged.notifyListeners);
    itemsFilterModel.prefix.addListener(onItemsChanged.notifyListeners);
      _Init();
  }

  Future<void> _Init()
  async {
    shoppingItemsModel.items.value = await storeItemsService.getAllItems();

  }
  
  @override
  void dispose() {
    shoppingItemsModel.items.removeListener(onItemsChanged.notifyListeners);
    itemsFilterModel.maxPrice.removeListener(onItemsChanged.notifyListeners);
    itemsFilterModel.prefix.removeListener(onItemsChanged.notifyListeners);
  }



  List<StoreItem> get  shoppingItems  {
    List<StoreItem> filterResult = List.empty(growable: true);
    for(StoreItem item in shoppingItemsModel.items.value)
      {
        if(!item.name.startsWith(itemsFilterModel.prefix.value)) continue;
        if(!(double.parse(item.price) <= itemsFilterModel.maxPrice.value)) continue;
        filterResult.add(item);
      }
    return filterResult;
  }


}


```
### Spiegazione del ViewModel

Questo ViewModel, denominato `ShoppingItemFilteredViewModel`, è progettato per fornire una lista filtrata degli oggetti in vendita in un e-commerce, in base a criteri specifici come prezzo massimo e prefisso del nome. Inoltre, notifica la View ogni volta che i dati filtrati cambiano.

#### Applicazione delle linee guida

1. **Progettazione dell’interfaccia**
    - **Input dalla View:**  
      La View non fornisce input diretto in questo caso, ma il ViewModel risponde indirettamente a modifiche nei modelli sottostanti, come i filtri. Ogni filtro attivo è rappresentato nel `ItemsFilterModel`, che invia eventi quando i valori cambiano.
    - **Dati forniti alla View:**  
      Il getter `List<StoreItem> get shoppingItems` fornisce una versione filtrata degli oggetti presenti nel `ShoppingItemsModel`. Questo getter è un esempio di trasformazione dei dati dal Model per soddisfare le esigenze della View.
    - **Eventi per la View:**  
      L'evento `Event onItemsChanged` notifica la View ogni volta che la lista degli oggetti o i filtri cambiano, permettendo alla View di aggiornarsi di conseguenza.

2. **Progettazione dell’implementazione**
    - **Derivazione dei dati:**  
      La lista filtrata è calcolata partendo dal `ShoppingItemsModel` e applicando i criteri definiti nel `ItemsFilterModel`. Questo dimostra come il ViewModel filtra e adatta i dati dei modelli per renderli più utili alla View.
    - **Gestione degli eventi:**  
      Il costruttore del ViewModel si registra agli eventi di modifica di entrambi i modelli (`shoppingItemsModel.items` e `itemsFilterModel`) per garantire che qualsiasi modifica nei dati o nei filtri attivi si rifletta automaticamente nella View.
    - **Risposta agli input:**  
      Sebbene non vi sia un metodo specifico per gestire input diretti della View in questo esempio, il ViewModel utilizza il `StoreItemService` per popolare inizialmente i dati, dimostrando come i Service possono essere utilizzati per interazioni più complesse.

3. **Organizzazione e implementazione**
    - L'implementazione del getter e dell'evento è coerente con l'obiettivo del ViewModel di fungere da intermediario. Filtra i dati, si registra agli eventi dei Model e utilizza un Service per ottenere dati iniziali, senza avere uno stato interno.
    - La separazione delle responsabilità è chiara: i modelli gestiscono i dati grezzi, mentre il ViewModel si occupa di adattarli e notificare la View quando necessario.

### Esempio ViewModel per la card di un singolo oggetto in vendita

``` dart
//View model da utilizzare a supporto della Card di un singolo elemento
//notifica quando l'elemento di Id itemId è stato aggiunto/rimosso dal carrello
class SingleItemShoppingCartViewModel extends ViewModel{
  String itemId;
  ShoppingCartModel shoppingCartModel;

  //Event
  //Evento per notificare quando l'elemento interessato viene
  // aggiunto o rimosso dal carrello
  Event onCartStateChanged = Event();

  SingleItemShoppingCartViewModel(this.itemId, this.shoppingCartModel)
  {
    shoppingCartModel.items.addObservers(onCartChanged);
  }


  @override
  void dispose() {
    shoppingCartModel.items.removeObserver(onCartChanged);
  }

  //uso questo metodo come callback al SetNotifier contenuto nel modello
  //quando il carrello cambia controllo se il cambiamento riguardava l'id a cui
  //sono interessato e in quel caso chiamo l'evento
  void onCartChanged(String id,SetOperation operation)
  {

    if(id == itemId)
    {
      onCartStateChanged.notifyListeners();
    }
  }

}

```

### Spiegazione del ViewModel

Il `SingleItemShoppingCartViewModel` è progettato per supportare la gestione della Card di un singolo elemento in vendita, notificando la View quando lo stato dell'elemento (aggiunto o rimosso dal carrello) cambia. Si concentra su un singolo elemento identificato da un `itemId`.

#### Applicazione delle linee guida

1. **Progettazione dell’interfaccia**
   - **Input dalla View:**  
     La View non fornisce input diretto, ma il ViewModel è costruito con un parametro statico `itemId`, che rappresenta l'elemento specifico che deve monitorare. Questo approccio garantisce che il ViewModel sia specifico per un solo elemento.
   - **Dati forniti alla View:**  
     Questo ViewModel non espone direttamente dati complessi, ma notifica la View di eventi specifici tramite `Event onCartStateChanged`. In questo modo, la View può decidere come aggiornarsi senza accedere ai dati interni del carrello.
   - **Eventi per la View:**  
     L'evento `onCartStateChanged` viene lanciato ogni volta che il carrello cambia stato per l'elemento monitorato (`itemId`), permettendo alla View di aggiornare il pulsante o lo stato visivo del singolo prodotto.

2. **Progettazione dell’implementazione**
   - **Derivazione dei dati:**  
     Il ViewModel monitora il carrello (`ShoppingCartModel.items`) per verificare quando il cambiamento riguarda l'`itemId` associato. Questo è un esempio di filtraggio di un modello più complesso per soddisfare un caso d'uso specifico della View.
   - **Gestione degli eventi:**  
     Il costruttore si registra all'osservatore del carrello (`shoppingCartModel.items`) e si assicura che il ViewModel lanci l'evento `onCartStateChanged` solo quando l'elemento monitorato viene modificato. Questa logica è implementata nella callback `onCartChanged`, che controlla l'`id` dell'elemento interessato.
   - **Risposta agli input:**  
     Sebbene non ci siano metodi per gestire input diretti dalla View, la callback `onCartChanged` rappresenta una risposta indiretta agli eventi generati dai cambiamenti nel modello.



### Esempio ViewModel per aggiungere e rimuovere elementi dal carrello:

``` dart
//ViewModel il cui scopo è quello di aggiungere o rimuovere elementi dal carrello
//e verificare se un dato elemento è parte del carrello
class ShoppingCartViewModel extends ViewModel {
  ShoppingCartModel shoppingCartModel;

  ShoppingCartViewModel(this.shoppingCartModel);
  @override
  void dispose() {

  }

  void addItemToCart(String id) {
    shoppingCartModel.items.add(id);
  }

  void removeItemToCart(String id) {
    shoppingCartModel.items.remove(id);
  }

  bool IsItemInCart(String id) {
    return shoppingCartModel.items.set.contains(id);
  }
}

```

### Spiegazione del ViewModel

Il `ShoppingCartViewModel` è stato progettato per gestire le operazioni principali relative al carrello, fornendo metodi chiari per aggiungere, rimuovere e verificare la presenza di elementi.

#### Applicazione delle linee guida

1. **Progettazione dell’interfaccia**
   - **Quali input provengono dalla View?**  
     La View può richiedere 2 tipi di operazioni:
      - Aggiungere un elemento al carrello: gestito dal metodo `addItemToCart`.
      - Rimuovere un elemento dal carrello: gestito dal metodo `removeItemToCart`.
        Questi metodi permettono alla View di interagire con il carrello in maniera semplice e diretta.
   - **Quali dati sono forniti alla View?**  
     La View riceve informazioni sullo stato del carrello attraverso il metodo `IsItemInCart`, che restituisce un valore booleano. Questo dato è derivato direttamente dallo stato del `ShoppingCartModel`, ma è trasformato in una forma più facilmente utilizzabile dalla View.
   - **Quali eventi interessano alla View?**  
     In questo caso, il ViewModel non espone eventi diretti. Tuttavia, la gestione dello stato del carrello è strettamente legata alle modifiche del `ShoppingCartModel`.

2. **Progettazione dell’implementazione**
   - **Da dove derivano i dati forniti alla View?**  
     I dati esposti dal ViewModel (ad esempio, tramite il metodo `IsItemInCart`) sono derivati dal `ShoppingCartModel`, in particolare dal set degli `items`.
   - **Quando vengono lanciati eventi alla View?**  
     Anche se questo ViewModel non espone eventi, il suo scopo è principalmente quello di fungere da interfaccia per le operazioni sul modello. Eventuali notifiche alla View possono essere delegate ad altri componenti o gestite separatamente.
   - **Cosa fare con l’input della View?**  
     Gli input della View (ad esempio, richieste di aggiunta o rimozione di elementi) sono gestiti direttamente modificando il `ShoppingCartModel`. I metodi `addItemToCart` e `removeItemToCart` operano sul modello, aggiornandone lo stato.

### Gestione delle risorse nei ViewModel

Tutti i ViewModel espongono un metodo `dispose` per gestire la liberazione delle risorse allocate durante il loro ciclo di vita. Questo è particolarmente importante per evitare **memory leak**, che possono verificarsi quando i ViewModel restano in memoria a causa di listener o callback non deregistrati.
