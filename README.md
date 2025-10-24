# Deploy Contínuo com Kubernetes e ArgoCD

<img align="center" alt="Linux" width="80px" height="80px" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/kubernetes/kubernetes-original.svg"/> <img align="center" alt="Linux" width="80px" height="80px" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/argocd/argocd-original-wordmark.svg"/> <img align="center" alt="Linux" width="80px" height="80px" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/linux/linux-original.svg"/>


Este projeto demonstra como automatizar o deploy de uma aplicação hospedada no GitHub usando Kubernetes e ArgoCD em um ambiente Linux. Para esta demonstração foi utilizado **Ubuntu 24.04.3 LTS**.

A aplicação é gerenciada dentro de um cluster Kubernetes, garantindo escalabilidade e alta disponibilidade. O ArgoCD sincroniza automaticamente o estado do cluster com o repositório Git, aplicando mudanças sempre que houver atualizações no código ou nos manifests.

Funcionalidades:

- Deploy contínuo via GitOps.

- Monitoramento do estado da aplicação pelo ArgoCD.

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

![argocd-pods](/images/argocd-pods.png "Pods ArgoCD")

Com o argocd funcionando você deve criar um repositório no Github contendo o arquivo Yaml da sua aplicação. Neste projeto o nosso Yaml está localizado dentro da pasta k8s/online-boutique.yaml.
Com o repositório criado, copie a url para o seu repositório:


![github-path](/images/github-path.png "Github path")


Agora no seu terminal, devemos expor a porta do ArgoCD para que possamos fazer login, para isso digite o comando:

        kubectl port-forward svc/argocd-server -n argocd 8080:443

E terá como resposta: 

![argocd-forward](/images/argocd-forward.png "ArgoCD port forward")


Com a porta exposta, abra mais um terminal sem fechar o outro. Iremos fazer login no ArgoCD, para isso primeiro precisamos decodificar a senha do ArgoCD usando o comando:

    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Este comando irá encontrar a senha, decodificar em base64 e exibir no seu terminal. Guarde esta senha para poder fazer login no ArgoCD usando o seguinte comando:

    argocd login localhost:8080 --username admin --password sua-senha-aqui

Após o comando aperte 'y' e login será feito:


![argocd-login](/images/argocd-login.png "ArgoCD Login")


Ou se preferir, pode acessar no seu navegador *http://localhost:8080* para acessar a interface gráfica.


![argocd-interface](/images/argocd-interface.png "ArgoCD Interface")


Nesta demonstração permaneceremos usando o terminal, porém é possível fazer os mesmo processos via interface gráfica.
No terminal em que você fez login vamos adicionar o nosso repositório a nossa lista de repositórios do ArgoCD usando o comando:

    argocd repo add endereco-do-seu-repositorio-aqui

Se estiver tudo OK com seu yaml podemos iniciar nossa aplicação. Primeiro vamos criar um namespace separado para a aplicação, o namespace utilizado foi 'boutique':

    kubectl create namespace boutique

Iremos utilizar este namespace para rodar nossa aplicação. Para isso iremos usar o comando abaixo:

    argocd app create nome-da-sua-aplicação --repo caminho-do-seu-repositório  --path caminho-da-pasta-do-seu-yaml-no-github  --dest-server https://kubernetes.default.svc  --dest-namespace boutique  --sync-policy automated  --auto-prune  --self-heal
        
 - **--dest-server https://kubernetes.default.svc** -> Aponta para qual cluster a aplicação será iniciada 

 - **--sync-policy automated** -> Deixa como automática a sincronização com seu repositório, ou seja, qualquer alteração será aplicada automaticamente

 - **--auto-prune** -> Permite que o ArgoCD remova recursos antigos que foram deletados do repositório.

 - **--self-heal** -> Permite que o ArgoCD corrija manualmente mudanças fora do Git.

Após a criação de todos os recursos podemos conferir se os recursos foram criados com os comandos:

    kubectl get pods -n boutique
        

![pods-boutique](/images/pods-boutique.png "Pods Boutique")


    kubectl get services -n boutique 


![services-boutique](/images/services-boutique.png "Services Boutique") 


        
Repare que sendo um Load Balancer o status fica como 'pending' porque como estamos criando a aplicação localmente, não existe de fato um LoadBalancer, por isso ele não consegue um IP para usar. Mas é possível através dele acessar nossa aplicação expondo uma porta da mesma maneira que fizemos com o ArgoCD, para isso usaremos o seguinte comando:

        kubectl port-forward svc/frontend-external 8081:80 -n boutique


Depois podemos acessar nossa aplicação em *http://localhost:8081*

![front-boutique](/images/front-boutique.png "Front Boutique")

Como não estamos usando um load-balancer vamos alterar as configurações do nosso yaml, mudando as configurações para usar o NodePort no lugar de Load Balancer nas configurações do Service frontend-external e acessar via http://localhost:30080


    apiVersion: v1
    kind: Service
    metadata:
      name: frontend-external
      labels:
        app: frontend
    spec:
      type: NodePort
      selector:
        app: frontend
      ports:
      - name: http
        port: 80
        targetPort: 8080
        nodePort: 30080

Após o commit das atualizações o ArgoCD irá sincronizar a aplicação automáticamente, mas caso não tenha usado a configuração *----sync-policy automated*, use o comando para se sincronizar com o repositório:

    argocd app sync online-boutique

Se você acessar o seu ArgoCD pelo navegador este será o resultado:


![resultado-final](/images/resultado-final.png "Resultado final")


## Conclusão

Este projeto demonstra como automatizar o deploy de uma aplicação usando Kubernetes e ArgoCD, garantindo que o estado do cluster esteja sempre sincronizado com o repositório Git.

O ArgoCD monitora continuamente e gerencia múltiplos ambientes, facilitando a aplicação de práticas modernas de DevOps e GitOps.

Seguindo este guia, é possível ter uma ideia de como funciona uma implantação automatizada em cloud-native, usando GitHub, Rancher Desktop com Kubernetes e ArgoCD.
