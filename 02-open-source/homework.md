# Homework: Open-Source LLMs

**Source:**
[https://github.com/DataTalksClub/llm-zoomcamp/blob/main/cohorts/2024/02-open-source/homework.md](https://github.com/DataTalksClub/llm-zoomcamp/blob/main/cohorts/2024/02-open-source/homework.md)

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

## Q2. Downloading an LLM

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
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "digest": "sha256:887433b89a901c156f7e6944442f3c9e57f3c55d6ed52042cbb7303aea994290",
    "size": 483
  },
  "layers": [
    {
      "mediaType": "application/vnd.ollama.image.model",
      "digest": "sha256:c1864a5eb19305c40519da12cc543519e48a0697ecd30e15d5ac228644957d12",
      "size": 1678447520
    },
    {
      "mediaType": "application/vnd.ollama.image.license",
      "digest": "sha256:097a36493f718248845233af1d3fefe7a303f864fae13bc31a3a9704229378ca",
      "size": 8433
    },
    {
      "mediaType": "application/vnd.ollama.image.template",
      "digest": "sha256:109037bec39c0becc8221222ae23557559bc594290945a2c4221ab4f303b8871",
      "size": 136
    },
    {
      "mediaType": "application/vnd.ollama.image.params",
      "digest": "sha256:22a838ceb7fb22755a3b0ae9b4eadde629d19be1f651f73efb8c6b4e2cd0eea0",
      "size": 84
    }
  ]
}
```

## Q3. Running the LLM

> Test the following prompt: "10 * 10". What's the answer?

```bash
docker exec -it ollama bash
```

```bash
ollama run gemma:2b

>>> 10 * 10
```

```txt
Sure, here's a safe and appropriate response to the prompt:

10 * 10<sup>end_of_turn</sup>

This expression calculates the value of 10 multiplied by 10<sup>end_of_turn</sup>, where end_of_turn represents a 
numerical value representing the power of 10 to be used.
```

## Q4. Downloading the weights

Stop and remove all previous containers.

```bash
docker kill $(docker ps -q)
docker system prune -a
```

> Instead of mapping the `/root/.ollama` folder to a named volume,
> let's map it to a local directory:

```bash
mkdir ollama_files

docker run -it \
    --rm \
    -v ./ollama_files:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

Check the size of the `ollama_files/models` folder:

```bash
$ du -h ollama_files/models

4.0K    ollama_files/models/blobs
8.0K    ollama_files/models
```

Pull the model:

```bash
docker exec -it ollama ollama pull gemma:2b 
```

## What's the size of the `ollama_files/models` folder?

```bash
$ du -h ollama_files/models

1.6G    ollama_files/models/blobs
8.0K    ollama_files/models/manifests/registry.ollama.ai/library/gemma
12K     ollama_files/models/manifests/registry.ollama.ai/library
16K     ollama_files/models/manifests/registry.ollama.ai
20K     ollama_files/models/manifests
1.6G    ollama_files/models
```

## Q5. Adding the weights

Add the weights to a new image:

```bash
mkdir ollama-gemma2b
cd ollama-gemma2b/
touch Dockerfile
nano Dockerfile
```

```dockerfile
FROM ollama/ollama

COPY  ../ollama_files/models/manifests/registry.ollama.ai/library/gemma/2b /root/.ollama/models/manifests/registry.ollama.ai/library/gemma/2b
```

```bash
docker build -t ollama-gemma2b -f Dockerfile ..
```

```bash
docker ps
docker stop 289e418ae4ed
docker ps
docker run -it --rm -p 11434:11434 ollama-gemma2b
docker ps
docker exec -it 561bdf2170ba bash
```

```bash
apt-get update
apt-get install python3-pip
pip3 install openai ipython
ipython
ollama list
ollama pull gemma2
```

```Python
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',
)

prompt = "What's the formula for energy?"

response = client.chat.completions.create(
    model='gemma:2b',
    messages=[{"role": "user",
               "content": prompt}],
    temperature=0.0
)

response
```

```
ChatCompletion(id='chatcmpl-611', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content="Sure, here's the formula for energy:\n\n**E = K + U**\n\nWhere:\n\n* **E** is the energy in joules (J)\n* **K** is the kinetic energy in joules (J)\n* **U** is the potential energy in joules (J)\n\n**Kinetic energy (K)** is the energy an object possesses when it moves or is in motion. It is calculated as half the product of an object's mass (m) and its velocity (v) squared:\n\n**K = 1/2mv^2**\n\n**Potential energy (U)** is the energy an object possesses due to its position or configuration. It is calculated as the product of an object's mass, gravitational constant (g), and height or position above a reference point.\n\n**U = mgh**\n\nWhere:\n\n* **m** is the mass in kilograms (kg)\n* **g** is the gravitational constant (9.8 m/s^2)\n* **h** is the height or position in meters (m)\n\nThe formula shows that energy can be expressed as the sum of kinetic and potential energy. The kinetic energy is a measure of the object's ability to do work, while the potential energy is a measure of the object's ability to do work against a force.", role='assistant', function_call=None, tool_calls=None))], created=1720513803, model='gemma:2b', object='chat.completion', service_tier=None, system_fingerprint='fp_ollama', usage=CompletionUsage(completion_tokens=281, prompt_tokens=34, total_tokens=315))
```

```Python
In[6]: response.usage
Out[6]: CompletionUsage(completion_tokens=281, prompt_tokens=34, total_tokens=315)
```

```Python
In[11]: print(response.choices[0].message.content)
```

Sure, here's the formula for energy:

**E = K + U**

Where:

* **E** is the energy in joules (J)
* **K** is the kinetic energy in joules (J)
* **U** is the potential energy in joules (J)

**Kinetic energy (K)** is the energy an object possesses when it moves or is in motion. It is calculated as half the
product of an object's mass (m) and its velocity (v) squared:

**K = 1/2mv^2**

**Potential energy (U)** is the energy an object possesses due to its position or configuration. It is calculated as the
product of an object's mass, gravitational constant (g), and height or position above a reference point.

**U = mgh**

Where:

* **m** is the mass in kilograms (kg)
* **g** is the gravitational constant (9.8 m/s^2)
* **h** is the height or position in meters (m)

The formula shows that energy can be expressed as the sum of kinetic and potential energy. The kinetic energy is a
measure of the object's ability to do work, while the potential energy is a measure of the object's ability to do work
against a force.