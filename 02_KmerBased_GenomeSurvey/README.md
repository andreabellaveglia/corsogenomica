# Kmer based Genome survey

 `kat hist -t <THREADS> -m <MER LENGTH> -o <OUTPUT PREFIX> <paired_1> <paired_2>` 

--> comando per analizzare le k-mer per assemblamento del genoma e valutazione, in questo contesto lo compileremo a mano

__kat hist -t 8 -m 27 -o anoste_27 SRR11672503_1_paired.fastq SRR11672503_2_paired.fastq__

il risultato dell'analisi fatta lanciando il comando sopra citato è disponibile al link:
http://genomescope.org/genomescope2.0/analysis.php?code=GnQDF5ndDygPwTvAOGso

Nei grafici sull'asse delle x è indicata la coverage, mentre l'asse y è la frequenza. Questa analisi ci permette di ottenere informazioni come grandezza stimata del genoma, omozigosità, eterozigosità, ploidia e quantità di sequenze ripetute ed uniche. La coverage in questo contesto rappresenta in quante short reads è identificabile in una specifica posizione uno specifico nucleotide, questo vuol dire che per avere una coverage maggiore serve anche un maggior numero di reads. Quando si ha già un genoma di riferimento si può rinunciare ad una coverage alta per fare un maggior numero di reads (sacrifichiamo la qualità per la quantità) in modo da coprire maggiormente la varietà genetica di una specie.  

L'analisi con Genome Scope è utilissima per comprendere la qualità del sequenziamento prima ancora di procedere all'assemblamento. 

