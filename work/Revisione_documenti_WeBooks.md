# Revisione del Problem Statement e del RAD WeBooks

Data della revisione: 16 novembre 2026.

Documenti esaminati: `work/ProblemStatement_WeBooks.docx` (versione 1.3), `work/RAD_WeBooks.docx` (versione 1.1), `WeBooks/deliverable/ProblemStatement_WeBooks.pdf` (versione 1.2). Esaminati il testo, le tabelle, i diagrammi e i mock-up incorporati nel RAD; consultato anche il sorgente PlantUML dei diagrammi di sequenza. Gli originali non sono stati modificati.

La valutazione riguarda la qualità interna e la coerenza fra questi documenti, non la conformità a una specifica griglia del docente o al libro di testo. Il rendering delle pagine Word non è stato possibile per assenza di LibreOffice nel runtime disponibile: non è quindi certificata l'impaginazione finale. I riferimenti seguenti usano sezioni e identificativi, non numeri di pagina non verificati.

## Valutazione complessiva

La base è buona per un progetto universitario: obiettivo comprensibile, scenari concreti, requisiti identificati, ambito ed esclusioni espliciti nel RAD, casi d'uso con precondizioni e postcondizioni, modello a oggetti, modello dinamico e mock-up. La distinzione Libro/Offerta e la conservazione del prezzo acquistato nella RigaOrdine sono scelte particolarmente utili per il dominio con più librai.

Il RAD richiede però una revisione di coerenza prima di diventare la base del design e dei test. I problemi principali riguardano chi può usare il carrello, quali oggetti vengono modificati nel catalogo, la disponibilità concorrente e la copertura delle operazioni amministrative. La quantità di diagrammi è buona; la corrispondenza tra le diverse rappresentazioni è ancora incompleta.

Priorità alta: può produrre comportamenti diversi a seconda della sezione seguita. Priorità media: lascia una decisione non specificata o ostacola tracciabilità e verifica. Priorità bassa: problema editoriale o terminologico.

## 1. Accesso al carrello e gerarchia degli attori

**Priorità alta. Riferimenti:** PS RF-06; RAD RF-06, UC-08, UC-09, UC-9.1, diagramma dei casi d'uso, modello delle classi e percorso del Visitatore.

RF-06 assegna il carrello al Lettore. Le schede UC-08/09 e le sequenze SD-8/9 lo assegnano al Visitatore; anche il percorso del Visitatore permette di riempire il carrello prima del login. Nel modello delle classi ogni Carrello appartiene invece a un Lettore.

Inoltre Utente registrato specializza Visitatore nel diagramma dei casi d'uso. Ne derivano anche la registrazione e la gestione del carrello per Libraio e Gestore utenti, senza che i requisiti descrivano questo comportamento. Il testo definisce poi Visitatore sia come persona senza account sia come persona non autenticata: non sono la stessa condizione.

**Correzione proposta:** mantenere il carrello riservato al Lettore, coerentemente con RF-06 e con il modello delle classi; aggiornare UC, SD e navigazione. Collegare esplicitamente Lettore e Visitatore alla ricerca e alla consultazione, separando la registrazione. Se si sceglie invece un carrello anonimo, specificare titolarità, durata e trasferimento/fusione al login e aggiornare requisiti e modello: non basta cambiare il nome dell'attore.

## 2. RF-12 non è coperto dai casi d'uso

**Priorità alta. Riferimenti:** RAD RF-12, UC-14/15, diagramma dei casi d'uso, MU-17 e percorso del Gestore utenti.

RF-12 consente al Gestore utenti di modificare e rimuovere qualsiasi libro. UC-14 e UC-15 sono riservati al Libraio, con precondizione relativa alla sua dashboard. Il diagramma non associa il Gestore a queste operazioni. MU-17 contiene invece i comandi Modifica/Rimuovi: l'interfaccia promette quindi una funzionalità non descritta dai casi d'uso.

**Correzione proposta:** aggiungere casi d'uso amministrativi oppure estendere quelli esistenti specificando attori, precondizioni, oggetti e autorizzazioni diversi. Completare navigazione e realizzazione dinamica corrispondenti. Una semplice associazione grafica non risolve la precondizione della dashboard del Libraio.

## 3. Libro condiviso e Offerta del singolo libraio

**Priorità alta. Riferimenti:** RF-09/10/12; UC-13/14/15; SD-13/14/15; RAD 3.4.3.1.

Il modello distingue correttamente il Libro bibliografico condiviso dall'Offerta con prezzo e disponibilità del singolo libraio. I flussi di catalogo continuano però a parlare genericamente di inserimento, modifica e rimozione del libro. In SD-14 compare `eliminaLibro(idLibro)`, senza identificazione dell'offerta o del libraio; SD-15 usa analogamente `updateLibro(nuoviDati)`.

Non è chiaro se il Libraio possa modificare i dati bibliografici comuni anche agli altri venditori. Non è descritto il ramo di UC-13 in cui l'ISBN esiste già, pur essendo richiesto da RF-09. La postcondizione di UC-14, «Il volume non è più acquistabile», è troppo generale se altri librai continuano a offrirlo.

**Correzione proposta:** distinguere esplicitamente creazione del Libro, creazione dell'Offerta, modifica dell'Offerta e rimozione dalla vendita dell'Offerta. Per ISBN già presente, riutilizzare il Libro e creare l'Offerta del Libraio; definire il caso in cui esiste già anche quella offerta. Stabilire chi modifica i dati bibliografici e cosa comporta una rimozione globale. Allineare i messaggi dei diagrammi alla conservazione dei dati storici già prescritta dal RAD.

## 4. Vincolo del carrello incompatibile con l'eccezione al checkout

**Priorità alta. Riferimenti:** PS RNF-03; RAD RNF-02, vincoli delle classi, UC-10.1 e SD-10.

Il vincolo dice che la quantità nel carrello non deve «mai» superare la disponibilità reale. Ma UC-10.1 contempla proprio un carrello divenuto non acquistabile. Esempio: A e B aggiungono entrambi l'ultima copia; A acquista, la disponibilità scende a zero e il carrello di B contiene ancora una copia. Non sono specificate prenotazioni dello stock né un aggiornamento automatico di tutti i carrelli.

**Correzione proposta:** se il carrello non prenota lo stock, richiedere la verifica all'aggiunta/modifica e una nuova verifica alla conferma, consentendo che un carrello diventi temporaneamente non soddisfacibile. Precisare che la conferma concorrente non può generare vendite oltre la disponibilità e che registrazione ordine, decremento stock e svuotamento carrello hanno esito unitario. SD-10 mostra oggi solo la sequenza positiva di questi passi: aggiungere il comportamento in caso di fallimento, senza dover anticipare il dettaglio implementativo della transazione.

## 5. Flussi alternativi non coerenti con il contesto

**Priorità alta. Riferimenti:** UC-05, UC-04.3, UC-08/09/09.1, SD-9.1 e scenario 3.

- UC-05 rinvia a UC-04.3 per email occupata. UC-04.3 richiede però un utente che modifica il proprio profilo e lo riporta al profilo: il Visitatore in registrazione non ne ha ancora uno. Serve un'alternativa che mantenga la pagina di registrazione e non crei l'account.
- UC-08 dichiara «Nessuna eccezione», ma lo scenario 3 richiede l'errore per quantità eccessiva durante l'aggiunta dalla scheda prodotto.
- UC-09 richiama UC-9.1 durante la modifica nel carrello, mentre SD-9.1 mostra un'aggiunta dalla SchedaProdotto_Boundary. La stessa regola di validazione può essere condivisa, ma origine, schermata finale e quantità conservata devono essere descritti per entrambi i contesti.

**Correzione proposta:** definire alternative specifiche per il caso d'uso chiamante; mantenere eventualmente un controllo comune, senza riutilizzare postcondizioni incompatibili. Aggiungere al checkout l'esito dei dati formalmente invalidi, attualmente verificati ma privi di un flusso di errore, e alla creazione del Libraio l'esito di email duplicata/dati invalidi.

## 6. Stato dell'ordine e stato della parte del libraio

**Priorità media. Riferimenti:** RF-08/11/13; UC-10/11/12/16/17/18; RigaOrdine; MU-09/10/15/16/18.

Il modello conserva lo stato sulle righe, mentre elenchi e UC-10 parlano anche di un singolo stato dell'Ordine. Nel dettaglio sono possibili righe con stati diversi; non è definita la regola che produce lo stato mostrato nell'elenco. UC-17 e SD-17 aggiornano la parte del Libraio in blocco, mentre il testo motiva lo stato per riga.

**Correzione proposta:** scegliere se tutte le righe dello stesso Libraio avanzano insieme o se possono avanzare separatamente. Definire lo stato riepilogativo come derivato, oppure mostrare esplicitamente i diversi stati. Portare in UC-17 il vincolo già presente nel modello dinamico: solo avanzamenti ammessi, senza ritorno a stati precedenti, con comportamento definito per tentativi invalidi. Non è indispensabile introdurre una nuova classe SottoOrdine se le regole scelte sono esprimibili con quelle esistenti.

## 7. Diagramma di stato dell'Offerta in conflitto con il testo

**Priorità media. Riferimenti:** RAD 3.4.4.2 e relativo diagramma Offerta.

Il testo ammette che un'offerta nasca Disponibile oppure Esaurita in base alle copie; il disegno passa da Creata a Disponibile senza guardia sulla quantità. Creata non compare nell'enumerazione StatoOfferta. L'aggiornamento copie nel disegno è un incremento positivo; i casi d'uso consentono genericamente di modificare la quantità, senza descrivere il passaggio a zero tramite aggiornamento manuale.

**Correzione proposta:** allineare gli stati e la distinzione iniziale in base alle copie, specificare se la modifica imposta una quantità assoluta o applica una variazione e coprire il passaggio a zero. Chiarire anche la ripubblicazione di un'offerta rimossa: il modello impone al più un'offerta per coppia Libro/Libraio e il diagramma rende terminale la rimozione; senza una regola non è chiaro come venderla nuovamente.

## 8. Tracciabilità fra classi e diagrammi di sequenza

**Priorità media. Riferimenti:** RAD 3.4.3 e 3.4.4.1.

Le sequenze usano `userRegister`, `Catalogo_Entity`, `Carrello_Entity` e `Ordine_Entity`. Non tutte queste denominazioni hanno una corrispondenza dichiarata nel modello delle classi; soprattutto i primi due oggetti sembrano rappresentare registri o collezioni, non le singole entità mostrate nel diagramma. Inoltre la tabella attribuisce UC-6.1 a Ricerca_Control, mentre SD-6.1 usa Catalogo_Control.

**Correzione proposta:** usare istanze tipizzate delle classi di analisi oppure spiegare e modellare gli oggetti ulteriori e le loro responsabilità. Allineare la tabella dei control ai diagrammi. Non è necessario che tutti i messaggi diventino subito operazioni dettagliate nel diagramma delle classi, ma deve essere riconoscibile quale oggetto li riceve.

## 9. Profilo e blocco account

**Priorità media. Riferimenti:** RF-15, UC-04, UC-20, SD-20, RNF-03.

RF-15 elenca nome, cognome, email e ruolo e poi dice di poterli modificare. UC-04 consente invece nome, cognome, email e password. Il ruolo dovrebbe essere esplicitamente di sola lettura nel profilo personale.

UC-20 impedisce nuovi login e rende non acquistabili le offerte del Libraio bloccato, ma non definisce la sorte delle sessioni già attive o degli ordini ancora da gestire. SD-20 omette la conferma prescritta nei passi 4-5 del caso d'uso e mostra soltanto il cambio di stato dell'account, senza rendere esplicito l'effetto sulle offerte.

**Correzione proposta:** distinguere dati visibili e modificabili; specificare gli effetti del blocco su sessioni, offerte e ordini pendenti, e rappresentare nel diagramma gli effetti richiesti. Non serve ampliare il progetto con nuove funzionalità: serve decidere il comportamento dei casi già inclusi.

## 10. Versioni e identificativi non allineati

**Priorità media. Riferimenti:** frontespizi PS, RAD 3.2 e 3.4.1, elenchi RNF.

Il PS Word è versione 1.3 del 29/10/2026; il PDF nei deliverable è versione 1.2 del 26/10/2026. Il RAD cita ancora la 1.2. Non è necessariamente errato mantenere una baseline precedente, ma bisogna dichiarare quale documento è autorevole e se il PDF è storico o la consegna corrente.

La numerazione RNF cambia significato fra PS e RAD:

| Contenuto | PS | RAD |
| --- | --- | --- |
| Prestazioni | RNF-01 | RNF-04 |
| Usabilità | RNF-02 | RNF-01 |
| Coerenza dati | RNF-03 | RNF-02 |
| Architettura | RNF-04 | RNF-06 |
| Manutenibilità | RNF-05 | RNF-05 |
| Sicurezza | RNF-06 | RNF-03 |

**Correzione proposta:** mantenere gli identificativi stabili o introdurre una tabella di corrispondenza ufficiale. RF-15 e la riduzione di RF-12 sono dichiarati come evoluzioni nel RAD: questo è positivo, ma anche il pagamento simulato e la baseline scelta devono essere riconciliati con il PS. Per RF-12 distinguere una creazione bibliografica da una messa in vendita: il Libro condiviso non appartiene a un singolo Libraio, a differenza dell'Offerta.

## 11. Requisiti di qualità e criteri di verifica

**Priorità media. Riferimenti:** RAD RNF-01/04/05 e 1.3.

Il limite di tre secondi e il massimo di tre passaggi sono buoni punti di partenza. «Almeno 100 utenti connessi» non specifica però quante richieste producano, dimensione del catalogo, ambiente di prova e criterio di misura. «Senza consultare alcun manuale» richiede utenti, compito e soglia di successo definiti. «Aggiungere nuove funzionalità senza modificare quelle esistenti» è troppo assoluto per una verifica oggettiva.

**Correzione proposta:** aggiungere condizioni e criteri di accettazione riproducibili, con valori concordati per il progetto. Per la manutenibilità scegliere regole verificabili sulla separazione delle responsabilità e sulla documentazione. Preparare una matrice RF/RNF → UC/modelli → test: non è necessario inventare nuove funzionalità per completarla.

## 12. Lacune editoriali e dati non allineati

**Priorità bassa o media secondo il punto.**

- La panoramica 1.4 annuncia una sezione 4 con il glossario, ma il documento termina con MU-20: aggiungere il glossario o correggere il rimando.
- Il modello dinamico annuncia un diagramma di stato del Carrello; sono presenti quelli di Account, Offerta e RigaOrdine, non quello del Carrello. Aggiungerlo se utile oppure rimuovere l'annuncio.
- UC-05 richiede un indirizzo nella registrazione, ma MU-03 e il modello Utente/Lettore non lo riportano. Decidere se raccoglierlo solo al checkout oppure modellare anche l'indirizzo del profilo.
- Libraio ha `nomeLibreria`, utilizzato nelle offerte mostrate, ma UC-19 e MU-20 non spiegano come sia acquisito. Aggiungerlo alla creazione o descrivere dove viene inserito.
- UC-1.1 dichiara come postcondizione «La sessione dell'utente è terminata», mentre il login non ha avuto successo; scrivere che non viene avviata una sessione autenticata. Il campo Eccezioni contiene invece una destinazione di navigazione.
- Nel diagramma dei casi d'uso la freccia `extend` fra Checkout ed Errore quantità in checkout punta verso l'errore; invertirla affinché l'estensione punti al caso base Checkout.
- Uniformare identificativi (`UC-01`/`UC 1`), Gestore utenti/UM, email/username e correggere refusi come «Librario», «maniente», «rleva», «caso di’uso», «in precedenze clicca» e i doppi punti finali.
- I mock-up aiutano a capire i flussi. Alcuni comandi sono però raggiunti nei casi d'uso dall'icona del profilo e nelle schermate da link diretti della barra: uniformare il percorso o descriverlo senza vincolarlo inutilmente al singolo controllo grafico.

## Qualità specifica del Problem Statement

La struttura copre problema, obiettivi, scenari, requisiti, ambiente e consegne. Gli scenari usano attori e dati concreti e includono un fallimento di disponibilità, utile per l'analisi.

La motivazione del mercato usa però affermazioni molto generali, come la mancanza di un ecosistema dedicato e la prevalenza di certe pratiche commerciali, senza fonti nel documento. Per il progetto è più solido circoscrivere il problema alle librerie e agli utenti di riferimento, evitando di sostenere l'assenza assoluta di alternative. Questa è una valutazione dell'argomentazione, non una ricerca di mercato.

Il PS dovrebbe chiarire già negli obiettivi che si tratta di un catalogo condiviso fra più librai, distinguere i vincoli imposti dal corso dalle scelte progettuali e allineare l'ambito del pagamento al RAD. Il pannello del venditore è un'interfaccia utente servita dal sistema: definirlo semplicemente «lato server» può confondere presentazione e logica applicativa.

## Ordine di intervento suggerito

1. Fissare le regole di dominio: carrello anonimo o autenticato, Libro/Offerta, stock, stati degli ordini e poteri del Gestore.
2. Aggiornare i requisiti e dichiarare la baseline del Problem Statement.
3. Correggere i casi d'uso e i flussi alternativi, completando RF-12.
4. Allineare classi, sequenze, stati, percorsi e mock-up alle stesse decisioni.
5. Completare tracciabilità, criteri di accettazione, glossario e revisione editoriale.

Non occorre riscrivere tutto il lavoro: la struttura è riutilizzabile. La priorità è rendere ogni comportamento leggibile nello stesso modo nei requisiti, nei casi d'uso e nei modelli.
