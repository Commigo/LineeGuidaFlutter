# Introduzione all'architettura MVVM

L'architettura MVVM (Model-View-ViewModel) è un design pattern che prevede tre componenti principali:

- **Model**
- **ViewModel**
- **View**

Oltre a questi, se necessario, si utilizzano anche i **Service**. Di seguito, una panoramica dettagliata sul loro utilizzo e sulle responsabilità di ciascun componente.

---

## Model
Il **Model** rappresenta i dati attivi dell'applicazione. Non va considerato come l'intero insieme di dati (ad esempio, tutto il database), ma come i dati rilevanti per una specifica sezione o stato dell'app.

**Caratteristiche principali del Model:**

- Contiene i dati risultanti da operazioni come filtri o elaborazioni effettuate sui dati grezzi.
- Rappresenta lo stato attuale dell'applicazione.

**Esempio:** In una pagina che mostra una lista di utenti con filtri applicati, il **Model** contiene solo gli utenti filtrati e non l'intero database.

---

## ViewModel
Il **ViewModel** è il mediatore tra la **View** e il **Model**. La sua interfaccia deve essere strettamente legata alla **View**, considerando le seguenti responsabilità:

1. **Gestione degli input della View:**
    - Riceve input come testi digitati o pulsanti premuti.

2. **Fornitura dei dati alla View:**
    - Espone i dati dinamici necessari per la **View** (ad esempio, la lista filtrata degli utenti).

3. **Notifica di aggiornamenti:**
    - Informa la **View** quando devono essere visualizzati cambiamenti (ad esempio, dopo un'operazione sul **Model**).

**Flusso generale del ViewModel:**

1. Riceve gli input dalla **View**.
2. Usa questi input per aggiornare il **Model** (può interagire con i **Service** se necessario).
3. Il cambiamento del **Model** provoca una notifica alla **View**, che aggiorna la sua rappresentazione grafica tramite i getter forniti dal **ViewModel**.

---

## View
La **View** è responsabile dell'interfaccia utente e degli elementi grafici dell'applicazione.

**Caratteristiche principali della View:**

- Definisce la struttura degli elementi grafici.
- Utilizza i dati forniti dal **ViewModel** per:
    - Riempire gli elementi grafici (ad esempio, una tabella con i dati degli utenti).
    - Stabilire quali elementi grafici mostrare.
- Gestisce gli input dell'utente e li inoltra al **ViewModel**.

---

## Service
I **Service** gestiscono la comunicazione con risorse esterne, come file, API o altri sistemi.

**Caratteristiche principali dei Service:**

- Sono utilizzati esclusivamente dal **ViewModel**.
- Espongono le loro funzionalità tramite interfacce.
- I risultati delle loro operazioni possono essere utilizzati per aggiornare il **Model**.

**Esempi di utilizzo dei Service:**

- Lettura o scrittura di file.
- Comunicazione con API esterne.
- Operazioni asincrone come il download di dati.

---

## Schema di riferimento
Ecco uno schema rappresentativo dell'architettura MVVM:

![Schema MVVM](/Images/MVVMSchema.png)

*Le linee piene rappresentano una chiamata diretta ai metodi di quella classe, 
mentre quelle tratteggiate rappresentano un collegamento tramite il pattern Observer,
quindi notifiche asincrone tramite eventi*

