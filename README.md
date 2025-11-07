# 🚀 CI/CD e GitOps na Prática com FastAPI, Docker Hub e ArgoCD

> Projeto de **automação completa do ciclo de desenvolvimento** com **FastAPI**, **GitHub Actions**, **Docker Hub**, **Kubernetes (Rancher Desktop)** e **ArgoCD**, aplicando os conceitos modernos de **CI/CD** e **GitOps**.

<br>

<p align="center">
  <img src="https://skillicons.dev/icons?i=python,fastapi,docker,github,kubernetes" />
  <img src="https://argo-cd.readthedocs.io/en/stable/assets/logo.png" width="48" height="48" alt="ArgoCD"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/rancher/rancher-original.svg" width="48" height="48" alt="Rancher Desktop"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/yaml/yaml-original.svg" width="48" height="48" alt="YAML"/>
</p>

<br>

## 🧭 Introdução

O desenvolvimento moderno exige **integração contínua (CI)** e **entrega contínua (CD)** para garantir agilidade, qualidade e segurança no deploy de aplicações.  

Neste projeto, foi construído um pipeline completo que faz **build, publicação e deploy automático** de uma aplicação **FastAPI** em um cluster **Kubernetes local**, seguindo o modelo **GitOps** com **ArgoCD**.

A estrutura combina as seguintes ferramentas:

- 🐍 **FastAPI** — backend em Python
- ⚙️ **GitHub Actions** — CI/CD para build e push automático
- 🐳 **Docker Hub** — container registry público
- ☸️ **Kubernetes (Rancher Desktop)** — cluster local
- 🔁 **ArgoCD** — sincronização GitOps para deploy automático

<br>

## 🎯 Objetivo

Automatizar **todo o ciclo de desenvolvimento**, desde o commit do código até o deploy no Kubernetes, utilizando um fluxo totalmente controlado por Git:

```bash
+------------------+       +------------------+       +--------------------+       +-------------------+
|    hello-app     |       |   Docker Hub     |       |   hello-manifests  |       |     ArgoCD        |
| (FastAPI + CI/CD)| ----> | (Container Repo) | ----> | (K8s Manifests Git)| ----> | (Sync no Cluster) |
+------------------+       +------------------+       +--------------------+       +-------------------+
```

<br>

## ⚙️ Pré-requisitos

Antes de começar, garanta que as ferramentas abaixo estejam instaladas e configuradas:

| Ferramenta | Função | Verificação |
|-------------|--------|-------------|
| **Python 3** | Executar o FastAPI localmente | `python --version` |
| **Docker** | Buildar e publicar imagens | `docker ps` |
| **Git** | Versionamento e controle de código | `git --version` |
| **GitHub** | Hospedagem dos repositórios | Conta criada |
| **Rancher Desktop** | Kubernetes local | `kubectl get nodes` |
| **ArgoCD** | Entrega contínua GitOps | Instalado no cluster |

<br>

## 🧩 Estrutura dos Repositórios

Este projeto utiliza **dois repositórios GitHub**:

### 1️⃣ Repositório [`hello-app`](https://github.com/pedro-albertini/hello-app)
Contém:
- Aplicação FastAPI [`main.py`](main.py)
- [`Dockerfile`](Dockerfile)
- Workflow [`.github/workflows/main.yml`](https://github.com/pedro-albertini/hello-app/blob/main/.github/workflows/main.yaml)

Responsável pela primeira etapa do processo:
- Buildar e publicar imagens no Docker Hub  
- Atualizar o repositório de manifests (`hello-manifests`)

### 2️⃣ Repositório [`hello-manifests`](https://github.com/pedro-albertini/hello-manifests)
Contém os arquivos Kubernetes:
- [`deployment.yaml`](https://github.com/pedro-albertini/hello-manifests/blob/main/deployment.yaml)
- [`service.yaml`](https://github.com/pedro-albertini/hello-manifests/blob/main/service.yaml)

Responsável por:
- Armazenar os manifests observados pelo ArgoCD  
- Garantir o modelo GitOps, onde o **Git é a fonte da verdade**

<br>

## ⚙️ Etapa 1 – Criar a aplicação FastAPI

Crie um novo repositório no seu GitHub chamado por exemplo `hello-app` 

Dentro desse novo repoistório, crie um arquivo python `main.py` para colocar sua API:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello World"}
```

Crie o arquivo `requirements.txt`, que vai servir para listar todas as dependências para o seu projeto e ajudar no processo de automação:

```
fastapi
uvicorn
```

Crie o Dockerfile para containerizar a aplicação:

```
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

<br>

## 🔁 Etapa 2 – Configurar CI/CD no GitHub Actions

Crie o arquivo .github/workflows/main.yml no repositório hello-app:

```
name: CI/CD

on:
    push:
        branches: [ "main" ] 
    pull_request:
        branches: [ "main" ]

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: Login Docker Hub
              uses: docker/login-action@v3
              with:
                username: ${{ secrets. DOCKER_USERNAME }}
                password: ${{ secrets.DOCKER_PASSWORD }}
            - name: Build e push da imagem para o Docker Hub
              run: |
                IMAGE=${{ secrets.DOCKER_USERNAME }}/hello-app
                TAG=$(date +%s)
                docker build -t $IMAGE:$TAG .
                docker push $IMAGE:$TAG
                echo "IMAGE_TAG=$TAG" >> $GITHUB_ENV
            - name: Clonar e atualizar manifests via SSH
              uses: actions/checkout@v4
              with:
                repository: <Seu GitHub>/hello-manifests
                ssh-key: ${{ secrets.SSH_PRIVATE_KEY }}
                path: hello-manifests
            - name: Atualizar imagem no deployment.yaml
              run: |
                cd hello-manifests
                sed -i "s#image: .*\$#image: ${{ secrets.DOCKER_USERNAME }}/hello-app:${{ env.IMAGE_TAG }}#" deployment.yaml
                git config --global user.email "github-actions@github.com"
                git config --global user.name "GitHub Actions"
                git add deployment.yaml
                git commit -m "Update image tag to ${{ env.IMAGE_TAG }}"
                git push origin main
```

Agora adicione os secrets necessários no GitHub, acessando:
Settings → Secrets and Variables → Actions

| Nome              | Valor                                                                    |
| ----------------- | ------------------------------------------------------------------------ |
| `DOCKER_USERNAME` | seu usuário do Docker Hub                                                |
| `DOCKER_PASSWORD` | sua senha ou token do Docker Hub                                                  |
| `SSH_PRIVATE_KEY` | chave privada SSH (com “Allow write access” no repositório de manifests) |

Esses valores serão usados para login no Docker Hub e atualização automática do repositório de manifests.

<br>

### 🔑 Gerando e configurando a chave SSH

A autenticação SSH é usada pelo GitHub Actions para conseguir escrever no repositório hello-manifests.

1️⃣ Gerar uma nova chave SSH na sua máquina

No terminal:

```
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
```

Pressione Enter para aceitar o caminho padrão (~/.ssh/id_ed25519).
Isso criará dois arquivos:

- id_ed25519 → chave privada
- id_ed25519.pub → chave pública

2️⃣ Adicionar a chave pública no repositório hello-manifests

Acesse:
Settings → Deploy keys → Add deploy key

- Title: CI Access
- Key: conteúdo do arquivo id_ed25519.pub
- Marque ✅ Allow write access

Clique em Add key

3️⃣ Adicionar a chave privada no repositório hello-app

Acesse:
Settings → Secrets and Variables → Actions → New repository secret

- Name: SSH_PRIVATE_KEY
- Value: conteúdo completo do arquivo id_ed25519 (sem espaços extras)

Clique em Add secret

Após isso, o GitHub Actions terá permissão para atualizar automaticamente o repositório de manifests sempre que uma nova imagem for criada.

<br>

### 💡 Função do Workflow

O workflow realiza automaticamente:

- Build da imagem Docker da aplicação.
- Push dessa imagem para o Docker Hub.
- Atualização automática do repositório hello-manifests com a nova tag da imagem.

Após um novo commit, o ArgoCD sincroniza automaticamente o deploy no cluster.

<br>

## 🧪 Etapa 3 – Testar a aplicação localmente

No terminal, entre na pasta do seu projeto onde está localizado o seu Dockerfile.

Crie a imagem Docker:
```
docker build -t hello-app .
```

Rode o container:
```
docker run -p 8000:8000 hello-app
```

Acesse:
🔗 http://localhost:8000

Saída esperada:

| <img width="1913" height="968" alt="image" src="https://github.com/user-attachments/assets/ed5f9521-3a52-48a9-bd44-29699be3c97c" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Rodando* |

<br>

## 🧾 Conclusão

Este repositório representa a primeira parte do fluxo CI/CD + GitOps, responsável por:

- Construir e publicar a imagem Docker no Docker Hub;
- Atualizar automaticamente o repositório de manifests usado pelo ArgoCD;
- Garantir que o ciclo de deploy seja automático, versionado e reproduzível.

<br>

## 📦 Continuação do Projeto

A segunda parte deste projeto está no repositório:

🔗 [`hello-manifests`](https://github.com/pedro-albertini/hello-manifests/tree/main?tab=readme-ov-file)

Lá estão os manifests Kubernetes monitorados pelo ArgoCD, que realiza a sincronização automática da aplicação no cluster sempre que este repositório é atualizado pelo pipeline do hello-app.

---
🧑‍💻 Desenvolvido por [Pedro Albertini Fernandes Pinto](https://github.com/pedro-albertini) 
Projeto prático do módulo **Automação CI/CD e GitOps com FastAPI e ArgoCD**


