# MovieLens Diversity Decay 

## Descrizione:
Analisi longitudinale della diversità e della novelty dei consumi cinematografici degli utenti nel dataset MovieLens-25M.

Il progetto studia come evolvono nel tempo la varietà dei contenuti consumati e l'introduzione di nuove caratteristiche nella storia individuale degli utenti, con particolare attenzione all'ipotesi di diversity decay nel contesto dei sistemi di raccomandazione.

L'analisi non stima un effetto causale dei sistemi di raccomandazione, poiché il dataset MovieLens non contiene informazioni sulle raccomandazioni effettivamente mostrate agli utenti. I risultati descrivono quindi le traiettorie di consumo osservate nel dataset e ne valutano la compatibilità con l'ipotesi di una progressiva riduzione della diversità.


## Obiettivo del progetto
L’obiettivo generale è analizzare come evolvono nel tempo la diversità e la novelty dei consumi cinematografici osservati su MovieLens, e valutare se le dinamiche osservate siano compatibili con l'ipotesi di diversity decay discussa nel contesto dei sistemi di raccomandazione.
In particolare, il progetto analizza:
- l'evoluzione della diversità dei generi nel tempo
- l'evoluzione della diversità geografica dei film consumati
- la variazione della novelty dei generi
- la variazione della novelty basata sui Genome Tags
- la sensibilità dei risultati rispetto a diverse metriche e scelte di misurazione
- la presenza di possibili fenomeni di saturazione delle misure di novelty


## Dataset utilizzato
Il progetto utilizza il dataset MovieLens-25M, sviluppato da GroupLens Research presso la University of Minnesota.
Il dataset contiene:
- 25.000.095 valutazioni
- 162.541 utenti
- 62.423 film
- valutazioni comprese tra 0.5 e 5
- informazioni sui generi cinematografici
- collegamenti a IMDb e TMDb
- Genome Scores e Genome Tags

Il dataset originale non è incluso nel repository.
Per utilizzare il progetto è necessario:
1. scaricare il dataset da: https://grouplens.org/datasets/movielens/25m/
2. estrarre il file .zip scaricato
3. creare manualmente le cartelle data/raw/ e data/processed/
4. posizionare la cartella ml-25m dentro: data/raw/
5. eseguire i singoli notebook in ordine 


## Setup ambiente di lavoro su nuova macchina
Per eseguire correttamente il progetto su un altro computer è necessario configurare l’ambiente Python e scaricare il dataset richiesto.
1. creare ambiente Python
```bash
conda create -n movielens python=3.10
conda activate movielens
```

2. installare dipendenze
```bash
pip install -r requirements.txt
```

3. registrazione kernel Jupyter (per garantire la corretta esecuzione nei notebook)
```bash
python -m ipykernel install --user --name movielens --display-name "Python (movielens)"
```

I notebook devono essere eseguiti utilizzando il kernel Python dell’ambiente Conda `movielens`. 


## Verifica ambiente Python nei notebook
Per controllare l'ambiente attivo:
```python
import sys
print(sys.executable)
```

l'output deve mostrare il percorso dell'interprete Python associato all'ambiente Conda movielens, se diverso selezionare il kernel "Python (movielens).


## Struttura del progetto
```
Tirocinio_MovieLens/
│
├── data/ → contiene i dati grezzi e i file elaborati
│   ├── raw/
│   │   └── ml-25m/
│   └── processed/
│
├── figures/ → contiene le figure generate durante l'analisi
│
├── notebooks/ → contiene l'intera pipeline di analisi, dalla preparazione dei dati alla produzione delle figure finali
│   ├── 00_setup.ipynb
│   ├── 01_preproc.ipynb
│   ├── 02_EDA.ipynb
│   ├── 03_geographic_diversity.ipynb
│   ├── 04_robustness.ipynb
│   ├── 05_model.ipynb
│   └── 06_figures.ipynb
│
├── EvoluzioneProgetto.md → documenta evoluzione del progetto, principali scelte metodologiche, risultati 
├── README.md
└── requirements.txt
```


## Pipeline di analisi
I notebook sono organizzati secondo una sequenza logica.

- 00_setup.ipynb: configura l'ambiente di lavoro e definisce i percorsi principali del progetto
- 01_preproc.ipynb: prepara i dati MovieLens e costruisce le strutture necessarie per l'analisi longitudinale
  Le principali operazioni comprendono:
  • caricamento dei dati
  • integrazione delle informazioni relative a rating, film e generi
  • trasformazione dei timestamp in anni
  • riorganizzazione dei generi
  • costruzione delle informazioni a livello utente-anno
  • applicazione dei criteri di selezione del campione

- 02_EDA.ipynb: realizza l'analisi esplorativa dei dati
  Vengono analizzati:
  • distribuzione dell'attività degli utenti
  • evoluzione della diversità dei generi
  • evoluzione della novelty
  • andamento delle principali variabili nel tempo
  • relazioni preliminari tra le misure

- 03_geographic_diversity.ipynb: integra le informazioni sulla provenienza geografica dei film e analizza la diversità dei consumi rispetto ai paesi di produzione
  Vengono costruite misure come:
  • geographic entropy
  • numero di paesi distinti osservati

- 04_robustness.ipynb: analizza la robustezza dei principali risultati rispetto a diverse scelte metodologiche.
  Le verifiche includono:
  • confronto tra Shannon e Simpson
  • confronto tra generi e Genome Tags
  • diverse soglie di relevance
  • selezione dei Top-K Genome Tags
  • analisi di change-point
  Particolare attenzione è dedicata alla possibile saturazione delle misure di novelty.

- 05_model.ipynb: stima i modelli longitudinali a effetti fissi per utente.
  Le principali variabili dipendenti considerate sono:
  • Genre Diversity
  • Geographic Diversity
  • Genre Novelty
  • Genome Tag Novelty
  I modelli tengono conto della progressione dell'attività osservata degli utenti e di variabili di attività, includendo effetti fissi individuali e temporali.

- 06_figures.ipynb: genera le principali figure utilizzate per la presentazione dei risultati e per la tesi


## Principali misure
- Diversity
  La diversità dei generi viene misurata principalmente attraverso l'entropia di Shannon, che considera sia il numero di generi osservati sia la distribuzione delle valutazioni tra le categorie.
  Come verifica di robustezza viene utilizzato anche l'indice di Simpson.

- Novelty
  La novelty misura la quota di categorie presenti nell'anno corrente che non erano state osservate nella storia precedente dell'utente.
  La misura viene calcolata sia a livello di genere sia utilizzando i Genome Tags, che permettono una rappresentazione più granulare delle caratteristiche dei film.

- Geographic Diversity
  La diversità geografica viene utilizzata come dimensione complementare alla diversità dei generi e viene misurata attraverso l'entropia della distribuzione delle valutazioni tra i paesi di produzione.


## Risultati principali
Nel campione analizzato, i risultati principali in sintesi sono:
• la diversità dei generi non mostra una progressiva riduzione nel corso dell'attività osservata
• un andamento analogo emerge considerando la diversità geografica
• la novelty diminuisce con il procedere dell'attività osservata
• la diminuzione della novelty è presente sia a livello di generi sia a livello di Genome Tags
• una rappresentazione più granulare attraverso i Genome Tags non elimina il fenomeno di saturazione della novelty
• le analisi di robustezza confermano nel complesso la stabilità dei principali risultati rispetto alle metriche e alle specificazioni considerate

Nel complesso, i risultati distinguono quindi tra varietà dei consumi e introduzione di caratteristiche nuove: la riduzione della novelty non implica necessariamente una riduzione della diversità complessiva.


## Riproducibilità
Il progetto è organizzato per separare:
- dati originali e dati elaborati
- preparazione dei dati
- analisi esplorativa
- analisi di robustezza
- modellazione
- produzione delle figure.

Per una descrizione più dettagliata delle decisioni prese durante lo sviluppo, dei risultati intermedi e delle modifiche apportate alla pipeline, si rimanda al file: EvoluzioneProgetto.md


## Contesto della ricerca
Il progetto costituisce la base empirica per una tesi di laurea triennale sulla possibile evoluzione della diversità dei consumi cinematografici nel tempo.
Il lavoro si inserisce nel dibattito sui sistemi di raccomandazione, sulle filter bubble e sulla possibile omogeneizzazione dei consumi, ma mantiene separata l'evidenza osservata nel dataset dall'eventuale effetto causale degli algoritmi di raccomandazione.