# Preparação de uma amostra teste

Este curso utiliza o VCF HG005.singleSampleVC.filter.vcf do Genomes in a Bottle (GIAB) como input. Ele foi gerado com o genoma de referência hs37d5, que é compatível com o genoma disponível neste projeto.

## 1. Download
Baixe o arquivo utilizando o wget:
```bash
wget https://giab.s3.us-east-1.amazonaws.com/data/ChineseTrio/analysis/OsloUniversityHospital_Exome_GATK_jointVC_11242015/HG005.singleSampleVC.filter.vcf
```

## 2. Compactação e indexação
Na sequência, compacte o arquivo com o bgzip e use o tabix (baixado automaticamente com o bcftools) para indexar o VCF:
```bash
bgzip HG005.singleSampleVC.filter.vcf
tabix -p vcf HG005.singleSampleVC.filter.vcf.gz
```

## 3. Filtragem
Crie uma pasta para manter a amostra e utilize o bcftools para criar um VCF apenas com o cromossomo 20.
```bash
mkdir -p samples
bcftools view HG005.singleSampleVC.filter.vcf.gz -r 20 -o samples/HG005_exome_20.vcf.gz
```

Remova os arquivos originais.
```bash
rm HG005.singleSampleVC.filter.vcf*
```
