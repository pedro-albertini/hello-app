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

### 1️⃣ Repositório `hello-app`
Contém:
- Aplicação FastAPI (`main.py`)
- `Dockerfile`
- Workflow (`.github/workflows/main.yml`)

Responsável por:
- Buildar e publicar imagens no Docker Hub  
- Atualizar o repositório de manifests (`hello-manifests`)

### 2️⃣ Repositório `hello-manifests`
Contém os arquivos Kubernetes:
- `deployment.yaml`
- `service.yaml`

Responsável por:
- Armazenar os manifests observados pelo ArgoCD  
- Garantir o modelo GitOps, onde o **Git é a fonte da verdade**

<br>

## ⚙️ Etapa 1 – Criar a aplicação FastAPI

Crie um novo repositório no seu GitHub chamado por exemplo `hello-app` 

Dentro desse novo repoistório, crie um arquivo python [`main.py`](main.py) para colocar sua API:

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

## ⚙️ Etapa 2 – Configurar CI/CD no GitHub Actions

Crie o arquivo .github/workflows/main.yml no repositório hello-app:

Agora adicione os secrets necessários no GitHub, acessando:
Settings → Secrets and Variables → Actions

| Nome              | Valor                                                                    |
| ----------------- | ------------------------------------------------------------------------ |
| `DOCKER_USERNAME` | seu usuário do Docker Hub                                                |
| `DOCKER_PASSWORD` | sua senha ou token do Docker Hub                                                  |
| `SSH_PRIVATE_KEY` | chave privada SSH (com “Allow write access” no repositório de manifests) |

Esses valores serão usados para login no Docker Hub e atualização automática do repositório de manifests.

<br>

## 🧱 Etapa 3 – Criar os manifests do Kubernetes

Crie um novo repositório chamado por exemplo de hello-manifests e adicione os arquivos de manifesto do kubernetes:

deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
        - name: hello-app
          image: <Seu Docker Hub>/hello-app:latest
          ports:
            - containerPort: 8000

```

service.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8000
  type: ClusterIP

```
<br>

## ☸️ Etapa 4 – Configurar o ArgoCD

Primeiro no terminal, instale o ArgoCD:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verifique se todos os pods estão rodando:

```
kubectl get pods -n argocd
```

Crie o port-forward para acessa-lo:

```
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Acesse no navegador:
🔗 http://localhost:8081

E você verá uma página assim:

| <img width="1914" height="540" alt="image" src="https://github.com/user-attachments/assets/e0b237f1-3ba2-4c50-9e05-308189c35541" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Painel ArgoCD* |

Credenciais padrão:

- Usuário: admin

- Senha: (use o comando abaixo para descobrir)

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```


##  🌐 Etapa 5 - Criar o app no ArgoCD

No painel do ArgoCD, clique em NEW APP

Preencha os campos de acordo com a tabela a seguir:

| Campo                | Valor                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Application Name** | hello-app                                                                                                        |
| **Project**          | default                                                                                                          |
| **Repository URL**   | [https://github.com/pedro-albertini/hello-manifests.git](https://github.com/pedro-albertini/hello-manifests.git) |
| **Revision**         | HEAD                                                                                                             |
| **Path**             | `.`                                                                                                              |
| **Cluster URL**      | [https://kubernetes.default.svc](https://kubernetes.default.svc)                                                 |
| **Namespace**        | default                                                                                                          |

<br>

Ficando assim o preenchimento dos campos no ArgoCD:

| <img width="1917" height="867" alt="image" src="https://github.com/user-attachments/assets/5e2aa07c-ed54-4449-a37b-b3f9d06329fe" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

| <img width="1918" height="868" alt="image" src="https://github.com/user-attachments/assets/06a9380b-1986-4a6a-a53e-848ae6964273" />|
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

| <img width="1915" height="862" alt="image" src="https://github.com/user-attachments/assets/5758b4e6-6634-46f3-a231-0c8bd4cf50ee" />|
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

- Clique em Create

- Depois, clique em SYNC → SYNCHRONIZE

- Verifique se os pods da aplicação estão rodando:

```
kubectl get pods
```

<br>

## 🖥️ Etapa 6 – Acessar a aplicação localmente

Verifique os pods:

```
kubectl get pods
```

Crie o port-forward:

```
kubectl port-forward svc/hello-app-service 8080:8080
```

Acesse:
🔗 http://localhost:8080

E você verá sia aplicação rodando:

| <img width="1914" height="974" alt="image" src="https://github.com/user-attachments/assets/3aa6f440-61cb-4b69-a8b5-871cf8150ef7" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Rodando* |


## 🧪 Etapa 7 – Testar atualização automática

Edite o arquivo `main.py` e altere o return:

```python
return {"message": "Hello Compass"}
```

Faça commit e push no repositório hello-app.

O GitHub Actions buildará uma nova imagem:

| <img width="1912" height="969" alt="image" src="https://github.com/user-attachments/assets/549d740f-4b3d-4587-a906-f9cae27262d4" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Build GitHub Actions* |

<br>

Publicará no Docker Hub:

| <img width="912" height="574" alt="image" src="https://github.com/user-attachments/assets/e175fe6b-d2f1-4e78-a289-3a37075d3fa0" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Imagem Docker Hub* |

<br>

E atualizará o repositório hello-manifests com a mesma tag que foi publicado no Docker Hub:

| <img width="1893" height="591" alt="image" src="https://github.com/user-attachments/assets/740f0c38-af86-4fa2-8905-f13636505c77" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Repositório Atualizado* |



O ArgoCD detectará a mudança e fará o deploy automaticamente.

Após a sincronização, atualize a página em http://localhost:8080 — a nova mensagem aparecerá!

| <img width="1912" height="967" alt="image" src="https://github.com/user-attachments/assets/a649ceaa-0393-48f9-9d0e-37ad09ba2462" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Atualizando* |

<br>

## 🧾 Conclusão

Este projeto demonstra, de forma prática, o funcionamento do ciclo completo de CI/CD e GitOps:
desde o desenvolvimento e build automatizado, até a entrega contínua via ArgoCD.

Com essa abordagem, toda a infraestrutura e o estado da aplicação ficam versionados no Git, garantindo rastreabilidade, segurança e velocidade nas entregas.

---
🧑‍💻 Desenvolvido por [Pedro Albertini Fernandes Pinto](https://github.com/pedro-albertini) 
Projeto prático do módulo **Automação CI/CD e GitOps com FastAPI e ArgoCD**


