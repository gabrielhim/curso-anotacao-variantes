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
O VEP é uma das ferramentas de anotação mais utilizadas da atualidade e oferece uma quantidade imensa de configurações. Por padrão, ele utiliza como referência o cache, um arquivo compactado que agrega anotações das mais diversas fontes. Mas ele é pesado (~25 GB), então utilizaremos um GFF como referência para anotação biológica e o ClinVar como anotação customizada.

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
* `--gff`: recebe o GFF com as anotações biológicas que substitui o cache do VEP. Sem ele, o VEP procura pelo cache no diretório de execução.
* `--synonyms`: traz o de-para de nomes de cromossomos entre o VCF e o GFF.

São gerados três outputs:
1. Arquivo anotado no formato especificado na linha de comando.
2. Um HTML com métricas (que pode não ser gerado com a opção `--no_stats`).
3. Um arquivo com warnings.

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

Dica: note o aumento de tempo de execução com essas flags. Isso mostra a complexidade de anotar o HGVS.

Você pode explorar outras flags. Segue a sugestão de algumas delas:
* `--symbol`: inclui o símbolo do gene.
* `--numbers`: identifica o número do íntron ou éxon.
* `--variant_class`: inclui a classe da variante de acordo com o Sequence Ontology.

Teste também outros formatos de output alternando entre `--tab`, `--vcf` e `--json`. O formato JSON é interessante por ser estruturado e dar segurança na hora de consumir o conteúdo do arquivo usando ferramentas.

:warning: O VEP possui uma lista limitada de tipos de sequência suportadas do GFF, como CDS, miRNA, mRNA, pseudogene e tRNA (acesse https://www.ensembl.org/info/docs/tools/vep/script/vep_cache.html#gff para ver a lista completa). Regiões com tipos fora dessa lista, como enhancer, protein_binding_site e antisense_RNA, são ignoradas e vêm identificadas no arquivo de warnings.
