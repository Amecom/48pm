# Introduzione (BOZZA)

48pm è una piattaforma di data mining e analisi di articoli provenienti
dai principali giornali e magazine online in cinque lingue e 
in dodici nazione diverse. A oggi, 14 Aprile 2021, è possibile interrogare
gli articoli archiviati a partire dal 11 gennaio 2017: un base dati 
di poco meno di 30 milioni di articoli, oltre 21 milioni d'immagini, 
580 mila video, 180 mila entità e oltre 380 mila pattern testuali.

Considerando che ogni giorno vengono analizzati e archiviati
decine di migliaia di nuovi dati e vengono aggiunte nuove fonti, 
questi numeri sono in costante crescita. 

Dal 2017 il sistema ha subito una sola interruzione superiore alle sei ore 
e un solo un paio di interruzioni inferiori ai 60 minuti.

Le API possono essere utilizzate per costruire applicazioni come 
l'app 48pm. Questo documento vuole introdurre l’utilizzo 
delle API pubbliche fornendo, quando necessario, anche qualche dettaglio tecnico. 


## Introduzione alle API

Le API di 48pm forniscono un interfaccia stabile per interrogare le informazioni 
(articoli, video, scrittori, etc) archiviate e analizzate da 48pm.
Sono accessibili da specifici endpoint utilizzando svariati parametri di ricerca
che possono essere inviati indifferentemente tramite querystring 
o utilizzando il metodo POST.

* Una lista completa delle API pubbliche può essere ottenuta interrogando
  https://api.48.pm/list

* È possibile ottenere una documentazione sui parametri supportati da una 
  specifica API interrogando https://api.48.pm/<APINAME>/help 
  sostituendo `<APINAME>` con il nome della API di cui si vogliono 
  le informazioni. 

* È possibile ottenere una lista di valori supportati da alcuni parametri
  chiamando https://api.48.pm/codes


La API principale è *article* e mette a disposizione molti parametri 
per cercare articoli con estrema flessibilità. 
Oltre a questa esistono altre API come "author", "entity" o "writer"
per cercare in basi di dati più specifiche e altre ancora come
"topstory", "entity_trends" o "author_suggestion" che forniscono un accesso 
a informazioni più elaborate.

La API “article”, è sicuramente la più articolata, i cui parametri sono 
spesso comuni con le altre API per cui verrà spiegata più nel dettaglio.


# API ARTICLE

Endpoint: https://api.48.pm/article

La API *article* permette di cercare tra i 30 milioni di articoli archiviati
da gennaio 2017 a oggi. Per conoscere la lista completa dei parametri supportati 
basta chiamare https://api.48.pm/article/help

Per ottenere gli ultimi articoli archiviati è sufficiente chiamare 
l’endpoint senza alcun parametro: https://api.48.pm/article

Eseguendo questa chiamata tuttavia quello che dovrebbe saltare subito 
all’occhio è “dove sono gli articoli?!” 

Il nodo *items* del JSON di questa chiamata infatti restituisce solo
l’indice degli articoli trovati. L’invio dell'indice, piuttosto che del
contenuto completo, permette un notevole risparmio nella quantità di 
dati trasmessi. Sarà l’applicazione che utilizza la API a decidere cosa 
fare con gli ID ricevuti, potrebbe, ad esempio, recuperare le informazioni 
complete da una cache locale o recuperare le informazione complete 
utilizzando il parametro `article` della API *article*.

È possibile disabilitare questo comportamento impostando il parametro 
`page_index=0` ad esempio chiamando https://api.48.pm/article?page_index=0 
la risposta conterrà tutte le informazioni sugli ultimi articoli archiviati.

Il parametro `page_index` è attivo di default tranne quando la API viene 
utilizzata per recuperare uno o più ID articolo tramite il parametro `article`.
Ad esempio chiamando 
https://api.48.pm/article?article=l4dgb20 
l’indice viene disattivato automaticamente.
Questo comportamento è vero fin tanto che non vengono aggiunti altri parametri.
Gli unici parametri che possono essere utilizzati insieme al parametro 
`article` e che disattivano automaticamente il `page_index` sono `local_language`
e `page_limit`.

Nel corso dei successivi esempi verrà sempre appeso il parametro 
`page_index=0` a tutte le richieste per facilitare la lettura delle risposte.


### Valori multipli
Alcuni parametri supportano valori multipli e questi devono essere passati
separati da virgola. La ricerca viene eseguita concatenando questi valori con
l’operatore *OR* ed esempio chiamando 
https://api.48.pm/article?language=it,es&page_index=0
ottengo gli ultimi articoli pubblicati in Italiano o in Spagnolo.

I parametri che supportano valori multipli sono quelli che hanno specificato 
*multivalues: True* nella descrizione del parametro ottenibile chiamando 
l’help della API.

### Parametri di esclusione
Molti dei parametri supportati dalle API hanno un corrispettivo di esclusione.
Il parametro `language` ad esempio ha il suo corrispettivo `not_language`.
Chiamando
https://api.48.pm/article?not_language=it&page_index=0
escluderò dai risultati tutti gli articoli scritti in italiano.

## Filtrare per ID articolo
Il parametro `article` permette di recuperare informazioni su uno o più 
articoli di cui conosciamo l’ID. Supporta valori multipli ed esiste 
il corrispettivo di esclusione `not_article`.


## Filtrare per lingua

È possibile cercare articoli scritti in una o più lingue utilizzando il parametro 
`language` ad esempio
https://api.48.pm/article?language=it&page_index=0

I codici lingua corrispondono allo standard ISO 639-2 e 
la lista delle lingue supportate è reperibile nel nodo *data.language* 
del JSON ottenuto chiamando https://api.48.pm/codes

Il parametro `language` supporta valori multipli ed è possibile utilizzare 
il parametro `not_language` per escludere articoli scritti in una o più lingue.

## Filtrare per nazione

È possibile ottenere articoli provenienti da una o più nazioni 
utilizzando il parametro `country` ad esempio 
https://api.48.pm/article?country=es&page_index=0
I codici nazione corrispondono allo standard ISO 3166-1 alpha-2. 
È possibile ottenere la lista dei valori attualmente supportati nel nodo
*data.country* chiamando https://api.48.pm/codes

***NOTA:*** Alcuni giornali indicizzati provengono da nazioni fuori
di lista. Questi giornali sono identificati e ricercabili 
impostando `country=xx`.

Il parametro `country` supporta valori multipli ed è possibile utilizzare 
`not_country` per escludere articoli provenienti da una o più nazioni.

## Filtrare per autore

È possibile filtrare articoli per autore utilizzando il parametro `author`
tuttavia è necessario conoscere l’ID dell’autore per poterlo usare come filtro.

Ad esempio per ottenere articoli scritti dalla *CNN* NON POSSO chiamare
https://api.48.pm/article?author=cnn
Ma devo prima sapere qual’è l'ID autore associato alla CNN, 
per farlo devo utilizzare la API *author* e il suo parametro `search`:
https://api.48.pm/author?search=cnn 
Dalla risposta ottenuta ora so che l’ID della CNN è *129* e posso ottenere 
articoli scritti dalla CNN chiamando 
https://api.48.pm/article?author=129&page_index=0

L’utilizzo degli ID permette un interfaccia più stabile di ricerca dato
che il nome di un autore potrebbe modificarsi mentre il suo ID rimane
stabile nel tempo.

Il parametro `author` supporta valori multipli ed è possibile escludere autori
dai risultati utilizzando il parametro `not_author`.


## Filtrare per scrittore

Come per gli autori è possibile ottenere articoli scritti da uno o più 
scrittori utilizzando la stessa logica di author ma utilizzando il parametro
`writer`. Quindi se sappiamo il nome dello scrittore di cui volgiamo gli articoli
possiamo cercare il suo ID attraverso la API *writer*. 

Ad esempio se sono interessato agli articoli scritti da *Paul Krugman* 
posso cercarlo chiamando https://api.48.pm/writer?search=Paul%20Krugman
e, trovato che il suo ID è *34288*, posso ottenere i suoi articoli chiamando
https://api.48.pm/article?writer=34288&page_index=0

Il parametro `writer` supporta valori multipli ed è possibile escludere 
scrittori utilizzando il parametro `not_writer`.

**NOTA:** gli ID autore sono molto stabili ma gli ID scrittori 
vengono continuamente analizzati e possono cambiare a causa di aggregazioni 
o altre elaborazioni successive. Per mantenere aggiornata una lista di ID scrittori 
esiste la API *writer_maintainer* che, passata una lista di ID,
mi dice se questi sono cambiati e come. Per fare un esempio *Paul Krugman* 
per un breve lasso temporale è stato raggiungibile anche dal ID *565081*,
una successiva analisi ha poi aggregato l’ID *565081* all’ID *34288*.
Questa informazione mi viene data chiamando
https://api.48.pm/writer_maintainer?writer=565081

## Filtrare per entità

Una entità può essere una persona, un personaggio immaginario, una azienda,
una città, un film, un festival, una squadra di calcio, un argomento,
una categoria, etc, etc. Il sistema impara ogni giorno nuove entità 
utilizzando come risorsa Wikipedia e Wikidata, infatti l’ID di una entità
e del tutto corrisponde all'IDQ di una entità Wikidata.
Durante l’analisi di un articolo vengono associate a esso
una o più entità. Conoscendo l’ID entità è possibile ottenere gli articoli
a essa associati con la stessa logica già utilizzata per autori e scrittori.

Se per esempio voglio ottenere articoli su *Joe Biden* 
cercherò il suo ID utilizzando la API *entity*: 
https://api.48.pm/entity?search=joe%20biden
e, ora che so che il suo ID entità è 6279, posso ottenere gli articoli
a esso associati: https://api.48.pm/artice?entity=6279&page_index=0

Il parametro `entity` supporta valori multipli e il suo corrispettivo 
di esclusione è `not_entity` tuttavia se è vero che un articolo 
è pubblicato da un solo giornale ed è scritto in una sola lingua,
un articolo spesso può essere associato a più entità per questo esiste 
un ulteriore parametro che è `entity_all`.

Il parametro `entity_all` dovrebbe essere utilizzato solo passandogli 
valori multipli e ha lo scopo di concatenare più entità tra loro tramite 
l’operatore logico *AND* invece che tramite l'operatore *OR*

Quindi chiamando
https://api.48.pm/artice?entity=6279,7163&page_index=0
ottengo articoli che includono Joe Biden (6279) *OPPURE* Politica (7163) 
mentre chiamando 
https://api.48.pm/artice?entity_all=6279,7163&page_index=0 
ottengo articoli che includo Joe Binen (6279) *E* Politica (7163).

### Vantaggi nell’uso delle entità
L’utilizzo delle entità ha diversi vantaggi rispetto a un ricerca testuale:
1. La ricerca testuale è molto dispendiosa soprattutto 
   quando si cerca tra milioni di articoli e si ha a disposizione una
   bassa capacità computazionale, di contro la ricerca tramite ID entità
   è molto performante anche su milioni di articoli.
2. La ricerca attraverso ID entità è molto flessibile. Prendiamo le parole
   *Juve* e *Juventus*, entrambe si riferiscono alla stessa squadra di calcio
   tuttavia in una ricerca testuale pura restituirebbero risultati 
   differenti mentre durante l'analisi di un articolo il sistema 
   li riconosce come la medesima entità. Un altro esempio riguarda 
   la traduzione di termini in lingue diverse se *Italy* e *Italia*
   sono due parole “diverse” ma corrispondo allo stesso ID entità.

### Localizzazione delle informazioni entità
Quando si ottengono informazioni su un articolo il nodo *.entity* 
del JSON restituito contiene le informazioni sulle entità associate
ad esso associate. 
Tra le informazioni alcune come la descrizione e il titolo 
sono in lingua inglese tuttavia è possibile ottenere una versione 
localizzata di tali informazioni impostando il parametro `local_language` 
sul valore della lingua desiderata. 
Le lingue disponibili corrispondono a quelle del parametro `language`.

Facciamo un esempio guardando i dati restituiti chiamando
https://api.48.pm/article?page_index=0 e poi chiamando 
https://api.48.pm/article?page_index=0&local_language=it

## Filtrare per canale
Gli articoli analizzati vengono sempre ricondotti 
ad almeno un argomento come sport, tecnologia, politica, etc, etc,
chiamato canale. È possibile cercare articoli provenienti da uno o più canali
utilizzando il parametro `channel`. 
La lista dei canali disponibili è consultabile nel nodo channel del JSON
restituito da https://api.48.pm/codes ma, come si può notare,
ogni canale è associato a un ID entità. 
Infatti il parametro *channel* è solo uno "zuccherino" in quanto chiamare
https://api.48.pm/artice?channel=sport&page_index=0 è del tutto equivalente a chiamare
https://api.48.pm/artice?entity=349&page_index=0

Dato che la ricerca basata su ID entità è più flessibile 
è consigliato utilizzare sempre il paramento `entity` ed evitare l'uso del
parametro `channel`.

Il parametro `channel` supporta valori multipli ed è possibile
escludere articoli associati a un canale utilizzando `not_channel`.

## Filtrare per testo

Come detto la ricerca per ID entità è, nella maggior parte dei casi,
preferibile a un ricerca testuale tuttavia è possibile cercare articoli
che contengono un testo utilizzando il parametro `search`.

La ricerca testuale permette di escludere parole anteponendo 
il carattere “-” alla parole da escludere 
Esempio `search=mela -pera` mostra articoli che contengono 
il termine “mela” ma che non includo il termine “pera”.
Quando sono indicati più termini vengono restituiti soli gli articoli che 
li contengo tutti o che non includo uno tra quelli esclusi.

Al momento la ricerca non è molto performante ed è possibile cercare solo 
tra gli articoli pubblicati negli ultimi 15 giorni. È probabile che 
in futuro la ricerca testuale utilizzi un approccio 
misto sostituendo quando possibile le parole con gli ID entità associati 
relegando solo le parole “sconosciute” a una ricerca testuale pura 
e, di conseguenza, il lasso di tempo di ricerca potrebbe essere esteso.

## Ottenere articoli in evidenza
Gli articoli in evidenza hanno una API dedicata *topstory* 
tuttavia anche attraverso la API *article* e possibile selezionare 
gli articoli in evidenza attraverso il parametro `topstory=1`

## Ottenere la copertura completa di un articolo in evidenza
Gli articoli in evidenza sono tali solitamente perché più fonti trattano
lo stesso argomento, è possibile trovare l’ID argomento nel nodo *keyword* 
del JSON di risposta. Conoscendo l’ID di un argomento è possibile ottenere 
la copertura completa sull’argomento utilizzando il parametro `keyword`.
Ad esempio le parole *covid vacine* hanno generato una topstory 
il cui ID è *59715*. Per ottenere una copertura completa su questa parola chiave
basta chiamare https://api.48.pm/article?keyword=59715&page_index=0

## Rimuovere blog e fonti locali
Nel caso si desideri escludere dai risultati di ricerca 
le *fonti locali* e i *blog* occorre impostare il parametro `majorpress=1`

## Trovare articoli associati a un video
Alcuni articoli includo il collegamento a un video le cui informazioni 
sono inserite nel nodo *.video* del JSON di risposta. Ad esempio chiamando 
https://api.48.pm/article?article=l4ck7d0 si può vedere che l’articolo 
ha l'ID video “l4ceuf0” associato.
Conoscendo un ID video posso ottenere tutti gli articoli a esso associati
utilizzando il parametro `video` ad esempio chiamando
https://api.48.pm/article?video=l4ceuf0&page_index=0 
ottengo la lista degli articoli che puntano a quel video ma, 
visto che conosco l’ID articolo di partenza, potrei escluderlo dai risultati
chiamando https://api.48.pm/article?video=l4ceuf0&not_article=l4ck7d0&page_index=0

La ricerca di video può essere eseguita attraverso la API *video_type*:
https://api.48.pm/video_type


## Paginazione dei risultati
Interrogando una API è molto probabile che i risultati restituiti 
siano solo una parte dei risultati disponibili.
Prima di vedere come è possibile muoversi tra i risultati è necessario
dare un'occhiata alla risposta JSON ricevuta da una qualsiasi chiamata.

Il JSON di risposta restituisce un nodo *response* 
con due informazioni molto utili ovvero:
- *resonse.start*: l’ID del primo elemento valutato nella ricerca e
- *response.nextStart*: l’ID di paginazione successivo.

Quando *response.nextStart* è nullo significa che non ci sono più
risultati disponibili, se invece ha un valore questo può essere utilizzato
per ottenere la pagina di risultati successiva.

Tornando ai parametri di ricerca esistono tre parametri,
praticamente comuni a tutte le API, utili per paginare i risultati e sono:

* `page_limit` permette di modificare il numero massimo di risultati ottenibili 
  da una singola chiamata. Questo parametro di solito ha un default di 10
  e un valore massimo definito in ‘condition_range_max’ consultabile nell’help.

* `page_start` permette di ottenere altri dati partendo da una data
  o da un ID. Questo parametro è fondamentale per poter ottenere la
  paginazione successiva di una ricerca. In questo caso il suo valore 
  dovrebbe essere impostato sul valore di result.nextStart 
  ottenuto nella prima interrogazione. `page_start=<result.nextStart>`

* `page_stop` permette d'interrompere la ricerca a una data o a uno specifico ID
  articolo.


### Utilizzo dinamico di page_start e page_stop
Nelle API i cui risultati sono organizzati cronologicamente come *article* 
i parametri `page_start` e `page_stop` possono ricevere:

- *valori relativi* 'Dx' o 'Hx' (sostituendo 'x' con un numero di giorni o di ore),
  ad esempio impostando `page_start=D2` si otterranno risultati a partire
  da due giorni fa.

- *una data* nel formato 'YYYYMDDHHMM' (troncando il valore a piacimento). 
  Esempio `page_start=201910` eseguirà una ricerca partendo da Ottobre 2019
  (incluso).

- *un ID articolo* nel qual caso verranno restituiti articoli partendo
  dall’ID inserito (ID articolo non incluso).

### Ricevere aggiornamenti
Memorizzando il valore di *response.start* ottenuto da una ricerca posso
successivamente utilizzare questo valore nel parametro `page_stop`
per sapere si vi sono nuovi articoli disponibili dall’ultima chiamata:
`page_stop=<response.start>`.


### Ordinamento 
La API article non supporta nessun ordinamento, gli articoli vengono sempre
e solo restituiti in ordine cronologico dal più recente.
Altre API supportano l’ordinamento tramite l’uso del parametro `page_order`.
Consultare l’help delle singole API per sapere se l’ordinamento è supportato e
quali sono le opzioni di ordinamento disponibili.

## Nota tecnica
48pm utilizza MYSQL come database.

La API *article* supporta molti parametri 
che possono esser concatenati tra loro e molti di questi possono 
ricevere liste di valori. Questo significa che una query 
può facilmente diventare molto articolata.

Data la complessità e il numero elevato di record, è stato impossibile
creare una singola query che li includesse tutti a causa dei tempi assurdi
che MYSQL impiegava per eseguirla.

In breve, il problema è stato risolto:

1. creando tre liste di query che agiscono su un singolo parametro: 
   - query di selezione, 
   - query di inclusione,
   - query di esclusione;
    
2. ordinando le query sulla previsione del numero di articoli restituiti;

3. eseguendo le query in sequenza.

Questi tre passaggi hanno reso possibile eseguire 
in pochi millisecondi quello che una query unica impiegava svariati minuti e
ha semplificando sia la leggibilità che la manutenzione del codice.


# Query concorrenti
In alcuni casi è utile eseguire più ricerche contemporaneamente riducendo 
così i tempi di latenza per fare questo esiste un 
endpoit specifico: https://api.48.pm/concurrent_query.

L'endpoint può ricevere un file JSON con una lista di dizionari
con *paramentro: valore*, in aggiunta ai parametri supportati
da una API è necessario specificare a quale API ci si vuole riferire attraversa
il parametro *api*, ad esempio il JSON:

```json
[
  {
    "api": "article",
    "country": "it"
  },
  {
    "api": "article",
    "language": "en"
  }
]
```

Inviato all'endpoint https://api.48.pm/concurrent_query restituisce una lista
con i risultati delle due richieste utilizzando un singola chiamata.

# API TOPSTORY
Endpoint: https://api.48.pm/topstory

La API *topsory* restituisce gli articoli in evidenza.
Ha molto in comune con l'API *article* infatti, ***anche se non descritti
nel file di help*** https://api.48.pm/topstory/help questa API ***supporta tutti 
i parametri descritti nella API article*** più altri parametri specifici.
È utile spigare qualche differenza rilevante.

## Parametro author 
Il parametro `author` NON viene utilizzato come filtro ma come suggerimento.

Per comprendere meglio è utile ricordare che una topstory
è, nella maggior parte dei casi, l'articolo più rilevante 
di un gruppo di articoli che trattano tutti lo stesso argomento.
Solitamente l'articolo scelto come topstory è il più rappresentativo
del gruppo tuttavia tramite il parametro `author` e possibile
indicare una copertura preferenziale. Una topstory verrà coperta
quando possibile da un articolo pubblicato da un autore tra quelli indicati
nel parametro `author`, se non possibile verrà coperto 
da un qualsiasi altro autore (tranne quelli inclusi nel parametro `not_author`)

## Ordinamento
La API *topstory* supporta l'ordinamento attraverso il parametro `page_ order`, 

L'ordinamento è tuttavia sempre vincolato da un ordine cronologico giornaliero
per cui in linea di massima un articolo poco rilevante pubblicato
nelle ultime 24 ore verrà comunque visualizzato prima
di un articolo più rilevante ma pubblicato 24 ore prima.

I valori possibili di `page_order` sono:

- `time_relevance` mostra per prime le topstory più rilevanti nel momento,
- `relevance` mostra per prime le topstory più rilevanti della giornata,
- `time` mostra per prime le topstory più recenti,
- `toptoday` mostra per prime le rilevanti nelle ultime 18 ore. 

Vediamo alcune differenze

### Ordinamento time_relevance vs. relevance

Ogni articolo viene etichettato con due punteggi, uno di rilevanza e uno di
rilevanza nel tempo. Semplificando è possibile dire che
Il punteggio *relevance* viene calcolato in base la numero di autori che trattano
un argomento e può solo aumentare con il tempo mentre il punteggio 
*time_relevance* tiene conto del momento in cui l'argomento viene trattato
e col tempo può diminuire fino ad azzerarsi. Prendiamo l'esempio del risultato
di una partita di calcio. Appena terminata la partita molte fonti ne parleranno 
e la notizia potrebbe avere un valore di time_relevance molto alto ma,
passata qualche ora, il valore di time_relevance potrebbe crollare fino ad azzerasi
completamente. Ordinando per "time_relevance" quindi un articolo
meno rilevante ma molto attuale potrebbe essere mostrato prima di un articolo
più rilevante ma di cui non vengono pubblicati aggiornamenti da qualche
ora.

### Ordinamento toptoday
Simile a *relevance* ma attenua il problema di ordinamento
al passaggio tra un giorno e un altro. Come si diceva prima ogni forma di
ordinamento scelto è comunque vincolato da blocchi giornalieri. Questo vuol dire
che, ordinando per *relevance*, una topstory pubblicata dopo la mezzanotte (UTC)
verrà mostrata prima di qualsiasi altra pubblicata prima della mezzanotte.

Ordinando per *toptoday* invece viene creato un blocco di ordinamento 
per rilevanza che valuta come appartenenti allo stesso gruppo temporale
tutte le topstory delle ultime 18 ore anche a cavallo della mezzanotte.


# API di ricerca
Come già accennato durante la trattazione della API *article*
le query di ricerca sono quello utilizzate per cercare ID autori, scrittori 
ed entità tuttavia queste API mettono a disposizione
molti altri parametri oltre al `search` mostrato nei precedenti esempi
e possono essere utilizzate per ottenere molte più informazioni 
oltre a un semplice ID.

Gli endpoint sono:
* https://api.48.pm/author
* https://api.48.pm/writer
* https://api.48.pm/entity

Particolarmente utili in queste API possono risultare
le opzioni di ordinamento, per scoprire quali parametri 
sono ammessi da ogni API chiamare l'endpoint 
aggiungendo */help*.


# API Speciali
Esistono API che hanno lo scopo di suggerire o fornire classifiche e sono:

## API author_suggestion
Endpoint: https://api.48.pm/author_suggestion
Questa API suggerisci altri autori passando una lista di
autori preferiti tramite il parametro (obbligatorio) `author` 

Ad esempio abbiamo scoperto in precedenza che l'ID autore
della *CNN* è *129*, utilizzando questo ID posso
ottenere suggerimenti su altri autori chiamando
https://api.48.pm/author_suggestion?author=129
Il parametro `author` supporta liste di valori
ed è possibile impostare contemporaneità anche una lista
di autori da escludere tramite `not_author`.

Tra gli altri parametri posso specificare una lista di 
`favorite_channel` o una lista di `blocked_channel`
per ottenere dei suggerimenti 
più in linea con le mie preferenze.

Per la lista completa dei parametri supportati
chiamare https://api.48.pm/author_suggestion/help

## API entity_trends
Endpoint: https://api.48.pm/entity_trends
Questa API suggerisce le entità di tendenza. Le tendenze
sono sempre geo localizzate per cui è obbligatorio
specificare almeno una nazione attraverso il parametro
`country`.

Oltre alla nazione è possibile passare una lista
di canali da includere o escludere tramite i
parametri `channel` e `not_channel` ed escludere entità 
specifiche utilizzando `not_entity`

Per la lista completa dei parametri supportati
chiamare https://api.48.pm/entity_trends/help


## API video_type
Endpoint: https://api.48.pm/video_type
Questa API restituisce classifiche di video esistono diverse
classifiche selezionabili tramite il parametro `video_type`

Per sapere quali valori sono supportati da video_type
consultare il nodo *video_type* chiamando
https://api.48.pm/codes

Per conoscere la lista completa di parametri supportati,
incluse le opzioni di ordinamento consultare
https://api.48.pm/video_type/help


# Manipolazione delle immagini
Più avanti verrà esaminata nel dettaglio la risposta JSON
ottenuta quando viene chiamata una API ma per ora soffermiamoci
sugli url immagine associati ad autori, scrittori ed entità 
nei rispettivi nodi *.imageUrl* e all'immagine
associata a un articolo nel nodo *.image.url*.

Ad esempio la CNN riporta come imageUlr
https://img.48.pm/author/129 che mostra

![CNN](https://img.48.pm/author/129)

e Paul Krugman riporta come imageUrl https://img.48.pm/writer/34288

![Paul Krugman](https://img.48.pm/writer/34288)

Qualsiasi url immagine è manipolabile ottenendo un immagine
delle dimensioni volute appendendo all'url i parametri `h=n` e 
`w=n`

Proviamo a ottenere un immagine delle dimensioni desiderate
chiamando, ad esempio, https://img.48.pm/author/129?w=100

![CNN](https://img.48.pm/author/129?w=100)

Specificando la dimensione di un solo lato
l'altro di adegua proporzionalmente.

Il discorso è più interessante per quanto riguarda le immagini
associate a un articolo. Consideriamo le informazioni restituite
chiamando https://api.48.pm/article?article=l4ajEm1
dove troviamo nel nodo *.image* l'url dell'immagine
https://img.48.pm/n/l4ajEm1 che mostra:

![immgine](https://img.48.pm/n/l4ajEm1)

Come per tutte le immagini anche questa può essere
ridimensionata a piacimento utilizzando i parametri `w` e `h`. 
Ad esempio chiamando https://img.48.pm/n/l4ajEm1?w=200&h=200
visualizzeremo:

![immagine](https://img.48.pm/n/l4ajEm1?w=200&h=200)

Ma proviamo ora a passare dei parametri più "estremi"
chiamando ad esempio https://img.48.pm/n/l4ajEm1?w=100&h=300


![immagine](https://img.48.pm/n/l4ajEm1?w=100&h=300)

Sembra tutto "normale" in quanto il soggetto
principale dell'immagine è centrato orizzontalmente.
Proviamo a invertire l'orientamento chiamando https://img.48.pm/n/l4ajEm1?w=300&h=100

![immagine](https://img.48.pm/n/l4ajEm1?w=300&h=100)

Come si può notare l'immagine non è piacevole dato che 
la parte più rilevante, in questo caso il viso,
viene tagliato via dall'immagine.

Per ovviare a questo problema le immagini vengono 
analizzate e, dove possibile, vengono fornite le coordinate
x e y del punto in cui si trova il soggetto principale.

Nel caso in esempio nel JSON di risposta 
nel nodo *.image* è presente un nodo *.relevant*
con i valori {x: 379, y: 81}

Provando ad appendere questi due valori alla precedente
richiesta otterremmo il seguente url 
https://img.48.pm/n/l4ajEm1?w=300&h=100&x=379&y=81
che mostra:

![immagine](https://img.48.pm/n/l4ajEm1?w=300&h=100&x=379&y=81)

# Analisi del JSON di risposta

Il JSON di risposta ottenuto chiamando una API fornisce diverse
informazioni alcune sono abbastanza facili da interpretare
altre necessitano di una spiegazione. Per l'analisi del JSON
utilizzeremo la risposta che descrive un articolo dato che
comprende tutti i tipi di dati possibili. Per intenderci il nodo 
*.author* di un articolo descrive un autore esattamente
come farebbe la API *author* cercando un autore.

*** DA COMPLETARE ***