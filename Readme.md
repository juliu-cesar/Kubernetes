# Kubernetes

Desenvolvido pela Google, o Kubernetes é uma plataforma de orquestração de contêineres de código aberto que visa simplificar a gestão e operação de aplicativos em contêineres. Ela automatiza a implantação, o dimensionamento e a gestão de aplicativos em uma plataforma poderosa para desenvolvedores e administradores de sistemas. Temos alguns pontos a destacar como a **arquitetura baseada em cluster**, a **escalabilidade automação**, a **compatibilidade com Multi-Cloud**.

## Funcionamento

O Kubernetes opera em um ambiente de **cluster**, que é composto pela conexão de um conjunto de máquinas físicas ou virtuais que são chamados de **Nodes** (nós). Cada nó em um cluster Kubernetes desempenha um papel específico na operação do sistema e possui uma quantidade de vCPU e memoria.

### KubeCtl

O kubectl é a ferramenta de linha de comando principal para interagir com clusters Kubernetes. Ele atua como um cliente para a API do Kubernetes, permitindo que os usuários gerenciem recursos e executem operações no cluster.

### Cluster

Um Cluster Kubernetes é formado por pelo menos um nó mestre, chamado de **Master Node** e diversos nós de trabalho (Worker Nodes). Enquanto o nó principal é responsável pelo gerenciamento do cluster, os demais são encarregados de executar as cargas de trabalho.

<img src="https://github.com/juliu-cesar/GPU-Store/assets/121033909/6e0ab445-f33a-4a91-a61d-b6a0b84e9fb6" height="350"/>

### Master Node

O nó mestre é composto por vários componentes:

- **API Server**: É o ponto de entrada para o cluster. Ele expõe a API do Kubernetes, permitindo que os usuários e os componentes do Kubernetes interajam com o cluster.
- **Control Plane**: Inclui o *Controller Manager*, *Scheduler* e *etcd*. O Controller Manager gerencia ciclos de vida de objetos do Kubernetes, o Scheduler distribui cargas de trabalho nos nós e o etcd é um armazenamento de chave-valor usado para armazenar o estado do cluster.
- **kubelet**: É um agente que roda em cada nó de trabalho e mantém a comunicação com o nó mestre. Ele gerencia os contêineres em um nó, garantindo que estejam em execução e saudáveis.

### Worker Nodes

Cada nó de trabalho executa os contêineres reais e executa os seguintes componentes:

- **kubelet**: Como visto anteriormente, é o agente que gerencia os contêineres em um nó de trabalho.
- **kube-proxy**: Mantém as regras de rede do Kubernetes, gerenciando o tráfego de rede para os pods.

### Pods

É a unidade básica de implantação que contem os containers provisionados. Um Cluster pode conter um ou mais Pods e cada Pod pode conter um ou mais containers. Dentro do Pod temos um ambiente onde os containers compartilham recursos, como endereço ip, espaço em disco e namespace.

<img src="https://github.com/juliu-cesar/GPU-Store/assets/121033909/a9ee1579-733b-4e05-b7ed-3f82e046d2f0" height="350"/>

### Deployment

O Deployment é uma abstração que gerencia a implantação e a atualização dos Pods. Definimos para ele o estado do aplicativo e ele garante que este estado seja mantido se utilizando dos seguintes pontos:

- **ReplicaSets**: Define a quantidade de cópias de um Pod que devem ser mantidas em execução.
- **Resiliência e Auto-recuperação**: Quando uma réplica de Pod ou Nó cai, o Deployment identifica a interrupção e inicia a criação de uma nova réplica para manter o número de ReplicaSets em execução.
- **Gerenciamento de Nodes**: O Kubernetes monitora constantemente o estado dos nodes no cluster. Se um nó ficar inacessível ou falhar, o orquestrador (como o kube-controller-manager) identifica essa situação e toma medidas para mitigar o impacto. Se uma réplica de Pod estiver em um nó que se tornou indisponível, o Kubernetes tentará criar uma nova réplica em outro nó saudável (**caso ele possua recursos disponíveis**).
- **Atualizações e Rollbacks**: O Deployment permite atualizar a versão de um aplicativo de forma controlada. Ao atualizar a definição do Deployment com uma nova versão do contêiner, ele realiza uma atualização as poucos, substituindo gradualmente as instâncias dos Pods pela nova versão. Além disso, ele oferece a possibilidade de realizar rollbacks para versões anteriores caso algo dê errado durante a atualização.

## Kind

Para este projeto vamos executar o Kubernetes localmente na maquina, evitando assim qualquer custo com algum Cloud Provider. A solução escolhida sera a [Kind](https://kind.sigs.k8s.io/docs/user/quick-start), que se utiliza de containers Docker para rodar, dessa forma a cada novo container temos um novo Node.

> Apesar do Kind não precisar do [KubeCtl](https://kubernetes.io/docs/tasks/tools/) para rodar, nos precisamos dessa ferramenta para poder se comunicar com o Kubernetes.

### Principais comandos

O Kind possui tres principais comandos, e são eles:

- **Create**: Cria um novo cluster Kubernetes local.

```bash
kind create cluster --name meu-cluster
```

`--name` é usado para dar um nome ao cluster (opcional).

- **Delete**: Exclui um cluster Kubernetes existente.

```bash
kind delete cluster --name meu-cluster
```

`--name` é usado para especificar o nome do cluster a ser excluído.

- **Get**: Lista todos os clusters Kind existentes no sistema.

```bash
kind get clusters
```

### Criando um novo cluster

Para criar um novo cluster basta executar o comando `kind create cluster`, que ao final do processo teremos um container Docker com o Kubernetes rodando e o seguinte comando sera exibido: 

```bash
kubectl cluster-info --context kind-kind
```

Este comando é utilizado para definir o contexto do KubeCtl, e o `kind-kind` é o nome do cluster. Podemos verificas os nodes em execução com o comando `kubectl get nodes`.

### Criando múltiplos Nodes

No comando para criar um cluster do Kind podemos informar um arquivo `.yaml` com as configurações desejadas. Dentro da pasta `multiNode` vamos ter o arquivo de configuração:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

Primeiramente escolhemos a versão da Api, e em seguida selecionamos a quantidade de nodes desejada. O proximo passo é criar o cluster com estas configurações e alterar o contexto do kubectl:

```bash
kind create cluster --config=multiNodes/kind.yaml --name=kube-multi-nodes

kubectl cluster-info --context kind-kube-multi-nodes
```

Para verificar a quantidade de nodes de em pé podemos executar o mesmo comando visto anteriormente `kubectl get nodes`.

#### Trocando de contexto

Temos uma extensão para o VsCode que facilita o processo de visualização e mudança de contexto, que se chama [Kubernetes](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools). Ela cria uma nova aba na Barra de atividades onde temos diversas informações sobre os contextos cadastrados na maquina, e o cluster selecionado.

> Também é possível verificar os contextos cadastrados com o comando `kubectl config get-clusters`.
