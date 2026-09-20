# Marketing Mix Modeling su Databricks

Progetto personale di Marketing Mix Modeling (MMM): dalla creazione del dataset all'ingestion su Databricks (Unity Catalog), fino a un modello di regressione con adstock, saturation, decomposizione per canale e un MMM Optimizer per la riallocazione del budget.

## Obiettivo

Stimare quanto ciascun canale media contribuisce alle vendite, tenendo conto di due fenomeni chiave del marketing mix modeling — la persistenza dell'effetto pubblicitario nel tempo (adstock) e i rendimenti decrescenti della spesa (saturation) — e usare il modello risultante per suggerire come riallocare il budget tra i canali.

## Dataset

`mmm_dataset.csv` — 104 settimane di dati sintetici (gennaio 2023 – dicembre 2024): spesa su quattro canali media (TV, digital, social, search), un flag di promozione attiva, un indice di prezzo relativo, un flag per i periodi di festività (Black Friday, Natale) e la variabile target `sales` (vendite settimanali in €). I dati sono generati artificialmente ma con una struttura realistica (rumore, stagionalità, trend, confondenti), pensata per riprodurre le sfide tipiche di un dataset MMM reale.

## Metodologia

**Adstock.** Modella quanto dura nel tempo l'effetto di un euro spesato oggi, con un decadimento specifico per canale: più lento per la TV (decay 0.55, coerente con un media above-the-line la cui memoria dura settimane), più rapido per il search (decay 0.15, effetto quasi immediato legato all'intenzione di ricerca del momento).

**Saturation.** Modella i rendimenti decrescenti tramite una curva di Hill: raddoppiare la spesa su un canale non raddoppia le vendite, perché a un certo punto il mercato raggiungibile è già saturo.

**Regressione.** Un modello Ridge stima il contributo di ciascun canale (dopo adstock e saturation) alle vendite, controllando per stagionalità, trend, promozioni, prezzo e festività — necessario per non attribuire ai canali media un effetto che in realtà viene da altri fattori.

**Decomposizione e ROI.** I coefficienti stimati vengono usati per scomporre le vendite settimana per settimana e calcolare il ROI storico di ciascun canale.

**MMM Optimizer.** Dato un budget fisso, un'ottimizzazione numerica (SLSQP) suggerisce come riallocarlo tra i canali per massimizzare le vendite previste, sfruttando le curve di saturazione stimate.

## Risultati principali

Il modello Ridge raggiunge un **R² di 0.895** sul training set. Il ROI storico stimato per canale varia molto: la **TV ha il ROI più alto (0.59x)**, seguita da search (0.45x) e social (0.30x); il **digital ha il ROI più basso (0.16x)**, nonostante sia il canale con la spesa totale maggiore (~600k€ su 104 settimane).

Coerentemente con questo gap, l'MMM Optimizer suggerisce — a parità di budget settimanale (~16.700 €) — di spostare risorse dal digital (da 5.770 € a 504 €) verso la TV (da 4.432 € a 9.811 €), con un **uplift stimato del 16.6%** sulle vendite incrementali a parità di spesa totale.

## Struttura del repository

```
├── mmm_data.ipynb       # notebook Databricks (dataset → adstock/saturation → regressione → ROI → optimizer)
├── mmm_dataset.csv       # dataset sintetico usato come input
└── README.md
```

## Come riprodurlo

Il notebook è pensato per Databricks (usa `spark.read.csv` per l'ingestion da un volume Unity Catalog), ma la logica di trasformazione e modellazione è pandas/scikit-learn puro e gira ovunque: basta caricare `mmm_dataset.csv` con `pandas.read_csv` al posto della cella di lettura Spark.

## Limiti e possibili sviluppi

I parametri di adstock (decay) e saturation (alpha, gamma) sono impostati manualmente in base a ipotesi ragionevoli per canale, non stimati dai dati — un possibile sviluppo è ottimizzarli con una grid search o un approccio bayesiano (es. PyMC-Marketing), che permetterebbe anche di quantificare l'incertezza delle stime. I dati sono sintetici: su un dataset aziendale reale andrebbero validati ulteriori confondenti (prezzo dei competitor, eventi esterni, cambi di prodotto).
