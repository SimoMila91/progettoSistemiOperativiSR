# progettoSistemiOperativiSR
Libro mastro contenente i dati di transazioni monetarie fra diversi utenti. 

5 Descrizione del progetto: versione minima (voto max 24 su 30)
Si intende simulare un libro mastro contenente i dati di transazioni monetarie fra diversi utenti. A tal fine sono
presenti i seguenti processi:
• un processo master che gestisce la simulazione, la creazione degli altri processi, etc.
• SO_USERS_NUM processi utente che possono inviare denaro ad altri utenti attraverso una transazione
• SO_NODES_NUM processi nodo che elaborano, a pagamento, le transazioni ricevute.

5.1 Le transazioni
Una transazione `e caratterizzata dalle seguenti informazioni:
• timestamp della transazione con risoluzione dei nanosecondi (si veda funzione clock_gettime(...))
• sender (implicito, in quanto `e l’utente che ha generato la transazione)
• receiver, utente destinatario della somma
• quantit`a di denaro inviata
• reward, denaro pagato dal sender al nodo che processa la transazione
La transazione `e inviata dal processo utente che la genera ad uno dei processi nodo, scelto a caso.

5.2 Processi utente
I processi utente sono responsabili della creazione e invio delle transazioni monetarie ai processi nodo. Ad ogni
processo utente `e assegnato un buget iniziale SO_BUDGET_INIT. Durante il proprio ciclo di vita, un processo utente
svolge iterativamente le seguenti operazioni:

1. Calcola il bilancio corrente a partire dal budget iniziale e facendo la somma algebrica delle entrate e delle
uscite registrate nelle transazioni presenti nel libro mastro, sottraendo gli importi delle transazioni spedite ma
non ancora registrate nel libro mastro.
• Se il bilancio `e maggiore o uguale a 2, il processo estrae a caso:
– Un altro processo utente destinatario a cui inviare il denaro
– Un nodo a cui inviare la transazione da processare
– Un valore intero compreso tra 2 e il suo bilancio suddiviso in questo modo:
∗ il reward della transazione pari ad una percentuale SO_REWARD del valore estratto, arrotondato,
con un minimo di 1,
∗ l’importo della transazione sar`a uguale al valore estratto sottratto del reward
Esempio: l’utente ha un bilancio di 100. Estraendo casualmente un numero fra 2 e 100, estrae 50.
Se SO_REWARD `e pari al 20 (ad indicare un reward del 20%) allora con l’esecuzione della transazione l’utente trasferir`a 40 all’utente destinatario, e 10 al nodo che avr`a processato con successo la
transazione.
• Se il bilancio `e minore di 2, allora il processo non invia alcuna transazione

2. Invia al nodo estratto la transazione e attende un intervallo di tempo (in nanosecondi) estratto casualmente
tra SO_MIN_TRANS_GEN_NSEC e massimo SO_MAX_TRANS_GEN_NSEC.
Inoltre, un processo utente deve generare una transazione anche in risposta ad un segnale ricevuto (la scelta del
segnale `e a discrezione degli sviluppatori).
Se un processo non riesce ad inviare alcuna transazione per SO_RETRY volte consecutive, allora termina la sua
esecuzione, notificando il processo master.

5.3 Processi nodo
Ogni processo nodo memorizza privatamente la lista di transazioni ricevute da processare, chiamata transaction
pool, che pu`o contenere al massimo SO_TP_SIZE transazioni, con SO_TP_SIZE > SO_BLOCK_SIZE. Se la transaction
pool del nodo `e piena, allora ogni ulteriore transazione viene scartata e quindi non eseguita. In questo caso, il sender
della transazione scartata deve esserne informato.
Le transazioni sono processate da un nodo in blocchi. Ogni blocco contiene esattamente SO_BLOCK_SIZE transazioni da processare di cui SO_BLOCK_SIZE−1 transazioni ricevute da utenti e una transazione di pagamento per
il processing (si veda sotto).
Il ciclo di vita di un nodo pu`o essere cos`ı definito:
• Creazione di un blocco candidato
– Estrazione dalla transaction pool di un insieme di SO_BLOCK_SIZE−1 transazioni non ancora presenti nel
libro mastro
– Alle transazioni presenti nel blocco, il nodo aggiunge una transazione di reward, con le seguenti caratteristiche:
∗ timestamp: il valore attuale di clock_gettime(...)
∗ sender : -1 (definire una MACRO...)
∗ receiver : l’dentificatore del nodo corrente
∗ quantit`a: la somma di tutti i reward delle transazioni incluse nel blocco
∗ reward: 0
• Simula l’elaborazione di un blocco attraverso una attesa non attiva di un intervallo temporale casuale espresso
in nanosecondi compreso tra SO_MIN_TRANS_PROC_NSEC e SO_MAX_TRANS_PROC_NSEC.
• Una volta completata l’elaborazione del blocco, scrive il nuovo blocco appena elaborato nel libro mastro, ed
elimina le transazioni eseguite con successo dal transaction pool.

5.4 Libro mastro
Il libro mastro `e la struttura condivisa da tutti i nodi e gli utenti, ed `e deputata alla memorizzazione delle transazioni eseguite. Una transazione si dice confermata solamente quando entra a far parte del libro mastro. Pi`u in
dettaglio, il libro mastro `e formato da una sequenza di lunghezza massima SO_REGISTRY_SIZE di blocchi consecutivi. All’interno di ogni blocco sono contenute esattamente SO_BLOCK_SIZE transazioni. Ogni blocco `e identificato
da un identificatore intero progressivo il cui valore iniziale `e impostato a 0.
Una transazione `e univocamente identificata dalla tripletta (timestamp, sender, receiver). Il nodo che aggiunge
un nuovo blocco al libro mastro `e responsabile anche dell’aggiornamento dell’identificatore del blocco stesso.
5.5 Stampa
Ogni secondo il processo master stampa:
• numero di processi utente a nodo attivi
• il budget corrente di ogni processo utente e di ogni processo nodo, cos`ı come registrato nel libro mastro
(inclusi i processi utente terminati). Se il numero di processi `e troppo grande per essere visualizzato, allora
viene stampato soltanto lo stato dei processi pi`u significativi: quelli con maggior e minor budget.

5.6 Terminazione della simulazione
La simulazione terminer`a in uno dei seguenti casi:
• sono trascorsi SO_SIM_SEC secondi,
• la capacit`a del libro mastro si esaurisce (il libro mastro pu`o contenere al massimo SO_REGISTRY_SIZE blocchi),
• tutti i processi utente sono terminati.
Alla terminazione, il processo master obbliga tutti i processi nodo e utente a terminare, e stamper`a un riepilogo
della simulazione, contenente almeno queste informazioni:
• ragione della terminazione della simulazione
• bilancio di ogni processo utente, compresi quelli che sono terminati prematuramente
• bilancio di ogni processo nodo
• numero dei processi utente terminati prematuramente
• numero di blocchi nel libro mastro
• per ogni processo nodo, numero di transazioni ancora presenti nella transaction pool
6 Descrizione del progetto: versione “normal” (max 30)
All’atto della creazione da parte del processo master, ogni nodo riceve un elenco di SO_NUM_FRIENDS nodi amici.
Il ciclo di vita di un processo nodo si arricchisce quindi di un ulteriore step:
• periodicamente ogni nodo seleziona una transazione dalla transaction pool che `e non ancora presente nel
libro mastro e la invia ad un nodo amico scelto a caso (la transazione viene eliminata dalla transaction pool
del nodo sorgente)
Quando un nodo riceve una transazione, ma ha la transaction pool piena, allora esso provveder`a a spedire tale
transazione ad uno dei suoi amici scelto a caso. Se la transazione non trova una collocazione entro SO_HOPS l’ultimo
nodo che la riceve invier`a la transazione al processo master che si occuper`a di creare un nuovo processo nodo che
contiene la transazione scartata come primo elemento della transaction pool. Inoltre, il processo master assegna
al nuovo processo nodo SO_NUM_FRIENDS processi nodo amici scelti a caso. Inoltre, il processo master sceglier`a a
caso altri SO_NUM_FRIENDS processi nodo gi`a esistenti, ordinandogli di aggiungere alla lista dei loro amici il processo
nodo appena creato.

7 Configurazione
I seguenti parametri sono letti a tempo di esecuzione, da file, da variabili di ambiente, o da stdin (a discrezione
degli studenti):
• SO_USERS_NUM: numero di processi utente
• SO_NODES_NUM: numero di processi nodo
• SO_BUDGET_INIT: budget iniziale di ciascun processo utente
• SO_REWARD: la percentuale di reward pagata da ogni utente per il processamento di una transazione
• SO_MIN_TRANS_GEN_NSEC, SO_MAX_TRANS_GEN_NSEC: minimo e massimo valore del tempo (espresso in nanosecondi) che trascorre fra la generazione di una transazione e la seguente da parte di un utente

• SO_RETRY, numero massimo di fallimenti consecutivi nella generazione di transazioni dopo cui un processo
utente termina
• SO_TP_SIZE: numero massimo di transazioni nella transaction pool dei processi nodo
• SO_MIN_TRANS_PROC_NSEC, SO_MAX_TRANS_PROC_NSEC: minimo e massimo valore del tempo simulato (espresso
in nanosecondi) di processamento di un blocco da parte di un nodo
• SO_SIM_SEC: durata della simulazione (in secondi)
• SO_NUM_FRIENDS (solo versione max 30): numero di nodi amici dei processi nodo (solo per la versione full)
• SO_HOPS (solo versione max 30): numero massimo di inoltri di una transazione verso nodi amici prima che il
master creai un nuovo nodo
Un cambiamento dei precedenti parametri non deve determinare una nuova compilazione dei sorgenti.
Invece, i seguenti parametri sono letti a tempo di compilazione:
• SO_REGISTRY_SIZE: numero massimo di blocchi nel libro mastro
• SO_BLOCK_SIZE: numero di transazioni contenute in un blocco
La seguente tabella elenca valori per alcune configurazioni di esempio da testare. Si tenga presente che il progetto
deve poter funzionare anche con altri parametri.
parametro letto a . . . “conf#1” “conf#2” “conf#3”
SO_USERS_NUM run time 100 1000 20
SO_NODES_NUM run time 10 10 10
SO_BUDGET_INIT run time 1000 1000 10000
SO_REWARD [0–100] run time 1 20 1
SO_MIN_TRANS_GEN_NSEC [nsec] run time 100000000 10000000 10000000
SO_MAX_TRANS_GEN_NSEC [nsec] run time 200000000 10000000 20000000
SO_RETRY run time 20 2 10
SO_TP_SIZE run time 1000 20 100
SO_BLOCK_SIZE compile time 100 10 10
SO_MIN_TRANS_PROC_NSEC [nsec] run time 10000000 1000000
SO_MAX_TRANS_PROC_NSEC [nsec] run time 20000000 1000000
SO_REGISTRY_SIZE compile time 1000 10000 1000
SO_SIM_SEC [sec] run time 10 20 20
SO_FRIENDS_NUM run time 3 5 3
SO_HOPS run time 10 2 10
8 Requisiti implementativi
Il progetto (sia in versione “minimal” che “normal”) deve
• essere realizzato sfruttando le tecniche di divisione in moduli del codice,
• essere compilato mediante l’utilizzo dell’utility make
• massimizzare il grado di concorrenza fra processi
• deallocare le risorse IPC che sono state allocate dai processi al termine del gioco
• essere compilato con almeno le seguenti opzioni di compilazione:
gcc -std=c89 -pedantic
• poter eseguire correttamente su una macchina (virtuale o fisica) che presenta parallelismo (due o pi`u processori).
Per i motivi introdotti a lezione, ricordarsi di definire la macro GNU SOURCE o compilare il progetto con il flag
-D GNU SOURCE.
