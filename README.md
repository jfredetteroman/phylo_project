# Botany 563 Final Project: Reproducible Script
This README can be used to follow along with the steps used to construct a phylogenetic tree of Sodium/Potassium ATP-ase across several different arthropod species. 

# Step 0: Install software

## Alignment software: T-Coffee

Using a Unix/Linux system, download the T-Coffee installer:

`wget http://tcoffee.org/Packages/Stable/Latest/ `

Grant execution permission to the installer:

`chmod +x T-COFFEE_installer_Version_13.45.0.4846264_linux_x64.bin`

Launch the installation wizard:

`./T-COFFEE_installer_Version_13.45.0.4846264_linux_x64.bin`

After following the installation wizard and restarting the terminal, verify that the installation was successful:

`t_coffee -version`

## Tree construction software: IQ-TREE

Using a conda environment, run the following command:

`conda install -c bioconda iqtree`

## Using this github repository alongside your analysis

Much of the data collection and sequence translation was completed outside of the command line due to limitations of certain tools. While this readme still describes those steps in an effort to ensure reproducibility, those who wish to follow along starting with "Step 2: Multiple sequence alignment" should use the FASTA files included in this repository. In order to access these FASTA files (and the rest of the files included in this analysis), run the following git command:

`git clone https://github.com/jfredetteroman/phylo_project.git`

# Step 1: Collect sequences

## BLAST searches

Using the [species list](https://github.com/jfredetteroman/phylo_project/blob/main/SpeciesList.md) included in this repository and the genome database specified in the list, run a BLAST search on all species against the [*E. affinis* sequence](https://github.com/jfredetteroman/phylo_project/blob/main/FASTAs/Eurytemora_affinis_BLAST_reference.fa) included in this repository. For each BLAST result with an E-value smaller than 1e-5, add the resulting sequence to a FASTA file corresponding to that species. The results from this step can be found in the [FASTAs](https://github.com/jfredetteroman/phylo_project/tree/main/FASTAs) directory.

## Convert to amino acid sequences

Using [ExPASy's translate tool](https://web.expasy.org/translate/), input nucleotide sequences retrieved in BLAST searches, selecting "compact" output format and both forward and reverse DNA strands. The [resulting output](https://www.dropbox.com/s/qp16x93l1y2w0ig/ExPASy_Result.png?dl=0) includes each reading frame in both the 5'-3' and 3'-5' directions, with open reading frames highlighted in red. For each sequence, select the largest open reading frame and add it to a new FASTA file. While this step is susceptible to subjectivity, each translation should have an open reading frame that is clearly larger than the others. The [resulting FASTA file](https://github.com/jfredetteroman/phylo_project/blob/main/FASTAs/All_Species_Amino_Acid.fa) should contain an amino acid sequence corresponding to each nucleotide sequence retrieved from BLAST searches. Each sequence represents a different paralog of sodium/potassium ATP-ase belonging to the specified species.

It is possible to run ExPASy's translate tool from the command line, but the output is a FASTA file that contains all 6 reading frames and does not specify the locations of open reading frames. While this made the web interface more suitable for our goal, an example of the translate tool being used on the command line is shown below:

`curl -s -d "DNA_sequence=GAGAAACAATGT...&output_format=fasta" https://web.expasy.org/cgi-bin/translate/dna2aa.cgi > translation_output.fasta`

# Step 2: Multiple sequence alignment

## Generate alignment file

To create the multiple sequence alignment that will be used to construct a phylogeny, run T-Coffee on the FASTA file containing amino acid sequences for all species:

`t_coffee phylo_project/FASTAs/All_Species_Amino_Acid.fa`

## Trim data

Because the sequences were largely collected from un-annotated data, there is a significant possibility that there are pseudogenes in the dataset. Remove sequences that are too distantly related from the rest of the set with the following T-COFFEE command:

`t_coffee -other_pg sequence_reformat -in All_Species_Amino_Acid.aln -action +trim _aln_%%100_O45 > All_Species_Amino_Acid_trimmed.aln`

A threshold of 45% identity was chosen according to the recommended threshold in the T-COFEE documentation. Moreover, sequences from the outgroup *C. elegans* did not get removed with a threshold of 45% identity. We now have our trimmed alignment file and are ready to construct a tree. The [untrimmed alignment file](https://github.com/jfredetteroman/phylo_project/blob/main/Alignment_Files/All_Species_Amino_Acid.aln) and the [trimmed alignment file](https://github.com/jfredetteroman/phylo_project/blob/main/Alignment_Files/All_Species_Amino_Acid_trimmed.aln) can be found in this repository.

# Step 3: Construct phylogeny

To construct a tree with IQ-TREE, run the following command on the trimmed alignment file produced by T-COFFEE:

`iqtree -s All_Species_Amino_Acid_trimmed.aln -st AA -o CelA1 -m MFP -b 1000`

The command above tells IQ-TREE to construct a phylogeny with certain parameters. `-s All_Species_Amino_Acid_trimmed.aln` specifies the alignment file to use. `-st AA` specifies that the sequences in the alignment are in amino acid format. `-o CelA1` specifies the outgroup. `-m MFP` specifies that IQ-TREE should use ModelFinder Plus to identify the best substitution model that minimizes BIC score. `-b 1000` specifies that IQ-TREE should use the tool UFBoot with 1000 bootstrap approximations.

The resulting [treefile](https://github.com/jfredetteroman/phylo_project/blob/main/IQ-TREE_Files/All_Species_Amino_Acid.iqtree) produced by IQ-TREE can be viewed and analyzed using [iTOL](https://itol.embl.de/)
