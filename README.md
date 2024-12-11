#  Requirements
Librerie necessarie:
* Numpy
* Mathplotlib
* Networkx
* Ndlib
* PyTorch
* PyTorch geometric

# Project Overview
Il progetto consiste di due fasi principali: nella prima, realizzata nel notebook **advanced-syntetich-graph-notebook.ipynb**, si realizza la costruzione di reti sintetiche che simulano reti reali in cui sono presenti post misinformativi e non misinformativi, mentre nella seconda, realizzata nel notebook **gnn-learning.ipynb**, si realizza il processo di addestramento di varie reti GNN sul dataset creato precedentemente.\
Il notebook **dataset-analisys.ipynb** è stato realizzato per permettere di effettuare l'estrazioni di informazioni sulle proprietà delle reti generate come average degree, average centrality measure, ecc.

# Syntetich graph generation
Le reti sintetiche generate sono tali per cui i nodi rappresentano dei post e l'esistenza di un arco tra due nodi indica il fatto che almeno un certo numero di utenti nella rete ha interagito con i due post su cui l'arco incide.\
I nodi della rete sono caratterizzati da sei feature principali di cui cinque sono le **impressions** del post (Visuals, Shares, Likes,Dislikes, Comments) e una rappresenta uno **score** che sarebbe ottenuto da un sistema automatico in seguito a delle valutazioni sulla correttezza del contenuto del post (più alto e meno è corretto). 
A queste feature si aggiunge una etichetta binaria **Misinformative** che associa ad un nodo (ovvero un post) il valore di misinformativo (1) o non misinformativo (0).\
Anche gli archi della rete presentano due features: **Interactions** che rappresentano il numero di utenti che hanno interagito su entrambi i post e **Weight** che è il peso dell'arco.\
La rete sintetica è generata seguendo un processo in più step:
* crea la topologia di rete usando una combinazione di sottoreti generate tramite small world e preferential attachment, e connesse tra di loro tramite l'aggiunta di archi per garantire la connettività finale del grafo ottenuto;
* assegna le etichette ai post della rete simulando un processo di diffusione tramite modello independent cascade;
* elimina casualmente gli archi dal grafo con probabilità $p$ se l'arco incide tra una coppia di nodi tali che uno sia misinformativo e l'altro no e con probabilità $\frac{p}{2}$ altrimenti;
* se la topologia di rete non è più connessa collega le componenti disconnesse creando un nuovo arco per ogni componente disconnessa con la componente più grande;
* assegna agli archi il numero di interazioni campionando il valore tramite una distribuzione skewed (e.g pareto) che è più o meno skewed in base al fatto che almeno uno dei due nodi sia misinformativo;
* assegna agli archi i pesi calcolati secondo la formula: $$w_{u,v}=\frac{1}{2} (\frac{I_{u,v}}{\sum_{v' \in N(u)}{I_{u,v'}} } + \frac{I_{u,v}}{\sum_{u' \in N(v)}{I_{v,u'}}})$$ 
dove $I_{u,v}$ sono le interazioni associate all'arco $<u,v>$ e $N(x)$ sono i nodi adiacenti al nodo $x$;
* usa le informazioni sugli archi incidenti al nodo $v$ per calcolare le feature sulle impressions sfruttando i tassi di conversione specifici per ogni tipo di impressions.
# Dataset Creation
Si sono realizzati due insiemi di dati (uno usato come training/test set e l'altro come validation set) consistenti in insiemi di grafi generati mediante l'algoritmo visto sopra.
# Learning
Nella fase di learning si è effettuato l'addestramento di quattro modelli GNN:
* GCN
* GAT
* GraphSAGE
* GIN

Si è effettuato, per ogni modello, due fasi di addestramento per 10 epoche andando a considerare una volta anche la feature sui nodi associata agli score di misinformatività ed una volta non considerandola in modo da valutare le prestazioni dei modelli.
# Risultati
I modelli sono stati valutati considerando come metrica la misura F1 e sono riportati nella tabella qui di seguito gli score sul training set.

|   *Model*   | With Score | Without Score | 
|-------------|:----------:|:-------------:|
| **GCN**     |   0.8840   |    0.6416     |
| **GAT**     |   0.6466   |    0.3637     |
|**GraphSage**|   0.9513   |    0.9482     |
| **GIN**     |   0.9003   |    0.8538     |

Da qui vediamo le migliori performance del modello GraphSAGE rispetto a tutti gli altri modelli e un andamento migliore di tutti i modelli rispetto la metrica se si considera anche lo score dei post.
Questi risultati vengono riconfermati anche rispetto a grafi con struttura diversa come nel caso del validation set.

|   *Model*   | With Score | Without Score | 
|-------------|:----------:|:-------------:|
| **GCN**     |   0.9382   |    0.6385     |
| **GAT**     |   0.5646   |    0.2310     |
|**GraphSage**|   0.9770   |    0.9778     |
| **GIN**     |   0.9406   |    0.9112     |
