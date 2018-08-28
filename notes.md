<header>
<!--GWAS <img src="" height="40px"/> -->
<h1> GWAS </h1>
</header>

INPUTS :
* chr.1.dose.vcf.gz
* chr.2.dose.vcf.gz
* chr.3.dose.vcf.gz   
...
* chr.22.dose.vcf.gz
```bash
##fileformat=VCFv4.1
##filedate=2017.3.22
##source=Minimac3
##contig=<ID=1>
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
##FORMAT=<ID=DS,Number=1,Type=Float,Description="Estimated Alternate Allele Dosage : [P(0/1)+2*P(1/1)]">
##FORMAT=<ID=GP,Number=3,Type=Float,Description="Estimated Posterior Probabilities for Genotypes 0/0, 0/1 and 1/1 ">
##INFO=<ID=AF,Number=1,Type=Float,Description="Estimated Alternate Allele Frequency">
##INFO=<ID=MAF,Number=1,Type=Float,Description="Estimated Minor Allele Frequency">
##INFO=<ID=R2,Number=1,Type=Float,Description="Estimated Imputation Accuracy">
##INFO=<ID=ER2,Number=1,Type=Float,Description="Empirical (Leave-One-Out) R-square (available only for genotyped variants)">
#CHROM  POS   ID      REF  ALT  QUAL  FILTER  INFO                               FORMAT   1_534                        2_542                        3_546...                       582_1588
1       13380 1:13380 C    G    .     PASS    AF=0.00000;MAF=0.00000;R2=0.00021  GT:DS:GP 0/0:0.000:1.000,0.000,0.000  0/0:0.000:1.000,0.000,0.000  0/0:0.000:1.000,0.000,0.000... 0/0:0.000:1.000,0.000,0.000
```
**Objectifs**
* [ ] Diviser les vcf 1 par échantillon 
* [ ] Concatener tous les chromosomes pour un échantillon   

> Attention !   
> Garder les entêtes des fichiers.   
> Ajouter les lignes ##contig=<ID=1>, ##contig=<ID=2>, ##contig=<ID=3>..., ##contig=<ID=22>

**Démarche**

Outil pour diviser les VCF : [biostar130456](http://lindenb.github.io/jvarkit/Biostar130456.html)   
Même que celui utilisé pour 1000G, voir justification du choix dans la section 1000G   
du fichier notebook.md dans le répertoire git Ethnicity_Filter.   
*Emplacement : /home/jacques_group/Frederique/LevesqueS/databases/1000G/jvarkit/dist/biostar130456.jar*

```bash
# Division des VCF
# Utilisation biostar130456
echo "ml jdk64/8u74 && \
  cd /home/jacques_group/Frederique/BouchardL/gwas/ &&\
  cat chr.1.dose.vcf | java -Xmx25G -jar biostar130456.jar \
  --disable-vc-attribute-recalc \
  --vc-attribute-recalc-ignore-filtered \
  --vc-attribute-recalc-ignore-missing \
  -x -z -p "splitted_vcf/sample.__SAMPLE__.chrMT.vcf.gz" " | \
  qsub -q qwork@mp2 -N split.chr1 -l walltime=2:00:00 -l nodes=1 

# Concatener les chromosomes d'un même échantillon
nohup sh -c 'for ID in ID.1000Gindividual ; do
	cat sample.${ID}* >output/sample.${ID}.vcf.gz && \
	 gzip -d output/sample.${ID}.vcf.gz && \
   grep "#" output/sample.${ID}.vcf >header && \ 
	 grep -v "#" output/sample.${ID}.vcf > output/sample.${ID}.sans_header.vcf && \
   cat header output/sample.${ID}.sans_header.vcf > samples/sample.${ID}.vcf ;done' &
