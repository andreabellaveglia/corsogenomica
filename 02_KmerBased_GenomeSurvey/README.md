# Kmer based Genome survey

 `kat hist -t <THREADS> -m <MER LENGTH> -o <OUTPUT PREFIX> <paired_1> <paired_2>` 

--> comando per analizzare le k-mer per assemblamento del genoma e valutazione, in questo contesto lo compileremo a mano

__kat hist -t 8 -m 27 -o anoste_27 SRR11672503_1_paired.fastq SRR11672503_2_paired.fastq__

il risultato dell'analisi fatta lanciando il comando sopra citato è disponibile al link:
http://genomescope.org/genomescope2.0/analysis.php?code=GnQDF5ndDygPwTvAOGso

Nei grafici sull'asse delle x è indicata la coverage, mentre l'asse y è la frequenza. Questa analisi ci permette di ottenere informazioni come grandezza stimata del genoma, omozigosità, eterozigosità, ploidia e quantità di sequenze ripetute ed uniche. La coverage in questo contesto rappresenta in quante short reads è identificabile in una specifica posizione uno specifico nucleotide, questo vuol dire che per avere una coverage maggiore serve anche un maggior numero di reads. Quando si ha già un genoma di riferimento si può rinunciare ad una coverage alta per fare un maggior numero di reads (sacrifichiamo la qualità per la quantità) in modo da coprire maggiormente la varietà genetica di una specie.  

L'analisi con Genome Scope è utilissima per comprendere la qualità del sequenziamento prima ancora di procedere all'assemblamento. 

# Genome Assembly

se l'output di un comando fosse troppo lungo da leggere si può usare ctrl A e aperta parentesi quadra per scorrere con le freccette su e giù per l'output, per poi tornare a scrivere basta premere Q.

`wtdbg2 -x rs -g <EXPECTED_GENOMESIZE> -t <NUMBERS_OF_CORES> -i <FASTQ> -fo <OUT_PREFIX> #Default parameters` 
- '-x' #define the sequecing method. rs means PacBio RSII and sets some default parameters (listed in the program's help page).
- '-g' #the expected genome size inferred using GenomeScope (grandezza del genoma che ci aspettiamo stimata da Genome Scope) 
- '-t' #number of cores to parallelise the process (9)
- '-i' #require the long-read file, which can be compressed (il file contenente le reads, che peschiamo dalla repository di Mirko creando un link) 
- '-fo' #the prefix of our output (abbiamo usato Anoste_raw)

Con questo comando abbiamo creato il layout, ovvero sovrapposto le reads.

Il primo passo per l'assemblaggio di un genoma è la creazione dei contig, che avviene sovrapponendo le short reads permettendo la formazione di segementi contigui, ovvero senza alcun buco, di genoma. Questi segmenti continui subiscono poi lo scaffolding ovvero vengono orientati e ordinati per infine fare il gap filling ovvero l'utilizzo di short reads che precedentemente il programma aveva messo da parte perchè non sapeva dove mettere per riempire i buchi presenti tra i contig, alternativamente (anche se più costoso) è possibile effettuare un secondo sequenziamento atto a riempire questi buchi. 

`wtpoa-cns -t <NUMBER_OF_CORES> -i <LAYOUT_FILE-.ctg.lay.gz> -fo <OUT_PREFIX>`

Con questo comando creiamo il consensus. 
