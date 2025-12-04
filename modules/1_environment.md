# Preparação do ambiente

As ferramentas de bioinformática utilizadas neste curso serão providenciadas dentro de um container Docker. Instanciaremos um container contendo essas ferramentas e rodaremos todos os comandos de bioinformática lá dentro.

Pré-requisitos:
1. Docker instalado (https://www.docker.com/get-started/).
2. Acesso à internet.

## 1. Clonagem do projeto
Comece clonando o repositório para seu ambiente de execução e entrando nele pela linha de comando.

```bash
git clone https://github.com/gabrielhim/curso-anotacao-variantes.git
cd curso-anotacao-variantes
```

## 2. Montagem da imagem Docker
Faça o build da imagem:
```bash
docker build -t curso-anotacao:v1 docker
```

O processo de montagem da imagem leva alguns minutos. Esse comando utiliza o Dockerfile na pasta **docker** como base para criar a imagem, que será identificada como `curso-anotacao:v1`. Essa imagem contém um ambiente Linux com o conda e quatro ferramentas de bioinformática:
* bcftools v1.20
* samtools v1.20
* vt v0.57721
* VEP v115.2

## 3. Criação do container
Uma vez concluído, podemos iniciar um container. Certifique-se que você está na pasta raíz do projeto do curso e rode o seguinte comando:
```bash
docker run --rm -it -v .:/rsg-brazil -w /rsg-brazil curso-anotacao:v1
```

Esse comando instancia um container do Docker utilizando a imagem `curso-anotacao:v1`. A seguir está uma descrição dos parâmetros:
* `--rm`: deleta o container após sua utilização.
* `-it`: nos permite entrar no container e rodar comandos de forma interativa no shell.
* `-v`: define um ponto de montagem para compartilhamento de arquivos. No caso, o diretório de execução no host (`.`) é mapeado para a pasta `/rsg-brazil` dentro do container.
* `-w` define o diretório de execução.

Feito isso, você tem um ambiente para rodar os comandos de bioinformática. Faça um teste:
```bash
bcftools --help
```

Caso queira sair do container, basta digitar `exit` e Enter.
