# Preparação dos bancos de anotação

Vamos trabalhar com dois bancos de anotação: RefSeq e ClinVar.

## RefSeq

O RefSeq (Reference Sequence Database) é um banco de sequências biológicas mantido pelo NCBI. É utilizado como referência para a anotação biológica/funcional de variantes.

### 1. Download
Baixe o GFF e o arquivo com a definição dos nomes dos cromossomos.
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/refseq/H_sapiens/annotation/annotation_releases/105.20220307/GCF_000001405.25_GRCh37.p13/GCF_000001405.25_GRCh37.p13_genomic.gff.gz
wget ftp://ftp.ncbi.nlm.nih.gov/refseq/H_sapiens/annotation/annotation_releases/105.20220307/GCF_000001405.25_GRCh37.p13/GCF_000001405.25_GRCh37.p13_assembly_report.txt
```

### 2. Criação de lista de cromossomos
Uma etapa necessária para o VEP é criar um arquivo acessório contendo um de-para do nome do cromossomo no VCF ("20") para o identificador da sequência no RefSeq (no caso, "NC_000020.10"). Isso permite que o VEP faça anotações utilizando o NC_.

```bash
mkdir -p databases
grep "NC_" GCF_000001405.25_GRCh37.p13_assembly_report.txt | \
    awk -F'\t' -v OFS='\t' '$1=="20" {print $1,$7}' > databases/refseq_chromosomes.txt
```

### 3. Filtragem
Remova o cabeçalho e crie um GFF com anotações do cromossomo 20.
```bash
zgrep -v "^#" GCF_000001405.25_GRCh37.p13_genomic.gff.gz | grep ^NC_000020 > filtered.gff
```

Depois, ordene as posições para permitir a indexação.
```bash
sort -k1,1 -k4,4n -k5,5n -t$'\t' filtered.gff > sorted.gff
```

### 4. Compactação e indexação
Faça a compactação e indexação do arquivo final.
```bash
bgzip -c sorted.gff > databases/refseq_20.gff.gz
tabix -p gff databases/refseq_20.gff.gz
```

Uma vez concluído, faça a remoção dos arquivos iniciais e intermediários.
```bash
rm GCF_000001405.25_GRCh37.p13_* filtered.gff sorted.gff
```


## ClinVar

O ClinVar é um catálogo de variantes de interesse clínico mantido também pelo NCBI, mas alimentado por laboratórios, empresas e universidades do mundo inteiro. É utilizado para fazer a anotação de variantes por colocalização.

O NCBI disponibiliza as anotações do ClinVar em diversos formatos. Utilizaremos os VCFs como arquivos de entrada para gerar os VCFs finais que constituirão o banco de anotação.

### 1. Download
Baixe o VCF e o índice.
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh37/weekly/clinvar_20251201.vcf.gz
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh37/weekly/clinvar_20251201.vcf.gz.tbi
```

### 2. Filtragem do VCF
Utilize o bcftools para criar um VCF do cromossomo 20.
```bash
bcftools view clinvar_20251201.vcf.gz -r 20 -o clinvar_filt.vcf
```

### 3. Decomposição e normalização de variantes
Utilize o vt para fazer a decomposição das variantes. Essa etapa separa variantes multi-alélicas em linhas distintas do VCF.

```bash
vt decompose clinvar_filt.vcf -o clinvar_decomp.vcf
```

Na sequência, faça a normalização das variantes para fazer o alinhamento à esquerda de indels e remover bases comuns no REF e no ALT.
```bash
vt normalize clinvar_decomp.vcf \
    -w 2000000 \
    -r references/human_g1k_v37_20.fasta \
    -o clinvar_norm.vcf
```

### 4. Compactação e indexação
Faça a compactação e indexação do arquivo final.
```bash
mkdir -p databases
bgzip -c clinvar_norm.vcf > databases/clinvar_20.vcf.gz
tabix -p vcf databases/clinvar_20.vcf.gz
```

Uma vez concluído, remova os arquivos iniciais e intermediários.
```bash
rm clinvar_20251201.vcf.gz* clinvar_*.vcf
```
