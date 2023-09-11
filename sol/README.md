# Laboratorio 12
In questo laboratorio viene richiesto di sviluppare un programma più complesso
che richiede l'utilizzo di molti dei concetti discussi durante il corso.
La struttura del laboratorio segue quella di una prova d'esame.


# Sistema idraulico
Scrivere un software per la descrizione e simulazione di un sistema idraulico.
I moduli e le classi vanno sviluppati nel package *hydraulics*.
Non spostare o rinominare le classi esistenti e non modificare le signature dei metodi.

In *main.py* viene fornito del semplice codice, da voi modificabile, che testa le funzionalità base.
Questo verrà fornito all'esame.

Nella package *tests* vengono forniti dei test simili a quelli che valuteranno la vostra prova d'esame.
Questi non verranno forniti all'esame ma saranno disponibili solamente per la correzione a casa.


## R1: Sistema ed elementi
Un sistema idraulico è composto da elementi di vario tipo connessi tra loro.
Un sistema idraulico è rappresentato da un oggetto di classe *HSystem*,
mentre gli elementi dalla classe base *Element*.

È possibile aggiungere elementi al sistema tramite il metodo ```add_element(self, elm: Element) -> None``` di *HSystem*,
il quale riceve come parametro un oggetto di tipo *Element*
e lo aggiunge ai componenti che formano il sistema idraulico.

La property ```elements(self) -> List[Element]```
permette di ottenere una lista contenente gli elementi presenti nel sistema.

Tutti gli elementi hanno un nome che può essere letto tramite la property ```name(self) -> str```.

**NOTA**: Non è possibile creare oggetti *Element* essendo la classe astratta.
Il type hint *Element* indica oggetti che si comportano come *Element* (hanno stessi metodi e property).


## R2: Elementi semplici
Sono definiti tre tipi di elementi semplici: sorgente, rubinetto e scarico,
che sono rispettivamente rappresentati dalle classi *Source*, *Tap* e *Sink* che ereditano da *Element*.

È possibile connettere l'uscita di un elemento all'ingresso di un altro
tramite il metodo ```connect(self, elm: "Element") -> None```
Il metodo riceve come parametro l'elemento al cui ingresso deve essere connessa l'uscita dell'elemento sui cui è invocato.
Ad esempio, *a.connect(b)* connette l'uscita di *a* all'ingresso di *b*.
Il metodo, se invocato su un oggetto *Sink*, non ha nessun effetto.

Dato un elemento semplice qualunque, è possibile sapere a quale altro elemento è connessa la sua uscita,
tramite la property ```output(self) -> Optional["Element"]``` che restituisce un oggetto di tipo *Element*.
Se nessun elemento è collegato alla sua uscita restituisce *None* 
(il type hint *Optional* serve proprio a indicare la possibilità di avere *None* come valore di ritorno).

**ATTENZIONE:** l'elemento restituito dalla property ```output``` deve essere l'oggetto originale, non una copia.


## R3: Elementi complessi
Oltre agli elementi semplici, sopra descritti, è possibile utilizzare degli elementi complessi.
L'elemento a T, rappresentato dalla classe *Split*,
permette di suddividere l'ingresso in due flussi in uscita uguali tra loro.

Per tale classe il metodo  ```connect_at(self, elm: Element, pos: int) -> None``` riceve un ulteriore parametro,
di tipo intero, che indica il numero dell'uscita a cui connettere l'elemento.
Tale intero ha valore *0* per la prima uscita e *1* per la seconda.

Per sapere quali elementi sono connessi in uscita a un elemento a T,
è possibile utilizzare la property ```outputs(self) -> List[Optional[Element]]```,
che restituisce una lista con i due elementi connessi.
Inserire ```None``` per le uscite non connesse.

**ATTENZIONE:** gli elementi restituiti dalla property ```outputs``` devono essere gli oggetto originali, non una copia.


## R4: Simulazione
Dato un sistema **CORRETTO**, ovvero un albero che ha come radice una sorgente
e in cui ogni percorso termina con uno scarico,
è possibile fare un calcolo delle portate e di come vengono ripartite nei vari elementi.

Il calcolo prevede due fasi: una prima fase (*setup*)
i cui si definiscono i parametri dei diversi elementi del sistema 
e una seconda fase (*simulazione*) in cui si avvia la simulazione.

Durante la fase si setup è possibile configurare il comportamento degli elementi:

- definire la portata per una sorgente (*Source*) tramite il property setter ```flow(self, flow: float) -> None```,
che riceve come parametro un numero reale che rappresenta i metri cubi al secondo erogati dalla sorgente.
La portata è ottenibile tramite la property ```flow(self) -> float```.

- impostare l'apertura dei rubinetti (*Tap*), tramite il property setter
```status(self, to_open: bool = True) -> None```, che riceve come parametro un boolean.
Lo stato di un rubinetto è ottenibile tramite la property ```status(self) -> bool```.
Se un rubinetto è aperto la portata in uscita è uguale a quell in ingresso,
altrimenti la portata in uscita è pari a *0*.

- Per i raccordi a T la portata in ingresso viene sempre ripartita equamente tra le due uscite.

Il metodo ```simulate(self) -> List[str]``` della classe HSystem effettua i calcoli di portata a partire da ogni sorgente e salva,
per ogni elemento: il nome e le portate in ingresso e in uscita.

Quando, durante la simulazione, sono noti i flussi in entrata e in uscita per un elemento,
deve essere creata un stringa contente il tipo di elemento (nome della classe),
il nome dell'elemento, il flusso in ingresso e i flussi in uscita (2 nel caso dello Split).
Il tutto deve essere separato da uno spazio, e i valori scritti su 3 cifre decimali e.g.,
*"Split Split_1 11.500 5.750 5.750"*

Il flussi in ingresso alle sorgenti e il flussi in uscita ai *Sink* devono essere pari a *0*.

Il metodo restituisce una lista contenente tutte queste stringhe.

**ATTENZIONE**: per la simulazione considerare che tutti gli ingressi e tutte le uscite siano connessi,
quando l'elemento lo permette.


## R5: MultiSplit
La classe *MultiSplit* eredita dalla classe *Split*, e prevede un numero variabile di uscite.
Pertanto il costruttore accetta, oltre al nome, il numero di uscite.

Il metodo ```connect_at(self, elm: Element, pos: int) -> None```, come per la classe *Split*,
permette di connettere un elemento, fornito come primo parametro, all'uscita il cui numero è fornito come secondo parametro.
Le uscite sono numerate a partire da zero.

La property ```outputs(self) -> List[Optional[Element]]```, restituisce una lista contenente gli elementi connessi.
Inserire ```None``` per le uscite non connesse.

La property ```proportions(self) -> List[Optional[float]]``` restituisce una lista contenente le proporzioni
di portata che vengono ripartite su ogni uscita connessa del *MultiSplit*.

Il property setter ```proportions(self, proportions: List[Optional[float]]) -> None```
permette d'impostare la lista di proporzioni.

Fare le opportune modifiche di modo che il metodo ```simulate(self) -> List[str]``` della classe *HSystem*
possa effettuare la simulazione di sistemi idraulici contenenti *MultiSplit*.
Il formato della stringa di simulazione è il medesimo, variando adeguatamente il numero di flussi in uscita.

**IMPORTANTE**: Si assuma che la somma delle proporzioni sia sempre pari a uno.
Anche per questi elementi considerare che tutte le uscite siano sempre connesse quando si simula il sistema.




