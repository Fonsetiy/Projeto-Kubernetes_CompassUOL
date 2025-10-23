# Projeto-Kubernetes_CompassUOL
O desenvolvimento moderno exige aplicações ágeis, seguras e fáceis de escalar. O Kubernetes permite orquestrar contêineres de forma automatizada e resiliente, enquanto o GitOps traz controle e rastreabilidade ao usar o Git como base da infraestrutura.
Neste projeto, será criada uma implantação automatizada de um aplicativo usando GitHub, Rancher Desktop e ArgoCD, demonstrando na prática como as equipes operam em ambientes cloud-native.

# Objetivo
Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.

# Tecnologias utilizadas
ArgoCD, Kubernetes, Git, PowerShell, Rancher Desktop, 

# Pré-requisitos
- Rancher Desktop instalado (com Kubernetes habilitado)
- Kubectl configurado (kubectl get nodes funcionando)
- ArgoCD instalado no cluster
- Conta no GitHub com repositório público
- Git instalado
- Docker funcionando localmente

# Passo a Passo
### Etapa 1  - Preparar repositório no GitHub.
Preparar o repositório que será a fonte para o GitOps, contendo apenas o manifesto necessário.

1.1 Fork do repositório oficial: vá até ```https://github.com/GoogleCloudPlatform/microservices-demo``` e faça um fork desse repositório.
1.2 Crie um novo repositório público na sua conta do GitHub.
1.3 Dentro do repostório que você acabou de criar, clique em "Add file" -> "Create new file". No campo do nome do arquivo, coloque "k8s/online-boutique.yaml" (O GitHub cria automaticamente a pasta k8s/ quando você escreve dessa forma). Agora vá até o seu fork do microservices-demo, abra o caminho ```release/kubernetes-manifests.yaml```, clique em Raw -> selecione tudo -> copie e cole no seu arquivo yaml e salve as alterações. 

### Etapa 2 - Instalar o ArgoCD no cluster local.

1. Crie o Namespace:
```kubectl create namespace argocd```

2. Instalar o ArgoCD:
```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```

### Etapa 3 - Acessar a Interface do ArgoCD
1. O usuário por padrão é "admin". Para obter a senha incial, que é gerada automaticamente, use o comando (para PowerShell):
```kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }```.

2. Faça port-forward para acessar a UI localmente com o comando (necessário manter rodando em um terminal):
```kubectl port-forward svc/argocd-server -n argocd 8080:443```.

3. Abra seu navegador e acesse ```https://localhost:8080```. Utilize o usuário a senha obtidos no passo anterior.
Você verá uma tela parecida com essa:
![argo application1](https://github.com/user-attachments/assets/13f23ca4-5b49-4d73-a2a2-f7d6267d6d52)

### Etapa 4 - Criar e Sincronizar a aplicação no ArgoCD
