# MovieLens Diversity Decay 
---

## Descrizione:
Questo progetto di tirocinio analizza la diversità dei consumi culturali degli utenti su piattaforme di raccomandazione, con l’obiettivo di studiare la possibile riduzione della diversità nel tempo (diversity decay).
---

## Obiettivo del progetto
L’obiettivo generale è analizzare come evolvono nel tempo la diversità e la novelty dei consumi cinematografici osservati su MovieLens, e valutare se le dinamiche osservate siano compatibili con l'ipotesi di diversity decay discussa nel contesto dei sistemi di raccomandazione.
---

## Dataset utilizzato
- MovieLens 25M dataset
- File principali utilizzati:
  - ratings.csv
  - movies.csv
  - links.csv

Il dataset è stato scaricato e organizzato localmente nella cartella `data/raw/`.
---

# Setup ambiente di lavoro su nuova macchina
Per eseguire correttamente il progetto su un altro computer è necessario configurare l’ambiente Python e scaricare il dataset richiesto.
1. creare ambiente Python
conda create -n movielens python=3.10
conda activate movielens

2. installare dipendenze
pip install -r requirements.txt

3. registrazione kernel Jupyter (per garantire la corretta esecuzione nei notebook)
python -m ipykernel install --user --name movielens --display-name "Python (movielens)"

Il dataset MovieLens 25m non è incluso
3. scaricare il database da: https://grouplens.org/datasets/movielens/25m/
4. estrarre il file .zip scaricato
5. posizionare la cartella ml-25m dentro: data/raw/
4. eseguire i singoli notebook in ordine 

I notebook devono essere eseguiti utilizzando il kernel Python dell’ambiente Conda `movielens`. 

# Verifica ambiente Python nei notebook
Per controllare l'ambiente attivo:
import sys
print(sys.executable)

l'output deve contenere: .../envs/movielens/bin/python, se diverso selezionare il kernel "Python (movielens)
---

## Notebook 00_setup
Nella fase iniziale è stata svolta la configurazione del progetto e la preparazione dei dati.

# 1. Setup ambiente
- Configurazione ambiente Python
- Import delle principali librerie (pandas, numpy, matplotlib)

# 2. Struttura progetto
Struttura delle cartelle inizializzata in fase di sviluppo:
data/
notebooks/
figures/

# 3. Caricamento dataset
Caricamento dei file MovieLens:
  - ratings.csv
  - movies.csv
  - links.csv
Verifica dimensioni dataset e struttura dati

# 4. Creazione mapping IMDb
È stato creato un mapping tra MovieLens e IMDb:
  - conversione `imdbId` → formato standard IMDb ID
  - creazione file: `imdb_mapping.parquet` salvato in `data/processed/`

## Stato attuale pipeline
• Dataset caricato  
• Struttura progetto definita  
• Mapping MovieLens → IMDb creato  
• Ambiente di lavoro configurato  
• Primo notebook (`00_setup`) completato  
---

## Notebook 01_preproc
In questa fase viene costruito il dataset analitico a partire dai dati grezzi di MovieLens.

L’obiettivo è trasformare i dati in una struttura longitudinale utente-tempo utile per l'analisi della diversità dei consumi e della sua evoluzione nel tempo

# 1. Preprocessing dei dati
Sono stati caricati i file principali del dataset:
  - ratings.csv
  - movies.csv
provenienti dal dataset MovieLens 25M.
Successivamente è stata effettuata una pulizia preliminare dei dati:
  - rimozione dei generi con valori "(no genres listed)"
  - sostituzione dei valori mancanti con “Unknown” nella variabile "genres"
  - conversione del timestamp in variabile temporale (year) per l’analisi temporale del comportamento degli utenti.

# 2. Trasformazione dei generi
I generi sono stati trasformati da stringa a lista ed espansi tramite `explode`, ottenendo una riga per ogni genere associato a una valutazione (coppia rating-genere), per permettere il conteggio corretto del numero di interazioni dell'utente con ciascun genere.

# 3. Integrazione dei dati
I dataset `ratings` e `movies` sono stati uniti tramite `movieId` ottenendo per ogni valutazione:
  - identificativo dell'utente
  - anno della valutazione
  - film valutato
  - generi associati ai film
Per associare:
  - comportamento dell'utente
  - informazione sui contenuti (generi)

# 4. Aggregazione dei dati
I dati sono stati aggregati per:
  - utente (`userId`)
  - anno (`year`)
  - genere (`genres`)
ottenendo, per ogni anno, il numero di interazioni associate a ciascun genere per ogni utente e anno.
Successivamente è stata costruita una matrice utente-anno in cui ogni colonna rappresenta uno dei 19 generi ufficiali di MovieLens

# 5. Costruzione delle metriche
 • Novelty dei generi
   Per ogni utente e anno è stata calcolata la percentuale di nuovi generi osservati rispetto alla storia passata dell'utente. 
   La metrica può assumere valori tra 0 e 1:
   - novelty = 0
    → tutti i generi dell’anno erano già stati osservati negli anni precedenti
    → nessuna esplorazione nuova
   - novelty = 1
    → tutti i generi dell’anno sono nuovi rispetto alla storia dell’utente
    → massima esplorazione
   - 0 < novelty < 1
    → presenza parziale di nuovi generi rispetto al passato
    → esplorazione parziale
 • Entropia di Shannon
   Per ogni coppia utente-anno è stata calcolata l'entropia di Shannon della distribuzione dei generi,  per misurare quanto è diversificato il consumo di un utente.
   Valori più alti = maggiore diversità, consumi distribuiti tra molti generi
   Valori più bassi = poca diversità, consumi concentrati su pochi generi

# 6. Costruzione del campione longitudinale
Per rendere l'analisi coerente con l'obiettivo dello studio del diversity decay, il dataset è stato filtrato secondo due criteri:
• Filtro sul volume di rating
  Per ogni coppia utente–anno è stato calcolato il numero totale di rating (`total_ratings`).
  Sono state eliminate le osservazioni con meno di 5 rating nell'anno, poiché producono stime troppo rumorose delle metriche di diversità.

• Filtro sugli utenti longitudinali
  Successivamente sono stati mantenuti soltanto gli utenti con almeno 2 anni attivi dopo il filtro precedente.
  In questo modo il dataset finale contiene esclusivamente utenti per i quali è possibile osservare un'evoluzione temporale del comportamento.

# 7. Variabili longitudinali
Dopo il filtraggio il dataset è stato ordinato cronologicamente per utente.
È stata quindi costruita la variabile `active_year`, che identifica il numero progressivo dell'anno di attività dell'utente (1, 2, 3, ...).
Questa variabile verrà utilizzata nei notebook successivi per distinguere l'anzianità dell'utente dal semplice anno di calendario.

# 8. Dataset finale
Il dataset finale contiene:
  - userId
  - year
  - distribuzione dei 19 generi (conteggi)
  - entropy
  - novelty_pct (esplorazione nuovi generi)
  - n_ratings_year
  - total_ratings
  - active_year

# 7. Output del notebook
Il dataset è stato salvato in formato Parquet:'data/processed/user_year_genre_longitudinal.parquet'

e sarà utilizzato nei notebook successivi per:
 - regressione a effetti fissi
 - analisi esplorativa (EDA)
 - studio e visualizzazione del diversity decay
 - modellazione statistica
 - integrazione della metrica geografica tramite dai IMDb

 ## Stato attuale pipeline
• Caricamento e preprocessing dataset MovieLens 25M  
• Pulizia variabile `genres` (gestione valori mancanti e rimozione "(no genres listed)")  
• Creazione variabile temporale (`year`) da timestamp  
• Unione dataset `ratings` e `movies`  
• Espansione dei generi tramite `explode`  
• Aggregazione per utente–anno–genere  
• Costruzione matrice utente-anno con distribuzione dei generi  
• Calcolo entropia di Shannon per misurare la diversità dei consumi  
• Calcolo novelty dei generi (percentuale di nuovi generi per utente nel tempo)  
• Calcolo del numero di rating annuali
• Costruzione del campione longitudinale (≥ 2 anni attivi)
• Esclusione delle osservazioni con meno di 5 rating/anno
• Creazione della variabile `active_year`
• Salvataggio dataset finale in formato Parquet 
• Secondo notebook (`01_preproc`) completato  
---

# Notebook 02_EDA
In questa fase è stata svolta un'analisi esplorativa del dataset longitudinale costruito nel notebook precedente, con l'obiettivo di:
 - verificare le caratteristiche del campione filtrato
 - validare preliminarmente il modello econometrico proposto
 - costruire il campione stratificato da utilizzare nelle analisi esplorative
 - analizzare l'evoluzione della diversità dei consumi nel tempo
 - valutare i limiti strutturali della metrica di Novelty

L'EDA rappresenta una fase fondamentale del progetto dato che permette di controllare la qualità del campione, individuare eventuali anomalie e verificare la correttezza delle metriche longitudinali utilizzate nelle analisi sul fenomeno del diversity decay.

# 0. Aggiornamento del setup dell'ambiente
Dato che il notebook 02_EDA utilizza la libreria Seaborn per la costruzione delle visualizzazini, è stata aggiunta anche questa dipendenza nel file requirements.txt.
Quindi installare nuovamente le dipendenze eseguendo nel terminale: 
pip install -r requirements.txt

# 1. Caricamento del dataset longitudinale
È stato caricato il dataset prodotto nel notebook precedente (user_year_genre_longitudinal.parquet).
Il dataset contiene, per ogni coppia utente–anno:
  - distribuzione dei consumi per genere 
  - entropia di Shannon
  - percentuale di nuovi generi esplorati (Novelty)
  - numero di rating annuali
  - variabile active_year, che rappresenta il numero di anni di attività dell'utente,
  - volume complessivo di rating

Sono state inoltre verificate:
 - dimensioni del dataset
 - struttura delle variabili
 - tipi di dato
 - intervallo temporale osservato

# 2. Preparazione delle variabili per le regressioni
Prima delle analisi statistiche vengono costruite le variabili necessarie alla stima dei modelli a effetti fissi.
 - viene applicata la trasformazione logaritmica del numero di rating annuali (log_ratings) per attenuare l'asimmetria del volume di attività annuale
 - vengono costruite le variabili demeaned mediante within transformation, sottraendo a ciascuna osservazione la media individuale dell'utente. Questa trasformazione consente di eliminare gli effetti individuali invarianti nel tempo tipici dei modelli a effetti fissi.
 - viene creato il sottocampione della Novelty escludendo il primo anno di attività (active_year = 1), nel quale la metrica assume valore unitario per costruzione

# 3. Regressioni prelimnari a effetti fissi ---------
Come richiesto durante la revisione metodologica, viene eseguita una prima verifica del modello econometrico sul campione corretto.
Le regressioni stimano la relazione tra:
• Entropia
  - variabile dipendente: Shannon Entropy
  - regressore principale: active_year
  - controllo: log_ratings
  - effetti di calendario: C(year)
  - errori standard clusterizzati per utente
• Novelty
medesima specificazione, limitatamente agli anni con active_year ≥ 2.

Le stime mostrano che:
- l'entropia cresce leggermente con gli anni di attività dell'utente
- la Novelty diminuisce progressivamente nel tempo
- il volume di rating risulta associato positivamente alle metriche considerate
Questa fase costituisce una verifica preliminare della specifica econometrica prima delle analisi descrittive.

# 4. Analisi della saturazione della Novelty
Per interpretare correttamente il coefficiente negativo della Novelty vengono costruite due analisi aggiuntive.
 • Saturazione annuale
   Per ciascun anno di attività dell'utente (`active_year`) vengono calcolati:
   - numero medio di generi distinti consumati annualmente (`avg_genres_per_year`)
   - entropia media di Shannon (`avg_entropy`)
   - Novelty media (`avg_novelty_pct`)
   I risultati evidenziano una dinamica differente tra esplorazione e diversità dei consumi:
   - la Novelty crolla rapidamente dopo il primo anno di attività 
     → passa da circa 99.9% nel primo anno a 5.6% nel secondo anno
     → raggiunge valori inferiori all'1% negli anni successivi. 
   - il numero medio di generi consumati rimane invece stabile 
     introno a 13.8-14 generi
   - anche l'entropia media rimane sostanzialmente costante, con 
     valori compresi circa tra 3.26 e 3.28
   Questa evidenza suggerisce che il calo della Novelty non corrisponde ad una riduzione della varietà dei consumi, ma ad una diminuzione della possibilità di incontrare nuovi macro-generi già dopo le prime fasi di attività.

 • Saturazione cumulativa
   Per verificare questa ipotesi viene analizzato anche il numero cumulativo di generi esplorati dagli utenti nel corso della loro attività sulla piattaforma.
   I risultati mostrano che:
   - già nel primo anno di attività gli utenti hanno esplorato mediamente circa 16.84 dei 19 generi disponibili
   - entro il terzo anno la media cumulativa raggiunge circa 18.29 generi
   - la maggior parte degli utenti ha quindi già raggiunto una condizione di quasi saturazione dello spazio dei macro-generi
   Questo risultato conferma che il calo della Novelty sia dovuto principalmente ad un effetto soffitto della metrica, piuttosto che ad una reale riduzione della propensione degli utenti a esplorare nuovi contenuti.

# 5. Analisi descrittiva del dataset
Sono state calcolate le principali statistiche descrittive del campione:
  - numero totale di osservazioni
  - numero di utenti presenti
  - periodo temporale coperto dal dataset
  - distribuzione dell'entropia
  - distribuzione della Novelty
  - numero di anni osservati per ciascun utente
Queste informazioni consentono di avere una prima idea del comportamento complessivo della popolazione analizzata.

# 6. Costruzione del campione stratificato
Il dataset MovieLens 25M contiene oltre 160.000 utenti.
Per limitare il costo computazionale delle analisi longitudinali è stato costruito un campione di 10.000 utenti.
Prima della selezione del campione è stata analizzata la distribuzione dell'attività degli utenti.
• Istogramma della distribuzione dell'attività degli utenti
  L'istogramma mostra la distribuzione del numero totale di rating espressi dagli utenti del dataset.
  Ogni barra rappresenta il numero di utenti che hanno espresso un certo numero di valutazioni.
  Dal grafico emerge una distribuzione fortemente asimmetrica:
   - la maggior parte degli utenti ha espresso un numero relativamente basso di rating
   - una piccola parte di utenti è invece estremamente attiva e produce migliaia di valutazioni
  Questo comportamento è tipico delle piattaforme online, nelle quali pochi utenti generano una quota consistente delle interazioni.
  Questa forte eterogeneità giustifica la scelta di effettuare un campionamento stratificato, evitando che il campione sia composto quasi esclusivamente da utenti poco attivi.

• Boxplot dell'attività degli utenti in scala logaritmica
  Il boxplot riassume la distribuzione del numero di rating per utente utilizzando:
   - mediana
   - quartili
   - valori estremi
  Dato che esistono utenti con un numero di rating molto elevato (outlier), il grafico è rappresentato in scala logaritmica per rendere visibili anche gli utenti meno attivi.
  Il grafico evidenzia una forte variabilità nell'attività degli utenti e la presenza di outlier molto distanti dalla distribuzione centrale.
  Questo grafico rafforza la necessità della stratificazione del campione e mostra che l'attività degli utenti non è distribuita in modo uniforme.

Per tenere conto di questa forte disuguaglianza, gli utenti sono stati suddivisi in 3 gruppi in base al numero totale di rating espressi:
  - Low activity
  - Medium activity
  - High activity
La suddivisione è stata effettuata tramite quantili (qcut), così da ottenere gruppi bilanciati rispetto alla distribuzione.
Successivamente è stato estratto un campione casuale stratificato da ciascun gruppo fino a ottenere 10.000 utenti totali, mantenendo una rappresentazione bilanciata dei diversi livelli di attività.

Il dataset campionato è stato salvato nel file: data/processed/sample_users_10k.parquet,
e verrà utilizzato nelle analisi successive.

# 7. Verifica delle regressioni sul campione stratificato
Per verificare la robustezza dei risultati rispetto alla riduzione del campione, le regressioni a effetti fissi vengono replicate sul campione stratificato di 10.000 utenti.
I coefficienti ottenuti risultano coerenti con quelli stimati sull'intero campione longitudinale, confermando la rappresentatività del campione stratificato.

Dopo la selezione del campione stratificato di 10.000 utenti, le regressioni a effetti fissi vengono ricalcolate utilizzando esclusivamente il nuovo campione.
Prima della stima dei modelli vengono ricostruite le variabili trasformate tramite demeaning per utente, necessario per controllare gli effetti individuali invarianti nel tempo.
Vengono stimati due modelli:
 • Entropia
   La regressione analizza l'effetto dell'invecchiamento dell'account sulla diversità dei consumi controllando per:
   - volume annuale di rating (log_ratings)
   - anno solare (C(year)), per catturare eventuali effetti temporali comuni alla piattaforma.
   Il modello stima  se, a parità di attività e periodo storico, gli utenti tendano nel tempo a diversificare o restringere i propri consumi.
 • Novelty 
   La regressione sulla Novelty viene effettuata su un sotto-campione che esclude il primo anno di attività (active_year >= 2), poiché il primo anno è caratterizzato da una quota artificiosamente elevata di nuovi generi esplorati.
   Anche in questo caso vengono inclusi:
   - effetti temporali di calendario
   - controllo per volume di rating
   - effetti individuali tramite demeaning.
I risultati delle regressioni vengono utilizzati successivamente per interpretare le dinamiche osservate nelle visualizzazioni

# 8. Definizione delle coorti
Per ogni utente è stato identificato il primo anno di attività all'interno del dataset (`cohort year`) che rappresenta il momento di ingresso dell'utente nella piattaforma.
Le coorti consentono di confrontare utenti che hanno iniziato ad utilizzare il sistema nello stesso periodo, rendendo possibile uno studio longitudinale dell'evoluzione dei comportamenti di consumo.

# 9. Definizione della finestra temporale
L'analisi longitudinale viene costruita utilizzando due dimensioni temporali complementari:
- anno solare (year): rappresenta il periodo storico della 
  piattaforma e permette di controllare eventuali cambiamenti generali del catalogo o del comportamento degli utenti;
- anno di attività (active_year): rappresenta il tempo biologico 
  dell'utente sulla piattaforma, ovvero gli anni trascorsi dal primo utilizzo.
La distinzione tra anno solare e anno di attività è fondamentale perché permette di separare gli effetti legati al ciclo di vita dell'utente dagli effetti temporali comuni della piattaforma.
Nel notebook viene adottata principalmente la prospettiva dell'anno solare per la definizione delle osservazioni, mentre active_year viene utilizzato per analizzare l'evoluzione del comportamento durante il ciclo di vita dell'utente.

# 10. Analisi dell'evoluzione della diveristà
È stata analizzata l'evoluzione della diversità dei consumi degli utenti attraverso l'entropia di Shannon calcolata sulla distribuzione annuale dei generi cinematografici.
La metrica permette di valutare se gli utenti, con il passare degli anni sulla piattaforma, mantengano una distribuzione ampia dei propri interessi oppure tendano a concentrarsi su pochi generi.
Prima dell'analisi longitudinale sono state calcolate alcune statistiche descrittive della metrica per comprenderne la distribuzione nel campione.

Sono stati prodotti due grafici: 
• Evoluzione dell'entropia delle principali coorti
  Il primo grafico mostra l'andamento dell'entropia media per le cinque coorti più numerose.
  Ogni linea rappresenta una coorte, ovvero un gruppo di utenti entrati nella piattaforma nello stesso anno.
  L'asse orizzontale rappresenta gli anni trascorsi dal primo utilizzo della piattaforma (`active_year`), mentre l'asse verticale rappresenta il valore medio dell'entropia di Shannon.
  Il grafico permette di osservare come cambia nel tempo la diversità dei consumi per utenti appartenenti a generazioni diverse.
  In particolare permette di verificare se, dopo i primi anni di utilizzo, gli utenti:
   - continuano ad esplorare molti generi
   - mantengono stabile la varietà dei consumi
   - tendono invece a concentrarsi progressivamente su pochi generi
  Questo è uno dei grafici più importanti dato che costituisce la prima verifica visiva dell'ipotesi di diversity decay.
  I punti in cui le curve mostrano una diminuzione sistematica dell'entropia, potrebbero indicare una progressiva riduzione della varietà dei contenuti consumati.
  Nel campione analizzato le curve risultano generalmente stabili, suggerendo che la varietà annuale dei consumi non diminuisce in modo evidente con l'invecchiamento dell'account.

• Andamento medio dell'entropia
  Il secondo grafico mostra l'entropia media considerando tutti gli utenti del campione per ciascun anno osservato.
  Non distingue più le singole coorti ma osserva il comportamento complessivo della popolazione.
  Permette di individuare eventuali cambiamenti generali della diversità dei consumi nel corso degli anni.
  Eventuali aumenti o diminuzioni possono suggerire cambiamenti nel comportamento degli utenti oppure effetti legati all'evoluzione della piattaforma
  L'andamento osservato evidenzia una sostanziale stabilità dell'entropia media, con valori intorno a 3.26–3.28.
  Questo suggerisce che gli utenti mantengono una distribuzione dei consumi ampia e diversificata anche dopo diversi anni di attività sulla piattaforma.
  Questa osservazione è coerente con la regressione a effetti fissi, dove l'effetto dell'anzianità dell'account sull'entropia risulta positivo: β(active_year) =0.0063, p<0.001,  indicando un leggero aumento della diversità dei consumi nel tempo.
  La regressione conferma quindi che il passare degli anni sulla piattaforma non determina una riduzione della varietà consumata, ma un leggero aumento della diversificazione osservata.

# 11. Heatmap della distribuzione dei generi
È stata costruita una heatmap che rappresenta la distribuzione media dei generi per ciascuna coorte
Nella heatmap:
 - ogni riga rappresenta una coorte di utenti (`cohort_year`)
 - ogni colonna rappresenta un genere
 - l'intensità del colore indica il livello medio di consumo del genere → colori più intensi corrispondono ad una maggiore presenza del genere nei consumi della coorte.
Questo grafico permette di individuare eventuali differenze nelle preferenze cinematografiche tra utenti nella piattaforma in periodi differenti, e eventuali cambiamenti strutturali nei gusti degli utenti nel corso degli anni.
è possibile verificare se alcune coorti mostrano una maggiore propensione verso un genere piuttosto che un altro, oppure se la distribuzione dei generi rimane abbastanza stabile

# 12. Analisi della Novelty
Per studiare la capacità degli utenti di esplorare nuovi contenuti sono stati realizzati due grafici (analizzando la metrica novelty_pct):
• Distribuzione della Novelty
  L'istogramma mostra come sono distribuiti i valori della metrica Novelty, che misura la quota di generi nuovi esplorati da ciascun utenti in un determinato anno.
  La metrica assume valori compresi tra 0 e 1.
  Nel campione si osserva una concentrazione dei valori agli estremi:
   - molti utenti presentano valori a 0, indicando l'assenza di esplorazione di nuovi generi
   - molti altri presentano valori pari a 1, indicando invece un'esplorazione completa di nuovi generi
  Il grafico consente di valutare quanto frequentemente gli utenti esplorino nuovi contenuti.
  La distribuzione evidenzia una forte concentrazione verso valori bassi, mostrando che dopo le prime fasi di utilizzo la scoperta di nuovi generi diminuisce rapidamente.

• Evoluzione temporale della Novelty
  Il grafico mostra il valore medio della Novelty per ciascun anno di attività dell'utente osservato.
  Ogni punto rappresenta la capacità media degli utenti di esplorare nuovi generi in quell'anno.
  L'andamento della curva permette di verificare se, nel corso del tempo, gli utenti tendano ad ampliare oppure restringere i propri interessi cinematografici.
  In questo caso, l'andamento evidenzia un forte decadimento:
  - nel primo anno gli utenti esplorano quasi tutti nuovi generi
  - già dal secondo anno la quota di nuovi generi diminuisce 
    drasticamente
  - negli anni successivi la Novelty tende verso valori prossimi allo 
    zero
  Questo comportamento non indica necessariamente una perdita di curiosità, ma è principalmente dovuto all'effetto soffitto della metrica: i 19 generi disponibili vengono rapidamente esplorati dalla maggior parte degli utenti.
  Per questo motivo, l'analisi sarà approfondita nei notebook successivi attraverso modelli statistici dedicati.

Questi due grafici insieme permettono di descrivere sia la varietà dei consumi sia la capacità di scoperta di nuovi contenuti (aspetti centrali del progetto)

# 13. Analisi dell'Entropia
Per completare l'analisi della diversità dei consumi sono stati aggiunti due grafici descrittivi sulla distribuzione dell'entropia:
 • Distribuzione dell'Entropia di Shannon
  L'istogramma mostra la distribuzione complessiva dei valori di entropia nel campione dei 10.000 utenti.
  Il grafico permette di valutare quanto siano eterogenei i comportamenti degli utenti rispetto alla varietà dei generi consumati.
  La distribuzione mostra una prevalenza di valori medio-alti di entropia, indicando che nella maggior parte degli anni gli utenti mantengono consumi distribuiti tra diversi generi.
 
 • Evoluzione dell'Entropia nel tempo biologico
   Il grafico mostra l'entropia media per ciascun anno di attività dell'utente (active_year).
   Questa rappresentazione permette di osservare direttamente se la diversità dei consumi diminuisce con l'invecchiamento dell'account.
   L'andamento conferma la stabilità osservata nelle analisi precedenti:
   - non emerge un decadimento della varietà dei consumi
   - l'entropia rimane pressoché costante nei primi anni di attività
   - gli utenti mantengono una distribuzione ampia dei propri interessi cinematografici

  Il confronto tra andamento dell'entropia e andamento della Novelty evidenzia che il calo della capacità di esplorare nuovi generi non coincide con una riduzione della varietà complessiva dei consumi.

# Conclusioni
L'analisi evidenzia una differenza sostanziale tra le due metriche utilizzate, tra esplorazione di nuovi generi e diversità complessiva dei consumi.
La Novelty diminuisce rapidamente nel tempo perché gli utenti raggiungono rapidamente la saturazione dei 19 macro-generi disponibili.
Al contrario, l'entropia di Shannon rimane stabile, indicando che gli utenti continuano a distribuire i propri consumi tra numerosi generi anche dopo diversi anni di attività.
Quindi, il decadimento osservato nella Novelty non rappresenta necessariamente un diversity decay reale, ma principalmente un limite della granularità della metrica utilizzata.
Questa evidenza suggerisce che, nelle fasi successive del progetto, sarà opportuno affiancare alla Novelty basata sui macro-generi una misura più granulare dell'esplorazione (ad esempio mediante sottogeneri).

In breve possiamo dire che: la Novelty cala, ma la diversità reale dei consumi (Entropy) non cala → il problema è la granularità dei 19 generi.

# Stato attuale della pipeline
• Caricamento del dataset longitudinale
• Preparazione delle variabili per regressioni a effetti fissi
• Verifica della qualità dei dati
• Analisi della saturazione della Novelty
• Verifica dell'effetto soffitto dei macro-generi
• Calcolo delle statistiche descrittive principali
• Costruzione del campione stratificato di 10.000 utenti
• Replica delle regressioni sul campione
• Definizione delle coorti di ingresso degli utenti
• Definizione della finestra temporale annuale
• Costruzione delle principali visualizzazioni esplorative
• Dataset campionato salvato in formato Parquet (`sample_users_10k.parquet`)
• Terzo notebook (`02_EDA`) completato
---

# Notebook 03_geographic_diversity
In questo notebook viene costruita la terza metrica richiesta dal progetto: la diversità geografica dei consumi cinematografici.
L'obiettivo è estendere l'analisi della diversità degli utenti oltre la dimensione dei generi, verificando se nel corso degli anni gli utenti mantengano una distribuzione varia anche rispetto ai paesi di produzione dei film consumati.
La metrica costruita è analoga all'entropia di Shannon utilizzata nei notebook precedenti per misurare la diversità dei generi, ma applicata alla distribuzione dei paesi di produzione dei film.
L'analisi segue tre passaggi principali:
 - recupero del paese di produzione dei film tramite il mapping IMDb–TMDb
 - costruzione della distribuzione annuale dei consumi per paese per ciascun utente
 - calcolo dell'entropia geografica a livello utente–anno

# 0. Aggiornamento del setup dell'ambiente
Dato che il notebook 03_geographic_diversity utilizza la libreria tqdm per monitorare l'avanzamento delle operazioni di recupero dei dati geografici tramite API e delle elaborazioni sui dataset.
Per questo è stata aggiunta la dipendenza nel file requirements.txt.
Quindi installare nuovamente le dipendenze eseguendo nel terminale: 
pip install -r requirements.txt

# 1. Caricamento dei dati
Per costruire la metrica geografica vengono utilizzati:
 - ratings.csv contenente le interazioni utente–film
 - imdb_mapping.parquet, creato nella fase precedente, contenente il collegamento tra movieId di MovieLens e
   identificativo IMDb
Il dataset dei rating contiene:
 - 25.000.095 osservazioni
 - 162.000+ utenti
 - 62.423 film
Il file imdb_mapping.parquet contiene:
 - 62.423 film
 - corrispondenza tra movieId e imdbId
Questo mapping viene utilizzato come punto di partenza per recuperare le informazioni geografiche dei film.

# 2. Recupero del paese di produzione dei film
Il dataset MovieLens non contiene direttamente informazioni relative al paese di produzione, per ottenere questa informazione viene utilizzata l'API di TMDb:
 - l'IMDb ID viene convertito nel corrispondente TMDb ID
 - dal dettaglio del film viene estratto il campo production_countries
 - viene salvato il primo paese di produzione disponibile
Prima dell'elaborazione completa è stato effettuato un test su un film campione per verificare il corretto funzionamento delle chiamate API.

# 3. Costruzione del mapping film–paese
Per ogni film presente nel dataset è stato costruito un nuovo mapping contenente:
 - movieId
 - imdbId_str
 - country
Il mapping finale è stato salvato nel file: `movie_country_mapping.parquet`

La copertura ottenuta è:
 - copertura dei film: 95.15%
 - film senza paese disponibile: 3.026
Nonostante una piccola quota di film senza informazione geografica, la copertura sui consumi effettivi è molto elevata →  rating con paese disponibile: 99.72%
Questo significa che i film mancanti sono prevalentemente titoli poco presenti nel dataset e non influenzano significativamente l'analisi longitudinale.

# 4. Costruzione della distribuzione geografica utente–anno
Dopo il recupero dei paesi, il dataset dei rating viene unito al mapping geografico.
Per ogni coppia utente - anno - paese, viene calcolato il numero di film consumati (`n_films`).
Successivamente viene calcolata la quota relativa di consumo di ciascun paese.
Questa distribuzione permette di misurare quanto i consumi annuali siano concentrati o distribuiti tra diversi paesi.

# 5. Calcolo dell'entropia geografica
La diversità geografica viene misurata utilizzando l'entropia di Shannon, in cui:
 - valori elevati indicano una distribuzione più uniforme tra più paesi
 - valori bassi indicano una maggiore concentrazione verso pochi paesi.
Per ogni utente e anno viene quindi ottenuta la variabile `geo_entropy` analoga alla variabile entropy costruita per i generi nei notebook precedenti.

# 6. Metriche aggiuntive calcolate
Oltre all'entropia geografica è stato calcolato anche il numero di paesi distinti esplorati annualmente (`n_countries`).
Questa variabile permette di distinguere:
 - ampiezza dello spazio geografico esplorato
 - distribuzione dei consumi tra i paesi

Esempio:
• userId = 1
• year = 2006
• geo_entropy = 3.42
• n_countries = 18
indica un consumo distribuito tra numerosi paesi con una diversità geografica elevata.

# 7. Dataset longitudinale geografico
Il dataset finale contiene una osservazione per ogni coppia utente - anno, con le seguenti variabili:
 - userId
 - year
 - geo_entropy
 - n_countries
Il dataset è stato salvato come `user_year_geo_longitudinal.parquet`

Questo file potrà essere utilizzato nelle analisi successive insieme alle metriche già costruite:
• entropia dei generi
• novelty
• metriche di attività degli utenti

# 8. Risultati
La costruzione della metrica geografica permette di estendere l'analisi della diversity decay oltre la dimensione dei generi cinematografici.
La nuova metrica consente di verificare se, con l'aumentare degli anni sulla piattaforma, gli utenti:
 - mantengano una distribuzione ampia dei paesi di produzione
 - concentrino progressivamente i consumi verso poche aree geografiche
 - mostrino un comportamento diverso rispetto alla diversità dei generi

L'entropia geografica verrà integrata nel notebook dei modelli finali per confrontare tre dimensioni della diversità:
 • diversità dei generi
 • capacità di esplorazione di nuovi contenuti
 • diversità geografica dei consumi

 # Stato attuale della pipeline
• Recupero informazioni geografiche tramite IMDb → TMDb
• Costruzione mapping film–paese
• Verifica della copertura geografica
• Merge con il dataset dei rating
• Costruzione distribuzione utente–anno–paese
• Calcolo Shannon Entropy geografica
• Salvataggio dataset longitudinale (`user_year_geo_longitudinal.parquet`)

La pipeline contiene ora tutte e tre le dimensioni principali della diversity analysis previste dal progetto:
 - genre diversity
 - novelty
 - geographic diversity
---

# Notebook 04_robustness - Analisi di robustezza e Novelty Granulare
Questo notebook verifica la robustezza delle metriche utilizzate nelle analisi longitudinali e introduce una nuova misura di Novelty basata sui MovieLens Genome Tags, come alternativa più granulare alla rappresentazione tramite soli 19 macro-generi.
Il notebook persegue due obiettivi principali:
1. costruire una misura di Novelty più granulare, in grado di rappresentare la continua esplorazione dei
   contenuti anche dopo la saturazione dei macro-generi
2. verificare se questa maggiore granularità attenui l'effetto soffitto osservato in `02_EDA`
3. valutare la robustezza dei risultati principali (diversità dei generi, novelty) rispetto alla scelta
   della metrica, della soglia di rilevanza e alla presenza di rotture strutturali nel tempo

# 0. Setup
Eseguire pip install -r requirements.txt per installare le dipendenze 

# 1. Analisi preliminare dei Genome Tags
MovieLens 25M mette a disposizione il dataset Genome, costituito da oltre 1.100 tag descrittivi associati ai film mediante un punteggio di rilevanza (relevance score).
Nel notebook vengono effettuati alcuni controlli preliminari:
 - numero totale di Genome Tags disponibili
 - numero di film coperti dal dataset Genome
 - distribuzione dei punteggi di rilevanza
 - copertura del catalogo rispetto ai film presenti e ai rating presenti nel dataset

L'analisi mostra che:
 • sono disponibili 1128 Genome Tags
 • solo 13.816 film su 59047 (23.4%) dispongono di Genome Scores
 • nonostante questo, la copertura a livello di rating risulta superiore al 98%: : i film con Genome Tags sono quindi sistematicamente i più popolari/valutati del catalogo, rendendo il dataset idoneo alle analisi longitudinali

NB: il forte scarto tra copertura a livello di film (23.4%) e a livello di rating (98.7%)
introduce un bias sistematico verso i consumi mainstream: ogni conclusione basata sui Genome Tags si applica
a un sotto-campione popolare del catalogo ed esclude sistematicamente i film di nicchia. Questo limite va
tenuto presente in ogni interpretazione della metrica di novelty granulare descritta più avanti.

# 2. Scelta della soglia di rilevanza
Poiché ogni film è associato a centinaia di Genome Tags con differenti livelli di importanza, viene effettuata un'analisi di sensibilità sulla soglia di rilevanza.
Sono confrontate tre possibili soglie: 0.30, 0.50, 0.70
Per ciascuna soglia viene calcolato il numero medio di tag associati a ogni film.

L'analisi evidenzia che:
 • soglie troppo basse mantengono numerosi tag poco informativi
 • soglie troppo elevate eliminano una quota consistente dell'informazione disponibile
 • la soglia 0.50 rappresenta il miglior compromesso tra copertura e qualità descrittiva.
Di conseguenza, tutte le analisi successive utilizzano: relevance ≥ 0.50.
La soglia 0.70 (~18 tag/film) e una variante top-k (10 tag più rilevanti per film) vengono poi riutilizzate nella sezione di robustezza per verificare la stabilità dei risultati.

# 3. Costruzione della Novelty granulare 
Per ciascun film vengono selezionati solamente i Genome Tags con rilevanza almeno pari a 0.50.
Successivamente:
 - ogni film viene rappresentato come un insieme di Genome Tags
 - per ogni utente e anno vengono aggregati tutti i tag osservati
 - viene calcolata la quota di tag realmente nuovi rispetto agli anni precedenti
La nuova misura permette quindi di stimare la scoperta di nuovi concetti cinematografici, anziché la semplice scoperta di nuovi macro-generi.

# 4. Dataset finale
Il notebook produce il file: `user_year_genome_longitudinal.parquet` contenente, per ogni coppia utente–anno, le principali metriche della nuova Novelty:
 - tag_novelty
 - n_tags
Questo dataset viene poi integrato con:
 - user_year_genre_longitudinal.parquet
 - user_year_geo_longitudinal.parquet
nel notebook successivo dedicato ai modelli panel.

# 5. Analisi di robustezza
A completamento del notebook vengono eseguite quattro verifiche di robustezza sui risultati principali
delle analisi longitudinali:
1) Novelty classica (19 generi) vs Novelty granulare (Genome Tags): le due metriche vengono confrontate
   tramite la stessa regressione a effetti fissi (demeaning per utente, errori standard clustered, controllo per anno di calendario) usata in `02_EDA`. 
   Risultato: il coefficiente di declino della novelty è quasi doppio sui Genome Tags (-0.0046) rispetto ai generi (-0.0022). La causa è meccanica: con ~44 tag medi per film, un utente copre in media il 61.2% del vocabolario di 1128 tag già in un solo anno di consumo normale per cui la saturazione avviene più rapidamente, non più lentamente, che con soli 19 generi.

2) Sensitivity sulla soglia di rilevanza e sul vocabolario: ripetendo la stessa regressione con soglia 0.70
   e con un top-10 tag per film, il coefficiente di declino non solo resta negativo ma **si rafforza** ulteriormente: -0.0046 → -0.0071 (soglia 0.7) → -0.0142 (top-10). Il pattern di saturazione regge quindi anche con un vocabolario più selettivo, confermando che l'effetto non dipende dalla granularità scelta.

3) Shannon Entropy vs Simpson Diversity: l'indice di Simpson viene calcolato sulla stessa distribuzione di generi 
   usata per l'entropia di Shannon (correlazione tra le due metriche: r = 0.943). Il coefficiente di crescita della diversità con l'anzianità sulla piattaforma è positivo e significativo per entrambe le metriche (Shannon: 0.0063; Simpson: 0.0006), confermando che il risultato principale sulla diversità dei generi è robusto alla scelta della metrica.

4) Change-point detection: viene applicato l'algoritmo PELT alla traiettoria annua dell'entropia media, con tre
   livelli di penalizzazione (1, 3, 10). Con penalizzazioni conservative (3 e 10) non emerge alcun break-point
   interno; con la penalizzazione più bassa (1) emerge un possibile break nel 1999, verosimilmente legato alla
   scarsità di dati nei primissimi anni del campione. La crescita della diversità nel tempo resta quindi meglio
   descritta come un trend graduale piuttosto che come una rottura netta legata a un evento specifico.

# Motivazioni
L'ipotesi di partenza era che una tassonomia più granulare (i Genome Tags, oltre 1100 descrittori semantici) potesse attenuare questo effetto, rivelando un'esplorazione più fine mascherata dalla scarsità dei 19 generi.
Questa ipotesi non è confermata dai dati: l'analisi di robustezza (5.1, 5.2) mostra che la novelty
granulare satura ancora più rapidamente di quella sui macro-generi, e che l'effetto si rafforza (non si
attenua) restringendo ulteriormente il vocabolario. 
Si tratta quindi di un risultato contro-intuitivo ma robusto: l'aumento della granularità della tassonomia non elimina l'effetto di saturazione osservato con i gener, che sembra dipendere anche dalla struttura cumulativa della progressiva copertura delle caratteristiche già osservate dagli utenti.

# Stato attuale della pipeline
• Caricamento del dataset MovieLens Genome
• Analisi esplorativa dei Genome Tags
• Valutazione della copertura del catalogo MovieLens (a livello di film e di rating, con verifica del bias verso i consumi mainstream)
• Analisi della distribuzione dei relevance scores
• Sensitivity analysis sulla soglia di rilevanza (0.30, 0.50, 0.70. top-10)
• Selezione della soglia ottimale (relevance ≥ 0.50)
• Costruzione del mapping film–Genome Tags
• Costruzione della rappresentazione utente–anno tramite Genome Tags
• Calcolo della Novelty granulare basata sui Genome Tags
• Salvataggio del dataset longitudinale (`user_year_genome_longitudinal.parquet`)
• Confronto Novelty classica vs granulare tramite regressione a effetti fissi
• Sensitivity sulla soglia di rilevanza e sul numero di tag per film
• Confronto Shannon Entropy vs Simpson Diversity sulla diversità dei generi
• Change-point detection (PELT) sulla traiettoria temporale della diversità
• Sintesi finale dei risultati robusti vs fragili

La pipeline comprende ora tutte le metriche necessarie per la costruzione dei modelli panel finali:
- Genre Diversity (Shannon Entropy,  verificata anche con Simpson Diversity)
- Genre Novelty (macro-generi)
- Geographic Diversity
- Genome Tag Novelty (metrica granulare, con relativo limite di copertura documentato)
---

# Notebook 05 – Modelli panel finali
Questo notebook rappresenta la fase conclusiva dell'analisi quantitativa. Integra tutte le metriche longitudinali costruite nei notebook precedenti (`01_preproc`, `03_geographic_diversity`, `04_robustness`) in un unico panel utente–anno, stima i modelli a effetti fissi (FE) definitivi per ciascuna dimensione della diversity, confronta la loro evoluzione temporale e ne propone
un'interpretazione congiunta rispetto alla domanda di ricerca.

Le quattro metriche considerate:
- Genre Diversity (Shannon Entropy sui 19 macro-generi)
- Genre Novelty (quota di generi nuovi per anno)
- Geographic Diversity (Shannon Entropy sui paesi di produzione)
- Genome Tag Novelty (metrica granulare, con i limiti di copertura discussi in `04_robustness`)

# 1. Costruzione del panel integrato
Il panel viene costruito unendo il campione stratificato di 10.000 utenti (`sample_users_10k.parquet`, già filtrato e con `active_year`/`log_ratings` pronti) con i dataset `user_year_geo_longitudinal` e `user_year_genome_longitudinal`, tramite left join su `[userId, year]`.

Il tasso di match è risultato praticamente completo: 100.0% per la Geographic Diversity e 100.0% per la Genome Tag Novelty (solo 2 righe su 35.624 senza `geo_entropy`). Nonostante il limite di copertura a livello di catalogo documentato in `04_robustness` (solo il 23.4% dei film ha Genome Tags), a livello di utente-anno risulta comunque disponibile una misura in quasi tutti i casi — coerente con l'alta copertura osservata sui rating (98.7%). I valori mancanti non vengono riempiti con zero, per non introdurre
diversità artificialmente nulle: il panel integrato (35.624 righe, 37 colonne) viene salvato in `panel_finale.parquet`.

# 2. Relazione tra le dimensioni della diversity
Prima di stimare i modelli, viene calcolata la correlazione tra le quattro metriche a livello utente-anno. 
Genre Diversity e Geographic Diversity risultano moderatamente correlate con entrambe le metriche di novelty (r ≈ 0.27-0.29): dimensioni collegate ma distinte. 
Il risultato più rilevante è che Genre Novelty e Genome Tag Novelty sono correlate a r = 0.84, quasi coincidenti. Questo conferma che la novelty granulare sui Genome Tags introduce poca informazione realmente nuova rispetto a quella sui 19 generi, ma non nulla.

# 3. Modelli a effetti fissi (FE)
Per ciascuna metrica viene stimato lo stesso modello a effetti fissi utente (demeaning per utente + errori standard clustered per utente + controllo per anno di calendario) già usato in `02_EDA` e `04_robustness`.
Entropy e Geographic Diversity vengono stimate sul panel completo; Genre Novelty e Genome Tag Novelty solo su `active_year >= 2`, per evitare l'artefatto meccanico del primo anno (novelty ~100% per costruzione).

Tutti e quattro i coefficienti sono altamente significativi: Diversity (generi e paesi) aumenta con l'anzianità sulla piattaforma, novelty (generi e tag) diminuisce (coerentemente con l'effetto soffitto documentato in `02_EDA` e `04_robustness`).

# 4. Robustezza: modello a effetti misti
Come verifica metodologica aggiuntiva, viene stimato un modello mixed-effects con intercetta casuale per utente sulla
Genre Diversity. Il coefficiente su `active_year` (0.0060) è concorde per segno e ordine di grandezza con quello del modello FE via demeaning (0.0065): il risultato principale sulla diversità dei generi è quindi robusto anche alla scelta della tecnica di stima.

# 5. Significatività statistica vs rilevanza pratica
Con decine di migliaia di osservazioni, coefficienti anche molto piccoli risultano statisticamente significativi. Per valutarne la rilevanza pratica, la variazione attesa su un orizzonte di 10 anni di attività viene espressa in deviazioni standard della metrica stessa.

Solo la Geographic Diversity supera la soglia convenzionale di 0.2: è l'unico effetto abbastanza ampio da essere potenzialmente percepibile anche a livello di singolo utente, non solo in aggregato su un campione di questa dimensione. Gli altri tre effetti sono statisticamente solidi ma di rilevanza pratica modesta anche cumulata su un decennio.

# 6. Interpretazione congiunta
Guardando i quattro modelli insieme, e non uno alla volta, emerge un quadro più sfumato di un semplice "sì" o "no" alla domanda di ricerca:
- Diversity (generi e paesi) e Novelty (generi e tag) si muovono in direzioni opposte nel tempo.
  Non sono risultati contraddittori: un utente può continuare a mescolare in modo sempre più equilibrato ciò che già conosce (diversity in aumento), pur smettendo progressivamente di scoprire categorie mai viste prima (novelty in calo).
- La domanda "gli algoritmi ci restringono?" ha quindi una risposta diversa a seconda della nozione di diversità considerata: la 
  varietà del consumo corrente non sembra restringersi — anzi aumenta — ma la propensione a scoprire contenuti realmente nuovi sì.
- Tra le quattro dimensioni, la Geographic Diversity è l'unica con un effetto anche praticamente rilevante, le altre tre, pur solide 
  statisticamente, restano di magnitudine contenuta.

Limiti importanti:
- La Genome Tag Novelty eredita il bias verso i consumi mainstream discusso in `04_robustness` (Genome Tags disponibili solo per il 
  23.4% dei film, pur coprendo il 98.7% dei rating).
- Nessuna rottura strutturale è stata rilevata nella traiettoria della diversità dei generi (`04_robustness`): l'evoluzione osservata 
  è un trend graduale, non un salto legato a un evento specifico, e i dati MovieLens non permettono di distinguere causalmente un effetto "algoritmo" da una naturale evoluzione delle preferenze dell'utente nel tempo.
- Le correlazioni tra le quattro metriche non implicano un meccanismo causale condiviso, e la correlazione forte  le due misure di 
  novelty (r = 0.84) va tenuta presente nell'interpretare i risultati come evidenza indipendente.

# Stato della pipeline
• Panel integrato utente–anno costruito e salvato (`panel_finale.parquet`)
• Correlazione tra le quattro dimensioni di diversity analizzata
• Traiettorie descrittive per anno di attività
• 4 modelli a effetti fissi stimati e confrontati (tabella + forest plot)
• Verifica di robustezza tramite modello a effetti misti
• Effect size pratico calcolato e interpretato
• Interpretazione congiunta dei risultati completata

A questo punto la pipeline quantitativa è completa. Il prossimo notebook (`06_figures`) riprenderà le figure prodotte qui per la versione finale da inserire in tesi.
---

# Notebook 06_figures – Figure finali per la tesi
Questo notebook non introduce nuove analisi statistiche: riprende il panel integrato costruito in `05_model` (`panel_finale.parquet`) e ne produce le **figure finali ad alta risoluzione** (300 dpi) da inserire in tesi. 
Copre i tre deliverable previsti dal piano: heatmap coorte finali, scatter anzianità-vs-diversità, e tasso di rinnovamento aggregato.

# 1. Heatmap coorte × anno di attività
Per ogni coorte (utenti raggruppati per anno di primo utilizzo, `cohort_year = min(year)` per utente) si calcola la diversità media per ciascun anno di attività, escludendo le celle con meno di 30 osservazioni per evitare medie poco affidabili.

Genre Diversity
La prima colonna (`active_year = 1`) è sistematicamente la più scura (H ≈ 3.50) per quasi tutte le coorti, seguita da un calo netto al secondo anno e da una risalita graduale fino a un massimo intorno all'anno 8-12. 
Non è in contraddizione con il coefficiente positivo trovato nella regressione FE di `05_model` (+0.0065 su `active_year`): il picco isolato al primo anno è un artefatto tipico dei dataset MovieLens, dove molti utenti, appena iscritti, valutano in blocco un ampio numero di film già visti in passato ("rating dump" iniziale, eterogeneo per costruzione). Il coefficiente positivo
della FE riflette la crescita genuina osservata a partire dal secondo anno, quando il comportamento si stabilizza su un consumo consolidato

Geographic Diversity
Qui il pattern è più diretto: il colore si scurisce (H più alta) in modo piuttosto lineare al crescere dell'anno di attività, senza il picco iniziale osservato per i generi, coerente con il coefficiente FE più marcato trovato per questa metrica (+0.0213).

# 2. Scatter: anzianità sulla piattaforma vs diversità media (per utente)
Confronta, questa volta tra utenti diversi (non nel tempo per lo stesso utente), il numero totale di anni attivi (`tenure`) con la Genre Diversity media di ciascuno. La relazione è debolmente positiva (r = 0.031), sostanzialmente piatta: coerente con l'effect size già identificato come piccolo in `05_model` (0.16 deviazioni standard su un orizzonte di 10 anni). Va tenuto distinto dal risultato della regressione FE: qui si osserva se chi resta di più ha, in media, più diversità (un confronto tra utenti, soggetto a
possibili effetti di selezione), non se lo stesso utente diversifica nel tempo.

# 3. Tasso di rinnovamento aggregato
Calcolato come media di `novelty_pct` sulla popolazione campionata (escluso il primo anno di attività di ciascun utente), sia nel tempo che per livello di attività:
- Nel tempo: il tasso scende bruscamente dal 1997 al 2003 circa, poi si stabilizza su valori bassi (2-3%) con lieve tendenza al 
  rialzo dal 2010 in poi — coerente con il calo di novelty osservato nella regressione FE, seppur non perfettamente monotono anno per anno.
- Per livello di attività: pattern netto e monotono — gli utenti "Low" rinnovano molto di più (≈8%) di "Medium" (≈3%) e "High" (≈1%). 
  Questo non implica necessariamente che gli utenti più attivi siano meno curiosi: guardando più film per anno, esauriscono meccanicamente prima i 19 generi disponibili (effetto soffitto), il che è coerente con il coefficiente positivo di `log_ratings` osservato in tutti i modelli FE di `05_model`.
Il tasso di rinnovamento aggregato complessivo, su tutto il periodo, è pari al 2.7%.

# 4. Figure prodotte
Tutte salvate in `figures/` a 300 dpi:
- `heatmap_cohort_entropy.png`
- `heatmap_cohort_geo_entropy.png`
- `scatter_tenure_vs_entropy.png`
- `renewal_rate_by_year.png`
- `renewal_rate_by_activity_group.png`
(a cui si aggiungono, da `05_model`: `correlation_diversity_metrics.png`, `trajectories_active_year.png`,
`forest_plot_active_year.png`)

# Conclusioni e chiusura della pipeline quantitativa
Le figure di questo notebook non aggiungono nuovi risultati rispetto a `05_model`, ma li rendono verificabili a colpo d'occhio, incluso un pattern che a prima vista potrebbe sembrare in contraddizione con la regressione (il picco di diversità al primo anno di attività).

Sintesi complessiva della domanda di ricerca
"Gli utenti guardano film sempre meno diversificati nel tempo?"
La risposta, sulla base dell'intera pipeline (`01_preproc` → `06_figures`), è no per la diversità del consumo corrente** (che anzi aumenta, sia per generi, al netto del picco iniziale, sia per paesi), mentre è sì per la propensione a scoprire contenuti realmente nuovi (che diminuisce, con effetto soffitto confermato robusto su più metriche, soglie e tecniche di stima in `04_robustness` e
`05_model`).
Questa distinzione tra mescolare ciò che si conosce ed esplorare ciò che non si conosce è il contributo interpretativo principale del lavoro.

Limiti importanti:
- bias di copertura dei Genome Tags verso i consumi mainstream (`04_robustness`)
- assenza di identificazione causale di un effetto "algoritmo" distinto da una naturale evoluzione delle preferenze (`04_robustness`,
  `05_model`)
- il picco di diversità al primo anno di attività come possibile artefatto di comportamento di consumo all'iscrizione, non come vero effetto temporale