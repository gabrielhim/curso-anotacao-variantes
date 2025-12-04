# Preparação do genoma de referência

O genoma de referência escolhido para esse curso foi o hs37-1kg. Esse genoma de referência segue boas práticas para análises de NGS e pode ser facilmente baixado da internet. A seguir estão as etapas de preparação dele.

## 1. Download
Comece baixando o FASTA e o índice do repositório de dados do 1000 Genomes.
```bash
wget ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/reference/human_g1k_v37.fasta.gz
wget ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/reference/human_g1k_v37.fasta.fai
```

## 2. Descompactação
Utilize esses comandos para descomprimir o FASTA.
```bash
truncate -s 891946027 human_g1k_v37.fasta.gz
gunzip human_g1k_v37.fasta.gz
```

## 3. Filtragem
Crie uma pasta para armazenar a referência. Depois, utilize o samtools para criar um arquivo FASTA apenas com o cromossomo 20.
```bash
mkdir -p references
samtools faidx human_g1k_v37.fasta 20 > references/human_g1k_v37_20.fasta
```

## 4. Criação do índice
Por fim, crie o índice do FASTA. 
```bash
samtools faidx references/human_g1k_v37_20.fasta
```

Remova os arquivos originais.
```bash
rm human_g1k_v37.fasta*
```
