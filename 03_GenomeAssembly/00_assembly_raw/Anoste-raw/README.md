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


I contig così creati possono essere poi controllati sia quantitativamente (la loro lunghezza) tramite un valore detto N50, sia qualitativamente con un indice detto BUSCO (che confrontando il genoma che stiamo assemblando con altri genomi già caricati nei database di riferimento riesce a riconoscere e a contare il numero di geni ortologhi identificati dal nostro assemblamento). 

Innanzitutto partiamo col lanciare BUSCO:

`export NUMEXPR_MAX_THREADS=<CPU_number>` --> con questo comando forniamo al programma il numero di processori a disposizione.

Fatto ciò possiamo lanciare il comando vero e proprio:
`busco -m <MODE> -l <LINEAGE> -c <CPU_NUMBER> -o <OUTPUT_NAME> -i <INPUT>`

- '-m' # define whether we are using genomes or proteomes

  There are three valid modes:
     - geno or genome, for genome assemblies (DNA)
     - tran or transcriptome, for transcriptome assemblies (DNA)
     - prot or proteins, for annotated gene sets (protein)

- '-l' # redirect to the link of the library of reference

     Specify the name of the BUSCO lineage to be used.

- '-c' # the number of cores to parallelise the process
- '-o' # name of the output **folder**
- '-i' # the input. In our case, the raw assembly

`busco -m geno -l $BUSCO/diptera_odb12 -c 8 -o Anoste-raw -i Anoste_raw.fasta`



## Genome polishing with long and shot reads 

Il prerequisito per il programma che andremo ad utilizzare per il polishing in questo caso (hypo) è il mapping, ovvero lanciamo un BLAST per trovare le corrispondenze tra l'assembly e le reads. Per farlo usiamo uno script ovvero un pacchetto di comandi che il computer lancerà in maniera sequenziale permettendoci di risparmiare tempo facendo runnare i comandi uno dopo l'altro senza la necessità del nostro intervento.  


### Mapping short and long reads on assembly

Per scrivere lo script creiamo un file nel text editor con il comando:

`nano mapping.sh` 


```
#!/bin/bash

# Mapping short and long reads on assembly

CPU=$1
INPUT=$2
SR_1=$3
SR_2=$4
PREFIX=$5
LR=$6

#MAPPING SHORT READS
# mapping short reads on assembly
minimap2 -ax sr --MD -t "$CPU" "$INPUT" "$SR_1" "$SR_2"  > "$PREFIX"_sr.sam

# conversion SAM to BAM
samtools view -b "$PREFIX"_sr.sam > "$PREFIX"_sr.bam
rm "$PREFIX"_sr.sam

# sorting and indexing of BAM file
samtools sort -@"$CPU" -o "$PREFIX"_sr_sorted.bam "$PREFIX"_sr.bam
samtools index "$PREFIX"_sr_sorted.bam

#MAPPING LONG READS
# mapping long reads on assembly
minimap2 -ax pb-map --MD -t "$CPU" "$INPUT" "$LR"  > "$PREFIX"_lr.sam

# conversion SAM to BAM
samtools view -b "$PREFIX"_lr.sam > "$PREFIX"_lr.bam
rm "$PREFIX"_lr.sam

# sorting and indexing of BAM file
samtools sort -@"$CPU" -o "$PREFIX"_lr_sorted.bam "$PREFIX"_lr.bam
samtools index "$PREFIX"_lr_sorted.bam

```

Nella parte iniziale abbiamo specificato delle variabili con il simbolo $ per specificarne un ordine in modo che l'inserimento possa essere posizionale e non dipendere da singoli comandi e per poterli richiamare tra le virgolette all'interno dello script. 

Adesso per lanciare lo script basta solo usare il comando _bash_ <nome_script> seguito dai valori delle variabili che abbiamo scelto nell'orine in cui le abbiamo ordinate.

`
bash mapping.sh 8 Anoste_raw.fasta SRR11672503_1_paired.fastq SRR11672503_2_paired.fastq Anoste_pol SRR11672506.fastq.gz
`








