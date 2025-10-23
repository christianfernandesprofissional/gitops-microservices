# Deploy Contínuo com Kubernetes e ArgoCD

Este projeto demonstra como automatizar o deploy de uma aplicação hospedada no GitHub usando Kubernetes e ArgoCD em um ambiente Linux. Para esta demonstração foi utilizado **Ubuntu 24.04.3 LTS**.

A aplicação é gerenciada dentro de um cluster Kubernetes, garantindo escalabilidade e alta disponibilidade. O ArgoCD sincroniza automaticamente o estado do cluster com o repositório Git, aplicando mudanças sempre que houver atualizações no código ou nos manifests.

Funcionalidades:

- Deploy contínuo via GitOps.

- Monitoramento do estado da aplicação pelo ArgoCD.

- Rollback fácil para versões anteriores.

- Suporte a múltiplos ambientes (dev, teste, produção).

Este projeto é ideal para quem deseja aprender práticas modernas de DevOps, GitOps e Kubernetes.


## Requisitos

Para este projeto é necessário que você tenha instalado na sua máquina algumas ferramentas, para isso basta seguir os tutoriais descritos nos links abaixo:

- **kubectl:** <a>https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/</a>
- **rancher-desktop:** <a>https://docs.rancherdesktop.io/getting-started/installation/</a>
- **ArgoCD:** <a>https://argo-cd.readthedocs.io/en/stable/getting_started/</a>
- **ArgoCD CLI:** <a>https://argo-cd.readthedocs.io/en/stable/cli_installation/</a>


***

Para dar início, após a instalação das ferramentas devemos testar se tudo está funcionando. Com o Rancher-Desktop aberto, abra o terminal e digite o seguinte comando:

    kubectl get pods -n argocd

E você deve esperar esta resposta:

IMAGEM 1

Com o argocd funcionando você deve criar um repositório no Github contendo o arquivo Yaml da sua aplicação. Neste projeto o nosso Yaml está localizado dentro da pasta k8s/online-boutique.yaml.
