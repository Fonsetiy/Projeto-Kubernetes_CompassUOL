# Projeto - GitOps na prática com Kubernetes e ArgoCD
O desenvolvimento moderno exige aplicações ágeis, seguras e fáceis de escalar. O Kubernetes permite orquestrar contêineres de forma automatizada e resiliente, enquanto o GitOps traz controle e rastreabilidade ao usar o Git como base da infraestrutura.
Neste projeto, será criada uma implantação automatizada de um aplicativo usando GitHub, Rancher Desktop e ArgoCD, demonstrando na prática como as equipes operam em ambientes cloud-native.

# Objetivo
Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

# 🛠️ Tecnologias Utilizadas

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/docs/)
[![Argo CD](https://img.shields.io/badge/ArgoCD-FB6E00?style=for-the-badge&logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/en/stable/)
[![Rancher Desktop](https://img.shields.io/badge/Rancher%20Desktop-0075A8?style=for-the-badge&logo=rancher&logoColor=white)](https://docs.rancherdesktop.io/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://docs.github.com/)
[![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)](https://git-scm.com/doc)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/)
[![YAML](https://img.shields.io/badge/YAML-000000?style=for-the-badge&logo=yaml&logoColor=white)](https://yaml.org/spec/)
[![Online Boutique](https://img.shields.io/badge/Online%20Boutique-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://github.com/GoogleCloudPlatform/microservices-demo)

# ⚙️ Pré-requisitos
- Rancher Desktop instalado (com Kubernetes habilitado)
- Kubectl configurado (kubectl get nodes funcionando)
- ArgoCD instalado no cluster
- Conta no GitHub com repositório público
- Git instalado
- Docker funcionando localmente

# Passo a Passo
### 🔷 Etapa 1  - Preparar repositório no GitHub.
Preparar o repositório que será a fonte para o GitOps, contendo apenas o manifesto necessário.


🔹 1.1 Fork do repositório oficial: vá até ```https://github.com/GoogleCloudPlatform/microservices-demo``` e faça um fork desse repositório.


🔹 1.2 Crie um novo repositório público na sua conta do GitHub.


🔹 1.3 Dentro do repostório que você acabou de criar, clique em "Add file" -> "Create new file". No campo do nome do arquivo, coloque "k8s/online-boutique.yaml" (O GitHub cria automaticamente a pasta k8s/ quando você escreve dessa forma). Agora vá até o seu fork do microservices-demo, abra o caminho ```release/kubernetes-manifests.yaml```, clique em Raw -> selecione tudo -> copie e cole no seu arquivo yaml e salve as alterações. Seu repositório ficará assim:

    
    gitops-microservices/
    └── k8s/
        └── online-boutique.yaml
    

### 🔷 Etapa 2 - Instalar o ArgoCD no cluster local.

1. Crie o Namespace:
``` powershell
kubectl create namespace argocd
```

2. Instalar o ArgoCD:
``` powershell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 🔷 Etapa 3 - Acessar a Interface do ArgoCD
1. O usuário por padrão é "admin". Para obter a senha incial, que é gerada automaticamente, use o comando (para PowerShell):
``` powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

2. Faça port-forward para acessar a UI localmente com o comando (necessário manter rodando em um terminal):
``` powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3. Abra seu navegador e acesse ```https://localhost```. Utilize o usuário a senha obtidos no passo anterior.
Você verá uma tela parecida com essa:

![Captura de tela 2025-10-26 121649](https://github.com/user-attachments/assets/8468dec1-e0b9-4f4d-a0eb-443a6d79dd1c)

![argo application1](https://github.com/user-attachments/assets/13f23ca4-5b49-4d73-a2a2-f7d6267d6d52)

### Etapa 4 - Criar e Sincronizar a aplicação no ArgoCD
Vincular o repositório Git ao ArgoCD e sincronizar a aplicação, aplicando os manifests no cluster Kubernetes.

🔹 4.1 - Agora você deve vincular o ArgoCD ao seu repositório Git e criar a aplicação que será implantada automaticamente no Kubernetes.
Com o comando abaixo, você informa ao ArgoCD onde estão os manifests YAML e como ele deve aplicá-los no cluster.

❗ Lembre-se de substituir o campo <seu_usuario> pelo endereço do repositório que você configurou na primeira etapa.

``` powershell
argocd app create online-boutique `
  --repo https://github.com/<seu-usuario>/gitops-microservices.git `
  --path k8s `
  --dest-server https://kubernetes.default.svc `
  --dest-namespace default `
  --directory-recurse
```

![ONLINE-BOUTIWUE CRRATED](https://github.com/user-attachments/assets/9de628ba-e130-4a13-8d96-753955f7a82d)

🔹 4.2 - Sincronize a sua aplicação utilizando o comando:
``` powershell
argocd app sync online-boutique
```

<img width="983" height="654" alt="image" src="https://github.com/user-attachments/assets/c741e520-423a-4b7a-90be-37af840453e6" />


### 🔷 Etapa 5 - Acessar o Front-End

🔹 5.1 - Verifique os Pods e serviços:
``` powershell
kubectl get pods -n default
kubectl get svc -n default
```
<img width="629" height="214" alt="image" src="https://github.com/user-attachments/assets/1eefcd2b-0498-400d-92ed-b4d4761b32ee" />

<img width="558" height="203" alt="image" src="https://github.com/user-attachments/assets/43fa9c63-e948-44e0-a158-a4aad72f9fd5" />

🔹 5.2 - O frontend no manifest é um ClusterIP (frontend) com port 80 → targetPort: 80. Para acessar localmente faça port-forward:
``` powershell
kubectl port-forward svc/frontend -n default 80:80
```
<img width="592" height="89" alt="image" src="https://github.com/user-attachments/assets/5527a2d1-9e84-46c5-8ffa-982d41e73fec" />


🔹 5.3 - Acesse a loja abrindo seu navegador e acessando ```http://localhost```. Você deverá ver a página da "Online Boutique".

![Captura de tela 2025-10-26 125954](https://github.com/user-attachments/assets/a172f596-0201-44e8-abc9-e365acd5375d)

## Entregas esperadas:

- Configurar um repositório Git público contendo os manifests YAML do projeto.

- Implantar o ArgoCD corretamente no cluster Kubernetes.

- Criar a aplicação no ArgoCD vinculando ao repositório Git configurado.

- Sincronizar o aplicativo e confirmar que todos os pods estão em execução.

- Acessar o frontend da aplicação utilizando o comando kubectl port-forward.

- (Opcional) Personalizar os manifests, como ajustar o número de réplicas de um microserviço.
