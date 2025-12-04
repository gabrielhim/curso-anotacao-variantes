# Anotação de variantes

Nesta etapa, utilizaremos os arquivos preparados nas etapas anteriores para testar a anotação de variantes.

## 1. Decomposição e normalização de variantes da amostra
Como fizemos com o ClinVar, precisamos decompor e normalizar as variantes de input.
```bash
vt decompose samples/HG005_exome_20.vcf.gz -o HG005_exome_20.decomp.vcf

vt normalize HG005_exome_20.decomp.vcf \
    -r references/human_g1k_v37_20.fasta \
    -o HG005_exome_20.norm.vcf.gz

tabix -p vcf HG005_exome_20.norm.vcf.gz
```

## 2. Anotação com bcftools
O bcftools possui o módulo `annotate` que permite fazer a anotação de bancos pontuais em formato VCF ou BED.

```bash
bcftools annotate \
    HG005_exome_20.norm.vcf.gz \
    -a databases/clinvar_20.vcf.gz \
    -c 'INFO/CLNREVSTAT,INFO/CLNSIG,INFO/CLNSIGCONF' \
    -o HG005_exome_20.bcftools.vcf
```

Ele recebe um parâmetro `-a` com a base de dados de anotação (no caso, o ClinVar criado anteriormente) e um parâmetro `-c` especificando os campos de interesse. As anotações são adicionadas à coluna INFO do VCF da amostra.

O bcftools não suporta anotações biológicas, então não conseguimos usar o RefSeq com ele.

## 3. Anotação com o VEP
O VEP é uma das ferramentas de anotação mais utilizadas da atualidade e oferece uma quantidade imensa de configurações. Por padrão, ele utiliza como referência o cacheum arquivo compactado que agrega anotações das mais diversas fontes. Mas ele é pesado (~25 GB), então utilizaremos um GFF como referência para anotação biológica e o ClinVar como anotação customizada.

A estrutura do comando básico é essa:
```bash
vep \
    --input_file HG005_exome_20.norm.vcf.gz \
    --assembly GRCh37 \
    --custom file=databases/clinvar_20.vcf.gz,format=vcf,short_name=clinvar,type=exact,fields=CLNREVSTAT%CLNSIG%CLNSIGCONF \
    --fasta references/human_g1k_v37_20.fasta \
    --force_overwrite \
    --format vcf \
    --gff databases/refseq_20.gff.gz \
    --quiet \
    --synonyms databases/refseq_chromosomes.txt \
    --tab \
    --output_file HG005_exome_20.vep.tsv
```

Há alguns parâmetros chave:
* `--custom`: define bases de dados customizadas em formato VCF, GFF, GTF ou BED, bem como os campos de interesse.
* `--gff`: recebe o GFF com as anotações biológicas que substitui o cache do VEP.
* `--synonyms`: traz o de-para de nomes de cromossomos entre o VCF e o GFF.

O comando pode ser incrementado. Por exemplo, se o interesse for anotar apenas o HGVS:
```bash
vep \
    --input_file HG005_exome_20.norm.vcf.gz \
    --assembly GRCh37 \
    --fasta references/human_g1k_v37_20.fasta \
    --force_overwrite \
    --format vcf \
    --hgvs \
    --hgvsg \
    --hgvsg_use_accession \
    --gff databases/refseq_20.gff.gz \
    --quiet \
    --synonyms databases/refseq_chromosomes.txt \
    --tab \
    --output_file HG005_exome_20.vep.hgvs.tsv
```

Você pode explorar outras flags. Segue a sugestão de algumas delas:
* `--symbol`: inclui o símbolo do gene.
* `--numbers`: identifica o número do íntron ou éxon.
* `--variant_class`: inclui a classe da variante de acordo com o Sequence Ontology.

Teste também outros formatos de output.
