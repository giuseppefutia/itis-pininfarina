# Cosa fare con i dati

#### Esempi pratici di applicazioni e servizi sviluppati con i dati

All'interno di questo [GitHub](https://github.com/) repository è disponibile il materiale per gli studenti dell'[Istituto Tecnico Industriale Pininfarina](http://www.itispininfarina.it/) con un esempio pratico per lo sviluppo di applicazioni basate sui dati messi a disposizione dalla pubblica amministrazione (PA).

Potete voi stessi migliorare il contenuto di questo repository: per sapere come, date uno sguardo al meccanismo del [*forking*](https://help.github.com/articles/fork-a-repo/) e delle [*pull request*](https://help.github.com/articles/fork-a-repo/).

## Indice dei contenuti
* [Cosa fare con i dati della PA](https://github.com/giuseppefutia/itis-pininfarina#cosa-fare-con-i-dati-della-pa)
  * [HACK4MED, una sfida per accrescere il valore dei dati](https://github.com/giuseppefutia/itis-pininfarina/#hack4med-una-sfida-per-accrescere-il-valore-dei-dati)
  * [I dataset](https://github.com/giuseppefutia/itis-pininfarina/#i-dataset)
  * [D3.js, una libreria per la visualizzazione di dati sul Web](https://github.com/giuseppefutia/itis-pininfarina/#d3js-una-libreria-per-la-visualizzazione-di-dati-sul-web)
  * [Processare i dati](https://github.com/giuseppefutia/itis-pininfarina/#processare-i-dati)
    * [Gestire i dati geografici (Shapefile)](https://github.com/giuseppefutia/itis-pininfarina/#gestire-i-dati-geografici) 
    * [Gestire i dati di Certificazione Energetica (ACE)](https://github.com/giuseppefutia/itis-pininfarina/#gestire-i-dati-di-certificazione-energetica-ace)
  * [Il risultato: una mappa interattiva](https://github.com/giuseppefutia/itis-pininfarina/#il-risultato-una-mappa-interattiva)
 
## Cosa fare con i dati della PA
In questa sezione vengono date le linee guida generali per sviluppare una visualizzazione come quella disponibile all'indirizzo: http://giuseppefutia.github.io/homer/.

### HACK4MED, una sfida per accrescere il valore dei dati
[HACK4MED](http://www.hackunito.it/hack4med/) è la 24 ore non stop di hackathon del progetto
<img src="https://raw.githubusercontent.com/giuseppefutia/itis-pininfarina/master/images/hack4med.png" align="right" width=150>[HOMER](http://homerproject.eu/) (*Harmonising Open Data in the Mediterranean Through better access and reuse of public sector information: results and beyond*) che si è svolta il 17/18 Maggio 2014 presso il Campus Luigi Einaudi dell'Università di Torino. 700 dataset liberi (open data) sono stati a disposizione di sviluppatori e aziende per sviluppare un progetto tecnologico che facesse uso di almeno uno di questi set di dati.

#### OpenEnergyMap
In HACK4MED il nostro obiettivo era quello di creare una visualizzazione che permettesse di identificare in Piemonte (a livello regionale o di città) le zone a più alta efficienza energetica (dati sulla certificazione energetica) per una pianificazione del territorio e per permettere una scelta consapevole.

### I dataset
* *Numero di Attestati di Certificazione Energetica (ACE) distinti per classe energetica e per comune* ([dati.piemonte.it](http://www.dati.piemonte.it/catalogodati/dato/100319-numero-di-attestati-di-certificazione-energetica-ace-distinti-per-classe-energetica-e-per-comune.html))

* *Confini amministrativi, dei sistemi locali del lavoro e delle NUTS2 in versione generalizzata* ([istat.it](http://www.istat.it/it/archivio/24613))

### D3.js, una libreria per la visualizzazione di dati sul Web
[D3.js](http://d3js.org/) è una libreria JavaScript per manipolare documenti basati sui dati. D3 consente di "portare i dati alla vita" utilizzando tecnologie come HTML, SVG e CSS. Con d3 è possibile sfruttare tutte le funzionalità dei browser moderni, utilizzando un approccio basato sui dati per la manipolazione del [DOM](http://it.wikipedia.org/wiki/Document_Object_Model).

### Processare i dati
In questa sezione vengono riportate le istruzioni necessarie per manipolare i dati geografici, elaborare i dati relativi alla certificazione energetica e costruire infine un **mapping** tra i due dataset che consenta la creazione di una mappa interattiva.

#### Gestire i dati geografici
Per poter rappresentare i confini geografici della province e dei comuni del Piemonte utilizzando la libreria d3.js è stato necessario scaricare i dati dei confini amministrativi dal sito dell'ISTAT e convertire gli shapefile in formato [**TopoJSON**](https://github.com/mbostock/topojson/wiki).

Per compiere tale operazione si può utilizzare un tool che si chiama [**ogr2ogr**](http://www.gdal.org/ogr2ogr.html) presente all'interno della [Geospatial Data Abstraction Library](http://www.gdal.org/) o GDAL, libreria Open Source per leggere e scrivere numerosi formati di dati geografici.

GDAL presenta un modello di dati astratto comune attraverso il quale le applicazioni possono accedere a tutti i formati di dati geografici raster supportati. Insieme alla libreria vera e propria GDAL è accompagnata da numerose applicazioni a linee di comando che permettono di eseguire traduzioni di formato e semplici processamenti dei dati geografici (descrizione tratta da [Wikipedia](http://it.wikipedia.org/wiki/GDAL)).

Per prima cosa occorre scaricare gli shapefile dal sito dell'ISTAT delle province e dei comuni italiani al 2011:
* Province: http://www.istat.it/it/files/2011/04/prov2011_g.zip
* Comuni: http://www.istat.it/it/files/2011/04/com2011_g.zip

Per la visualizzazione dei file in formato shapefile si possono utilizzare gratuitamente i seguenti programmi open source:
* Quantum GIS: http://www.qgis.org
* GVSIG: http://www.gvsig.com

Successivamente occorre installare il tool ogr2ogr. Su Ubuntu si possono utilizzare i seguenti comandi:

    #add ppa lauchpad for ubuntugis
    sudo add-apt-repository ppa:ubuntugis/ppa
    sudo apt-get update
    sudo apt-get install gdal-bin

Una volta scaricati i dati e installato ogr2ogr è possibile utilizzare lo script definito da @riccardoscalco per trasformare gli shapefile in GeoJSON e successivamente in TopoJSON:

    ogr2ogr -f GeoJSON -s_srs prov2011_g.prj -t_srs EPSG:4326 sub.json prov2011_g.shp
    topojson -p nome_pro=NOME_PRO -p nome_pro --id-property NOME_PRO -s 0.00000001 -o itx.json sub.json

(ovviamente per i comuni italiani andrà applicato lo stesso procedimento).

A questo punto occorre definire un filtro per pulire i file TopoJSON dalle province e dai comuni che non ci interessano.

Tale pulizia dà come risultato i seguenti file che rappresentno i comuni e le province del Piemonte in formato TopoJSON utilizzabile dalla libreria d3.js.

* https://github.com/giuseppefutia/itis-pininfarina/blob/master/data/municipality.json
* https://github.com/giuseppefutia/itis-pininfarina/blob/master/data/province.json
 
#### Gestire i dati di Certificazione Energetica (ACE)
Per collocare su una mappa i dati sulla certificazione energetica, ci siamo basati su due dataset scaricati da dati.piemonte.it:
* [Numero di Attestati di Certificazione Energetica (ACE) distinti per classe energetica e per comune](https://github.com/giuseppefutia/itis-pininfarina/blob/master/data/comuni-efficienza.csv)
* [Numero di impianti termini distinti per tipologia e per comune](https://github.com/giuseppefutia/itis-pininfarina/blob/master/data/impianti-termici.csv)

Il secondo dataset è stato utile in particolare per avere un mapping diretto tra comuni e provincie del Piemonte.

Questi dati sono stati convertiti in JSON utilizzando il seguente script Python: https://github.com/giuseppefutia/itis-pininfarina/blob/master/scripts/convert.py.

In questo modo siamo riusciti ad ottenere il seguente file JSON: https://github.com/giuseppefutia/itis-pininfarina/blob/master/data/result.json

### Il risultato: una mappa interattiva
* Codice sorgente: https://github.com/giuseppefutia/giuseppefutia.github.io/tree/master/homer
* Risultato: http://giuseppefutia.github.io/homer/

## Contatti
* Mail: giuseppe.futia[at]polito[punto]it
* Twitter [@giuseppe_futia](https://twitter.com/giuseppe_futia)
* GitHub: https://github.com/giuseppefutia
