FROM --platform=linux/amd64 continuumio/miniconda3:24.3.0-0

RUN conda config --add channels bioconda \
    && conda config --add channels conda-forge \
    && conda install -y -c bioconda -c conda-forge \
    bcftools=1.20 \
    samtools=1.20 \
    ensembl-vep=115.2 \
    vt=0.57721 \
    && conda clean -afy
