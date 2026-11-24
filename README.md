# corsogenomica

### per spostare file dalla repository locale a quella online usiamo i comandi:

* _git add_ --> per aggiungere quei file tra quelli da spostare

* _git status_ --> controllare i file rossi e verdi (locali e non)

* _git commit_ -m  "contenuto del commit" --> per committare le modifiche e confermarle

* _git push_ --> spinge i file nella repository online

* _git log_ --> registro dei commit




### per l'accesso al server
--> `ssh STUDENTI^andrea.bellaveglia@137.204.142.242`

* _ls_ --> lista dei file

* _ll_ --> lista completa di dati riguardanti e caratteristiche dei file

* _cd_ --> change directory
* cd .. permette di uscire da una cartella
* _mkdir_ <nome_cartella>/ permette di creare una nuova cartella

* _conda activate_ --> cambiare environment

* per entrare nella _screen_ --> tmux a -t nome della screen (la mia si chiama general)
* per creare una nuova _screen_ --> tmux new -s

* ln -s permette di creare un link e per fare in modo che venga creato nella cartella desiderata bisogna mettere in fondo un punto.

* _mv_ <nome_file> <nuovo_nome> --> permette di cambiare nome di un file, mv può essere usato anche per spostare file da una directory all'altra

* _cat_ --> ci permette di stampare in standard output il contenuto di un file

---

Per creare una nuova screen premere Ctrl+A e poi C, per muoversi poi tra le screen si può usare Ctrl+A e poi N (next) o P (previous).


# Genome Annotation 

L'annotazione è un processo ciclico diviso in diversi round durante i quali avvengono annotazioni cicliche del genoma. Il programma che usiamo per l'annotazione è Maker. 
Per iniziare è necessario modificare le opzioni (i control file) del programma per adattarlo alle nostre necessità. 

Con il comando maker -CTL creiamo i ctl file. Quello di nostro interesse è maker_opts.ctl. 

Three files are generated:

* maker_bopts: parameters for the software used by MAKER; we maintain default parameters. Here, a great source to understand each input parameter in MAKER.
* maker_exe: paths to all softwares
* maker_opts: MAKER run parameters; here we need to modify

Con nano possiamo modificare lo script inserendo il genoma da annotare, i proteomi di organismi vicni filogeneticamente a quello da noi in studio per creare un pool
di proteine con il quale il programma possa fare matchmaking rispetto al genoma che gli  abbiamo fornito e fornire una library che permetta di mascherare il materiale 
ripetuto. Dopodichè possiamo personalizzare il comportamento del programma dandogli indicazioni sui tipi di predizioni che deve fare (protein2genome nel nostro caso 
nel primo round), in numero di CPU che mettiamo a disposizione nonché fornito le informazioni riguardo le thrashold di ricerca delle proteine (minimo numero di amminoacidi 
per riconoscere una proteina come tale, permettere l'alternative splicing ecc..). 


Dal primo round di annotazione otteniamo un file di nome maker.tar.gz che decomprimiamo con tar -z -xf. 
Possiamo così lanciare i comandi: 

```
fasta_merge -d Anoste_rnd1_master_datastore_index.log
gff3_merge -d Anoste_rnd1_master_datastore_index.log
```

Dopodiché possiamo lanciare due comandi (enviroment GAAS) ce ci permettono di ottenere informazioni riguardo i dati che abbiamo ottenuto tramite l'annotazione:

```
agat_sp_statistics.pl --gff Anoste_rnd1.all.gff -o Anoste_rnd1_model_stats.log
agat_sq_repeats_analyzer.pl -i Anoste_rnd1.all.gff -o Anoste_rnd1_reapeats.log
```

__N.B.__(per il genome report --> con i risultati del secondo comando possiamo fare un pie chart per mostrare la distribuzione degli elementi ripetuti all'interno del genoma in analisi, mentre dal primo possiamo ricavare delle informazioni che possiamo anche comparare con altre annotazioni disponibili on line per capire la bontà della nostra analisi e anche le differenze esistenti) 

Con il pacchetto di tool SNAP è possibile creare modelli di predizione di porzioni codificanti del genoma, sotto forma di file utilizzabili poi da maker. Vengono estratti i modelli con i primi tre comandi, creato il modello unico e impacchettati poi con forge e assemblati in un file hammer.   

SNAP.sh :

```
maker2zff -c 0 -e 0 -l 80 -x 0.1 -d ../Anoste_rnd1_master_datastore_index.log #To extract gene models based on multiple filter criter>
fathom genome.ann genome.dna -gene-stats > Anoste_rnd1_fathom.stats #Print some summary statistics of the selected gene models
fathom genome.ann genome.dna -validate #Validate gene models and print summary statistics. Output is in stdout and cannot be redirect>
fathom genome.ann genome.dna -categorize 1000 #Extract gene models together with 1000 bp at both ends for training
fathom uni.ann uni.dna -export 1000 -plus #Export and convert unique genes to strand plus. Typically, only the unique genes are used >

mkdir forge
cd forge

forge ../export.ann ../export.dna #call the parameter estimation program, better in another directory

cd ..

hmm-assembler.pl Anoste_rnd1 forge > Anoste_rnd1.hmm
```






