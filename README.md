# scRNA-seq-analysis-custom-genome-reference
scRNA-seq analysis steps for custom genome reference

## 1: Extract mitochondrial sequence from a (old?) genome and append on to the most recent assembly

### 1.1: extract MT sequence from a genome reference .fa file
#### check which chromosomes are included in the assembly
```ruby
grep '>ssa\|>MT' Salmo_salar.ICSASG_v2.dna_sm.toplevel.fa 
#>ssa01 dna_sm:primary_assembly primary_assembly:ICSASG_v2:ssa01:1:159038749:1 REF
#>ssa02 dna_sm:primary_assembly primary_assembly:ICSASG_v2:ssa02:1:72943711:1 REF
#...
#...
#>ssa29 dna_sm:primary_assembly primary_assembly:ICSASG_v2:ssa29:1:42488238:1 REF
#>MT dna_sm:primary_assembly primary_assembly:ICSASG_v2:MT:1:16665:1 REF
```
#### extract MT sequence using samtools faidx
```ruby
samtools faidx Salmo_salar.ICSASG_v2.dna_sm.toplevel.fa
samtools faidx Salmo_salar.ICSASG_v2.dna_sm.toplevel.fa MT > MT.fa

# glimpse first four lines of the extracted MT sequence
head -4 MT.fa
#>MT
#ACGTTTCAGCTATGTACAACAATAAATGTTATATCTAGCTAACCCAATGTTATACTACAT
#CTATGTATAACATTACATATTATGTATTTACCCATATATATAATATCGCATGTGAGTAGT
#ACATTATATGTATTATCAACATAAGTGGATTTAACCCCTCATACATCAGCACTAATCCAA

#we can also count how many bases MT genome has
samtools faidx MT.fa
cut -f1-2 MT.fa.fai
#MT	16665
```
### 1.2: append MT sequence into another genome reference .fa file
#### lets first check which chromosomes are included in the assembly
```ruby
grep '>[0-9]\|MT' Salmo_salar.Ssal_v3.1.dna_sm.toplevel.fa 
#>1 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:1:1:174498729:1 REF
#>2 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:2:1:95481959:1 REF
#...
#...
#>29 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:29:1:43051128:1 REF
```
We can see that chr are in integer format (this doesn't matter) and MT sequence is missing

#### to keep it simple, we will extract chr from new assembly (to exclude scaffolds which start with ">CAJNNT")
```
samtools faidx Salmo_salar.Ssal_v3.1.dna_sm.toplevel.fa 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 > ssalar_select.fa
```

#### now append MT sequence onto your the genome .fa file
```
cat MT.fa >> ssalar_select.fa
```

#### sanity check MT chromosome added to the new assembly
```ruby
grep '>[0-9]\|MT' ssalar_select.fa 
#>1 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:1:1:174498729:1 REF
#>2 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:2:1:95481959:1 REF
#...
#...
#>29 dna_sm:primary_assembly primary_assembly:Ssal_v3.1:29:1:43051128:1 REF
#>MT dna_sm:primary_assembly primary_assembly:ICSASG_v2:MT:1:16665:1 REF
```
## 2: similarly, need to extract MT genes from old gtf file and append to new(er) gtf
```ruby
# extract MT genes
cat Salmo_salar.ICSASG_v2.105.gtf | awk '($1 == "MT") {print $0}' > MT.gtf

# append to newer gtf
cat MT.gtf >> Salmo_salar.Ssal_v3.1.108.gtf

# sanity check
grep 'MT' Salmo_salar.Ssal_v3.1.108.gtf
#MT	RefSeq	gene	1007	1074	.	+	.	gene_id "ENSSSAG00000000002"; gene_version "1"; gene_source "RefSeq"; gene_biotype "Mt_tRNA";
#MT	RefSeq	transcript	1007	1074	.	+	.	gene_id "ENSSSAG00000000002"; gene_version "1"; transcript_id "ENSSSAT00000000002"; transcript_version "1"; gene_source "RefSeq"; gene_biotype "Mt_tRNA"; transcript_source "RefSeq"; transcript_biotype "Mt_tRNA";
#...
#...

# save as new file name to reflect MT changes
cp Salmo_salar.Ssal_v3.1.108.gtf ssalar_MT.gtf
```
#### Now that we have added MT genes and sequence to .gtf and .fa files, respectively, we will use these files with MT entries for the rest of the scRNA-seq analysis
- ssalar_MT.gtf
- ssalar_select.fa
