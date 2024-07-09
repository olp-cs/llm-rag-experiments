# Homework: Open-Source LLMs

----------------

**Source:** [https://github.com/DataTalksClub/llm-zoomcamp/blob/main/cohorts/2024/02-open-source/homework.md](https://github.com/DataTalksClub/llm-zoomcamp/blob/main/cohorts/2024/02-open-source/homework.md)

-------------

## Q1. Running Ollama with Docker

GitHub CodeSpaces -> New with options -> Machine type: 4 cores

```bash
docker run -it \
    --rm \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

## What's the version of ollama client?
```bash
$ docker ps
CONTAINER ID   IMAGE           COMMAND               CREATED          STATUS          PORTS                                           NAMES
fc327446d2a3   ollama/ollama   "/bin/ollama serve"   18 minutes ago   Up 18 minutes   0.0.0.0:11434->11434/tcp, :::11434->11434/tcp   ollama

$ docker exec -it ollama bash
```
```bash
ollama -v
```
ollama version is 0.2.0

----

## Q2. Downloading an LLM 

-----

We will donwload a smaller LLM - gemma:2b. 

Again let's enter the container and pull the model:

```bash
docker exec -it ollama bash
```

```bash
ollama -v
ollama pull gemma:2b

cd root
cd .ollama/models/manifests/registry.ollama.ai/library
cd gemma
cat 2b
```

## Content of the file related to gemma

```json
{"schemaVersion":2,"mediaType":"application/vnd.docker.distribution.manifest.v2+json","config":{"mediaType":"application/vnd.docker.container.image.v1+json","digest":"sha256:887433b89a901c156f7e6944442f3c9e57f3c55d6ed52042cbb7303aea994290","size":483},"layers":[{"mediaType":"application/vnd.ollama.image.model","digest":"sha256:c1864a5eb19305c40519da12cc543519e48a0697ecd30e15d5ac228644957d12","size":1678447520},{"mediaType":"application/vnd.ollama.image.license","digest":"sha256:097a36493f718248845233af1d3fefe7a303f864fae13bc31a3a9704229378ca","size":8433},{"mediaType":"application/vnd.ollama.image.template","digest":"sha256:109037bec39c0becc8221222ae23557559bc594290945a2c4221ab4f303b8871","size":136},{"mediaType":"application/vnd.ollama.image.params","digest":"sha256:22a838ceb7fb22755a3b0ae9b4eadde629d19be1f651f73efb8c6b4e2cd0eea0","size":84}]}
```
