# Preparação do ambiente

Pré-requisitos:
1. Docker instalado (https://www.docker.com/get-started/)
2. Acesso à internet

## 1. Clonagem do projeto
Comece clonando o repositório para seu ambiente de execução e entrando nele pela linha de comando.

```bash
git clone https://github.com/gabrielhim/curso-anotacao-variantes.git
cd curso-anotacao-variantes
```

## 2. Montagem da image Docker
As ferramentas de bioinformática utilizadas neste curso serão providenciadas dentro de um container Docker. Instanciaremos um container contendo essas ferramentas e rodaremos todos os comandos de bioinformática lá dentro.

Para começar, precisamos fazer o build da imagem:

```bash
docker build -t curso-anotacao:v1 docker
```

Esse comando utiliza o Dockerfile na pasta **docker** como base para criar a imagem, que será identificada como `curso-anotacao:v1`. Essa imagem contém um ambiente Linux com o conda e quatro ferramentas de bioinformática:
* bcftools v1.20
* samtools v1.20
* vt v0.57721
* VEP v115.2

O processo de montagem da imagem leva alguns minutos.

## 3. Criação do container
Uma vez concluído, podemos iniciar um container. Certifique-se que você está na pasta raíz do projeto do curso e rode o seguinte comando:

```bash
docker run --rm -it -v .:/rsg-brazil -w /rsg-brazil curso-anotacao:v1
```

Esse comando instancia um container do Docker utilizando a imagem `curso-anotacao:v1`. O parâmetro `-it` nos permite entrar no container e rodar comandos de forma interativa. O `-v` define um ponto de montagem para compartilhamento de arquivos, que é o diretório de execução no lado do host e a pasta `/rsg-brazil` dentro do container. O `-w` define o diretório de execução.

Feito isso, você tem um ambiente para rodar os comandos de bioinformática. Faça um teste:
```bash
root@55cda44269a4:/rsg-brazil# bcftools --help
```
