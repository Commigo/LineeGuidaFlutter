### Istanziazione delle Classi e Gestione delle Dipendenze

Oltre a definire le classi e le loro responsabilità, è fondamentale stabilire dove queste classi debbano essere istanziate e come possano ottenere le dipendenze necessarie. Un esempio tipico è un ViewModel che ha bisogno di accedere ai Model e ai Service da cui dipende.

In Flutter, questo processo può essere semplificato tramite il package **Provider**, che consente di inserire risorse nell'albero dei widget. I Provider permettono di accedere a queste risorse attraverso il `BuildContext`, rendendole disponibili ai componenti discendenti.

---

### Utilizzo di un Provider

``` dart
Provider(create: (BuildContext context){
                  return ItemFilterViewModel();
              },
              dispose: (_,value){
                value.dispose();
              },
              child: SomeWidget()
              )

``` 

---

### Spiegazione del Funzionamento

- Quando un Provider viene creato all'interno dell'albero dei widget viene chiamato il suo metodo create, che inserisce la risorsa creata all'interno del context.
- Quando un Provider viene tolto dall'interno dell'albero dei widget viene chiamato il suo metodo dispose, che si può occupare di liberare le risorse allocate da quella creata dal provider.
- Tutti i widget figli del provider (nel caso dell'esempio di sopra SomeWidget e tutti i suoi figli) possono usare context.read() per ottenere la risorsa

in poche parole il provider è un semplice widget che specifica come creare una risorsa (Model,ViewModel o Service) e la rende disponibile a tutti i suoi figli.

---

### Posizionamento dei Provider

I Provider forniscono risorse solo ai componenti che sono loro discendenti nell'albero dei widget di Flutter. È quindi essenziale scegliere attentamente dove posizionare i Provider nell'albero:

per esempio:
1. **Risorse globali**  
   Se una risorsa deve essere accessibile da tutta l'applicazione (es. un servizio condiviso o uno stato globale), il Provider deve essere posizionato ai livelli più alti dell'albero, ad esempio in corrispondenza del widget principale dell'applicazione.

2. **Risorse specifiche per una pagina o un componente**  
   Se una risorsa è necessaria solo in una specifica pagina o sezione, il Provider dovrebbe essere posizionato appena sopra la radice di quella pagina o componente, in questo modo ogni volta che la pagina viene ricaricata i dati vengono riaggiornati.

---



## Esempi 

### Istanziazione del Model del carrello e il suo viewModel per un Ecommerce

```dart
//setto i model e view model globali
    //in questo caso il carello, i cui dati sono usati su entrambe le pagine e quindi
    //sono sempre attivi
    return MultiProvider(
      providers: [
        Provider(
          create: (_) {
            return ShoppingCartModel();
          },
        )
      ],
      //visto che per essere inizializzato il viewModel ha bisogno del model il suo provider
      //deve essere figlio di quello del model
      child: MultiProvider(providers: [
        Provider(
          create: (BuildContext context) {
            return ShoppingCartViewModel(context.read());
          },
          dispose: (_, value) {
            value.dispose();
          },
        )
      ], child: MaterialApp.router(
        title: 'Flutter Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
          useMaterial3: true,
        ),
        routerConfig: router,
      )),
    );

```

### Descrizione dell'Esempio: Istanziazione del Model del Carrello e del relativo ViewModel

In Questo esempio si fa utilizzo di MultiProvider, che in pratica corrisponde a specificare una serie di Provider tutti insieme per evitare di doverlo fare uno alla volta

Questo esempio dimostra come configurare un **Model** e un **ViewModel** globali per un'applicazione di e-commerce, utilizzando il package **Provider**.

#### Obiettivo
L'obiettivo è rendere il modello del carrello e il suo ViewModel disponibili globalmente nell'applicazione, poiché il carrello viene utilizzato in più pagine ed è necessario mantenerlo attivo durante l'intero ciclo di vita dell'app.

#### Dettagli dell'implementazione
- **ShoppingCartModel**:  
  Viene istanziato come risorsa globale attraverso un provider annidato all'inizio della gerarchia dei widget. Questo modello rappresenta i dati del carrello e, dato che è utilizzato su più pagine, deve essere sempre attivo.

- **ShoppingCartViewModel**:  
  Questo ViewModel richiede il **ShoppingCartModel** per funzionare. Per questo motivo, il suo provider viene dichiarato come figlio del provider del modello. Questo garantisce che il modello venga istanziato prima del ViewModel, rispettando la dipendenza.
- Infine come figlio del secondo provider si ha la MaterialApp, che sarebbe il widget che contiene l'intera applicazione, visto che è un discendente dei provider tutti gli elementi al suo interno (quindi l'intera app) possono far uso delle risorse dichiarate

#### Struttura del codice
1. **MultiProvider** principale:  
   Contiene il provider per il **ShoppingCartModel**, reso disponibile a tutti i widget discendenti.
2. **MultiProvider** figlio:  
   Qui viene configurato il provider per il **ShoppingCartViewModel**, che utilizza il metodo `context.read()` per accedere al modello istanziato nel provider principale.

#### Smaltimento delle risorse
Il metodo `dispose` viene implementato per il **ShoppingCartViewModel**, assicurandosi che eventuali risorse, come listener o registrazioni, vengano liberate correttamente quando il widget non è più necessario.

### Esempio pagina del carrello

``` dart
//La pagina del carrello
class ShoppingCartPage extends StatelessWidget {
  const ShoppingCartPage({super.key});

  Widget createWidget(BuildContext context){

    //ottengo il viewModel Dal contesto
    ItemsInCartViewModel viewModel = context.read();
    //grazie a ListenableBuilder faccio in modo che la lista si aggiorna ogni volta che
    //viene modificato il carrello
    return ListenableBuilder(
      builder: (context,_) {
        double width = MediaQuery.sizeOf(context).width;
        double height = MediaQuery.sizeOf(context).height;
        List<StoreItem> itemsInCart = viewModel.GetItems();
        return SizedBox(
          width: width,
          height: height,
          child: SingleChildScrollView(
            child: Column(children: [
              for(StoreItem item in itemsInCart)
                ShoppingItemCard(storeItem: item)
            ],),
          ),
        );
      }, listenable: viewModel.onCartChanged,
    );
  }

  @override
  Widget build(BuildContext context) {
    //istanzio il model che conterrà gli oggetti mostrati dalla pagina e il service
    //per ottenere i dati dal db
    return MultiProvider(
        providers: [
          Provider(create: (context) {
            return ShoppingItemsModel();
          }),
          Provider(create: (context) {
            return StoreItemService();
          })
        ],
        //istanzio il viewModel che rimpie il model instanziato sopra dei soli oggetti
        //presenti nel carrello
        child: Provider(
            create: (BuildContext context) {
              return ItemsInCartViewModel(context.read(), context.read(),context.read());
            },
            dispose: (_, value) {
              value.dispose();
            },
            builder: (context,_) {
              return createWidget(context);
            }));
  }

}

```

### Istanziazione delle risorse nella Pagina del Carrello

L'esempio si concentra su come la pagina del carrello instanzia le risorse necessarie e le gestisce tramite i provider. La struttura è organizzata in due parti principali: la definizione delle risorse e la costruzione dell'interfaccia grafica che le utilizza.

#### 1. **Definizione e Iniezione delle Risorse**
- La prima parte del codice nel metodo "build" si occupa di istanziare le dipendenze necessarie per la pagina tramite **MultiProvider**:
    - **ShoppingItemsModel**: contiene gli oggetti in vendita.
    - **StoreItemService**: gestisce le comunicazioni con l'esterno, come le API o il database.
    - **ItemsInCartViewModel**: utilizza le risorse sopra definite (model e service) per ottenere e gestire solo gli oggetti presenti nel carrello.

- Ogni risorsa è inserita all'interno dell'albero dei widget tramite i provider, permettendo di accedere a queste risorse nei widget figli utilizzando il **BuildContext**. Visto che la parte grafica della pagina, definita nel metodo createWidget, è figlia dei provider può accedere tramite context.read() ai viewModel necessari per il suo funzionamento

#### 2. **Costruzione dell'Interfaccia Grafica**
- La seconda parte del codice è gestita dalla funzione `createWidget`. Questa funzione crea e configura la parte grafica della pagina, utilizzando le risorse istanziate precedentemente.
    - Il **ItemsInCartViewModel**, ottenuto tramite `context.read()`, fornisce i dati necessari alla UI, come la lista degli oggetti nel carrello.
    - La funzione utilizza un **ListenableBuilder**, che permette di aggiornare automaticamente l'interfaccia ogni volta che i dati del carrello cambiano.

### Logica di Refresh Automatica delle Risorse

Grazie a questa struttura, ogni volta che la pagina viene visualizzata, le risorse necessarie vengono create e inizializzate. Ad esempio, i dati degli oggetti nel carrello vengono caricati nel model tramite una chiamata API eseguita nel costruttore del ViewModel.

Quando l'utente lascia la pagina e successivamente vi ritorna, i provider vengono di nuovo creati e di conseguenza le risorse vengono ricreate e i dati ricaricati da zero. Questo approccio implementa una logica di refresh automatica per la pagina e i suoi dati, integrata direttamente nella sua struttura. In questo modo, si garantisce che i dati siano sempre aggiornati senza dover gestire manualmente il loro stato.