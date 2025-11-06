# Repositório de Manifests – encurtador_url

Este repositório contém os **manifests do Kubernetes** usados para realizar o **deploy da aplicação Encurtador de URL** no cluster.
Ele é responsável por descrever **como o Kubernetes deve executar e expor a aplicação**.

---

## O que são Manifests?

Os manifests são arquivos YAML que definem recursos no Kubernetes, como:

- **Deployment** → Garante que a aplicação esteja sempre rodando.
- **Service** → Expõe a aplicação dentro (ou fora) do cluster.
- **ConfigMaps, Secrets, Ingress, etc.** → (opcionais) configuram e estendem o comportamento da aplicação.

 Cada mudança feita nesses arquivos pode ser aplicada diretamente no cluster via `kubectl apply -f`.

---

##  Estrutura do Projeto

```bash
manifests/
├── deployment.yaml
└── service.yaml
```

---

### `service.yaml`

Define o serviço interno no cluster:

```bash
apiVersion: v1
kind: Service
metadata:
  name: encurtador-service
spec:
  selector:
    app: encurtador-url
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8000
  type: ClusterIP
```

---

### `deployment.yaml`

Define o **Deployment** da aplicação — ou seja, como o Kubernetes vai executar e manter os containers em funcionamento.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: encurtador-url
spec:
  replicas: 1
  selector:
    matchLabels:
      app: encurtador-url
  template:
    metadata:
      labels:
        app: encurtador-url
    spec:
      containers:
        - name: encurtador-url
          image: icrowleyshr/encurtador-url:1762393531
          ports:
            - containerPort: 8000
```

---

## Pré-requisitos e Instalações

### 1. Instalar Rancher Desktop
**O que é:** Rancher Desktop cria e gerencia seu cluster Kubernetes local.

**Passo a passo:**
1. Acesse [https://rancherdesktop.io/](https://rancherdesktop.io/)
2. Clique em **"Download"**
3. Escolha sua versão (Windows, Mac ou Linux)
4. Execute o instalador e siga o assistente
5. Após abrir o Rancher Desktop:
   - Clique em **Settings** (⚙️ canto superior direito)
   - Vá em **Kubernetes**
   - Ative **"Enable Kubernetes"**
   - E na primeira inicialização selecione como Container Engine: Dockerd
   - Aguarde aparecer **"Kubernetes running"** com ✔️ verde

**Verificação:**

```bash
kubectl get nodes
```

**Resultado esperado:** Lista seu nó com STATUS "Ready"

### 2. Instalar kubectl
**O que é:** Ferramenta de linha de comando para controlar o Kubernetes.

**Passo a passo:**
1. Acesse [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)
2. Baixe a versão para seu sistema operacional
3. Siga as instruções de instalação
4. Abra o Prompt de Comando/Terminal

**Teste a instalação:**

```bash
kubectl version --client
```

**Resultado esperado:** Mostra a versão instalada

### 3. Instalar Docker
**O que é:** Plataforma para criar e executar containers.

**Passo a passo:**
1. Acesse [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Baixe o Docker Desktop
3. Instale e execute
4. Aguarde o ícone da baleia aparecer na barra de tarefas

**Teste a instalação:**

```bash
docker ps
```

 **Resultado esperado:** Lista containers (pode estar vazia)
 
 *Importante, feche para não dar conflito com o Rancher Desktop depois do teste, precisamos só das dependências dele*

---

## 🚀 Deploy Contínuo com ArgoCD

Este repositório de manifests foi projetado para funcionar com o **ArgoCD**, permitindo **deploys automáticos no Kubernetes** sempre que houver uma atualização (por exemplo, uma nova tag de imagem).

---

### 🧩 O que o ArgoCD faz

O **ArgoCD** é uma ferramenta de **Continuous Deployment (CD)** para Kubernetes baseada em GitOps.

Isso significa que:
- O ArgoCD **monitora este repositório**;
- Quando detecta uma **mudança no YAML** (ex: atualização da imagem no `deployment.yaml`);
- Ele **sincroniza automaticamente** o cluster Kubernetes com a versão mais recente deste repositório.

> 💡 Em resumo: o Git é a "fonte da verdade" — o ArgoCD aplica automaticamente o que está aqui no cluster.

---

## ⚙️ Configurando o ArgoCD

### 1. Instale o ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verifique os pods:

```bash
kubectl get pods -n argocd
```

---

### 2. Acesse o painel do ArgoCD

Crie o port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse:
```bash
https://localhost:8080
```

Usuário padrão: `admin`  
Senha inicial:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

### 3. Crie a aplicação no ArgoCD

| Campo | Valor |
|--------|--------|
| **Application Name** | `encurtador-url` |
| **Project** | `default` |
| **Sync Policy** | `Automatic` |
| **Repository URL** | `https://github.com/icrowleyshr/manifests-encurtador-url.git` |
| **Revision** | `main` |
| **Path** | `.` |
| **Cluster** | `https://kubernetes.default.svc` |
| **Namespace** | `default` |

---

### 4. Ative Auto-Sync

No painel do ArgoCD, habilite:

- `SYNC POLICY → Automatic`
- `PRUNE → Enabled`
- `SELF HEAL → Enabled`

Assim, sempre que o pipeline atualizar o `deployment.yaml` com uma nova tag de imagem, o ArgoCD fará o deploy automaticamente.

---
