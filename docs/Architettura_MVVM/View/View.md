# View

La **View** è responsabile della visualizzazione dell'interfaccia utente e della gestione degli input. Si occupa di
mostrare le informazioni, catturare gli input dell'utente, aggiornarsi in base allo stato dell'applicazione e istanziare
le risorse necessarie.

In questo capitolo spiegheremo:

1. **Come progettare e utilizzare una View.**
2. **Esempi pratici per implementare una View.**

---

## Come progettare una View

Per creare una **View** efficace in un'architettura MVVM, è fondamentale seguire questi passaggi:

### 1. Identificare gli elementi grafici

Prima di tutto, analizzare cosa deve essere visualizzato nell'interfaccia. Ogni componente grafico (ad esempio, testi,
bottoni, liste) deve avere una posizione e una funzione chiara nella **View**.

**Esempio:** In una pagina e-commerce, gli elementi grafici includono:

- Una lista di prodotti.
- Pulsanti per aggiungere o rimuovere prodotti dal carrello.

### 2. Classificare i dati in statici e dinamici

- **Dati statici:** Non cambiano durante la vita del componente e possono essere passati tramite il costruttore.  
  **Esempio:** Il nome e il prezzo di un prodotto.
- **Dati dinamici:** Possono cambiare in base all'interazione dell'utente o allo stato dell'applicazione. Vanno forniti
  dal **ViewModel** e gestiti tramite costrutti reattivi come `ListenableBuilder`.

**Esempio:** Lo stato di un prodotto (se è o meno nel carrello) è dinamico e dipende dal **ViewModel**.

### 3. Definire le condizioni di aggiornamento e utilizzare i costrutti reattivi

Per creare una **View** efficiente, è essenziale stabilire prima le condizioni che richiedono l’aggiornamento
dell’interfaccia e poi implementare costrutti reattivi per gestire tali aggiornamenti.

**Definire le condizioni di aggiornamento:**

- Analizzare ogni parte della **View** e chiedersi: *In quali circostanze questa porzione deve essere aggiornata?*
- Identificare gli eventi specifici che influenzano lo stato della **View**. Questi eventi sono definiti nel **
  ViewModel** e segnalano quando una parte dell'interfaccia necessita di un ricaricamento.
- Separare chiaramente le sezioni che devono essere dinamiche (reattive) da quelle statiche, per evitare aggiornamenti
  inutili.

**Utilizzare i costrutti reattivi:**

- Una volta definite le condizioni di aggiornamento, utilizzare costrutti reattivi come `ListenableBuilder` per
  avvolgere solo le porzioni della **View** che dipendono da dati dinamici.
- Limitare l’ambito di ciascun costrutto reattivo per ridurre al minimo le ricostruzioni e migliorare le prestazioni.
- Collegare i costrutti reattivi agli eventi del **ViewModel** identificati in precedenza, in modo che l’interfaccia si
  aggiorni automaticamente quando lo stato dell’applicazione cambia.

Seguendo questi passaggi, è possibile creare una **View** reattiva e ben ottimizzata, capace di aggiornarsi in maniera
precisa e solo quando necessario.

### 4. Gestire gli input dell'utente

Chiedersi: *In che modo l'utente può interagire con la **View**?*  
Rispondendo a questa domanda, si identificano gli eventi che portano a richiamare funzionalità sui ViewModel per
modificare lo stato dell'applicazione.

---

## Esempi pratici di View

### 1. Lista di prodotti in una pagina di e-commerce

Immaginiamo una pagina che mostra una lista di prodotti disponibili per l'acquisto. La **View** si occupa di:

- Visualizzare i prodotti uno sotto l'altro.
- Aggiornare la lista in caso di cambiamenti, ad esempio quando vengono applicati dei filtri.

#### Codice:

```dart
return ListenableBuilder(
  builder: (context, _) {
    return Column(
      children: [
        for (StoreItem item in shoppingItemReadViewModel.shoppingItems)
          ShoppingItemCard(storeItem: item),
      ],
    );
  },
  listenable: shoppingItemReadViewModel.onItemsChanged,
);

```

### Ragionamento progettuale

Seguendo le linee guida, ecco come è stata progettata la **View** per la lista degli oggetti in una pagina e-commerce:

1. **Identificare gli elementi grafici**

- **Obiettivo:** Mostrare una lista di prodotti.
- Ogni prodotto deve essere visualizzato come un elemento della lista con le relative informazioni tramite il componente
  ShoppingItemCard.

2. **Classificare i dati in statici e dinamici**

- **Dati statici:** Nessuno specifico in questa parte, poiché la lista di prodotti è dinamica.
- **Dati dinamici:** La lista degli oggetti disponibili, fornita dal **ViewModel** tramite il campo `shoppingItems`. La
  lista può cambiare, ad esempio, con l'applicazione di un filtro.


3. **Stabilire le condizioni di aggiornamento e utilizzare i costrutti reattivi**

- La lista deve aggiornarsi quando cambia l'elenco dei prodotti, ad esempio in seguito all'aggiunta di filtri.
- È stato utilizzato un `ListenableBuilder` intorno alla lista degli oggetti per garantire che si aggiorni solo quando
  l'evento `shoppingItemReadViewModel.onItemsChanged` viene attivato.

4. **Gestire gli input dell'utente**

- Non direttamente presente in questa porzione di **View**, ma i componenti della lista (come i pulsanti all'interno di
  ciascun prodotto) gestiscono gli input.

Risultato: La lista di prodotti viene visualizzata come una colonna, e si aggiorna quando viene chiamato l'evento
shoppingItemReadViewModel.onItemsChanged
(successivamente vedremo come questo evento viene chiamto quando vengono modificati i filtri).
![Pagina di oggetti](/Images/ItemsColumn.png)

## Componente di un singolo prodotto

Ogni prodotto viene rappresentato da un componente grafico che mostra:

- **Nome e prezzo del prodotto** (dati statici).
- **Pulsante dinamico** per aggiungere o rimuovere il prodotto dal carrello, il pulsante è dinamico poichè cambia a
  seconda se il prodotto è già presente oppure no nel carrello.

### Codice

```dart
return Card(
      child: Column(
        children: [
          Text(storeItem.name),
          Text(storeItem.price),
          //se l'oggetto viene tolto o inserito nel carrello allora il bottone e solo il bottone
          //si aggiorna, il resto rimane invariato
          ListenableBuilder(
            builder: (context, _) {
              bool isInCart = shoppingCart.IsItemInCart(storeItem.id);
              String text =
                  isInCart ? "Rimuovi dal carrello" : "Aggiungi al carrello";
              void Function() onPressed = isInCart
                  ? () => shoppingCart.removeItemToCart(storeItem.id)
                  : () => shoppingCart.addItemToCart(storeItem.id);
              return ElevatedButton(onPressed: onPressed, child: Text(text));
            },
            listenable: singleItemViewModel.onCartStateChanged,
          ),
        ],
      ),
    );
```

## Ragionamento Progettuale: Componente di un Singolo Prodotto

1. **Identificare gli elementi grafici**

- **Obiettivo:** Visualizzare le informazioni principali di un prodotto e consentire all'utente di interagire con esso
  tramite un pulsante.
- Elementi:
    - Nome e prezzo del prodotto.
    - Un pulsante per aggiungere o rimuovere il prodotto dal carrello, il cui stato cambia dinamicamente in base al
      contenuto del carrello.

2. **Classificare i dati in statici e dinamici**

- **Dati statici:** Nome e prezzo del prodotto, che non cambiano durante la vita del componente. Questi vengono passati
  come parametri tramite il costruttore della classe.
- **Dati dinamici:** Lo stato del pulsante (testo e comportamento) è dinamico e dipende dal contenuto del carrello,
  gestito tramite il **ViewModel**.

3. **Stabilire le condizioni di aggiornamento e uUtilizzare i costrutti reattivi**

- Il pulsante deve aggiornarsi quando il suo stato nel carrello cambia ovvero quando viene rimosso o aggiunto al
  carrello. Solo questa porzione della **View** è progettata per ricostruirsi in risposta a tale evento.
- Per aggiornare solo il pulsante senza ricostruire l’intero componente, si utilizza un costrutto reattivo
  ListenableBuilder. Questo permette di monitorare i cambiamenti dello stato del carrello attraverso eventi definiti
  nel **ViewModel**.


4. **Gestire gli input dell'utente**

- L'interazione dell'utente avviene tramite il pulsante:
- Cliccando sul pulsante, si attivano funzioni definite nel **ViewModel** per aggiungere o rimuovere il prodotto dal
  carrello.

  
---

