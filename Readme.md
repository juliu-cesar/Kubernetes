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

## Criando o primeiro Pod

O processo de criar um Pod é bastante simples, bastando criar um arquivo declarativo `.yaml` e definir algumas especificações, vejamos o exemplo para o arquivo `kube-go/pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: "go-server"
  labels: 
    app: "go-server-label"
spec:
  containers:
    - name: go-server-container
      image: "juliucesar/kube-golang:latest"
```

- apiVersion: Define a versão da api do Kubernetes.
- kind: Define o tipo de objeto Kubernetes que sera criado.
- metadata: Contem os metadados que identificam o objeto.
  - name: Define o nome do objeto.
  - label: Cria uma etiqueta para o objeto, que ajuda na sua identificação e permite filtro-los quando for efetuado uma busca entre os objetos criados.
- spec: Define as especificações do objeto.
  - containers: Lista os containers que serão executados no Pod.

> Dentro da pasta `kube-go` criamos um programa muito simples em Golang para ser executado dentro dos containers.

O proximo passo é aplicar essas configurações de Pod no Cluster criado no passo anterior [Criando múltiplos Nodes](#criando-múltiplos-nodes). Para isso vamos utilizar o comando:

```bash
kubectl apply -f kube-go/pod.yaml
```

Apos esse processo temos o Pod criado, e para verificar isso basta executar o comando `kubectl get pods`, que deve retornar algumas informações dele.

## Utilizando o ReplicaSet

Um dos processos mais importantes do Kubernetes é seu gerenciamento de containers, e isso não ocorre quando criamos manualmente os Pods como foi feito acima. O processo que utilizaremos sera com o **ReplicaSet**, e que possui uma estrutura semelhante ao do Pod. Ainda dentro da pasta `kube-go` vamos adicionar o arquivo `replicaset.yaml`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: go-server-replica
  labels:
    app: go-server-replica-label
spec:
  selector:
    matchLabels:
      app: go-server-label
  replicas: 5
  template:
    metadata:
      labels:
        app: "go-server-label"
    spec:
      containers:
        - name: go-server-container
          image: "juliucesar/kube-golang:latest"
```

- apiVersion: Especifica a versão da API do Kubernetes usada para criar este objeto.
- kind: Define o tipo de objeto que está sendo criado, que no caso é um ReplicaSet.
- metadata: Contém metadados para identificar e descrever o objeto.
- spec: Define as especificações do ReplicaSet.
  - selector e matchLabels: Especifica a label do Pod que este ReplicaSet deve gerenciar, no caso todos que tenham a label `go-server-label`
  - replicas: Indica o número desejado de réplicas dos Pods gerenciados por este ReplicaSet.
  - template: Descreve o modelo para os Pods que serão criados e gerenciados.

Para criar o ReplicaSet utilizaremos o mesmo comando anterior `kubectl apply -f kube-go/replicaset.yaml`. Com isso temos os Pods criados e caso algum deles caia, o Kubernetes se encarrega de subir novos para manter a quantidade definida.

### O problema do ReplicaSet

Suponhamos que houve atualizações no programa feito em Golang, e uma nova imagem Docker foi criada, então logo atualizamos o campo `image` do ReplicaSet para a nova versão e efetuamos o apply do arquivo. Porem o que o Kubernetes faz é aplicar a atualização somente nos novos Pods criados, onde os que continuam rodando, mantém a versão antiga. Ou seja, seria necessário excluir todos os Pods manualmente para que os novos fossem criado com a atualização. Vejamos como resolver esse problema no tópico seguinte.

> Podemos obter as informações de um Pod com o comando `kubectl describe pod nome_do_pod`.

## Deployment

Bastante parecido com o ReplicaSet, o Deployment cria objetos de acordo com um estado que foi definido. Este objeto nada mais é que um ReplicaSet em si, ou seja, temos `deployment` que cria o `replicaset` que cria os `pods`. O conceito por traz do Deployment é bastante importante, mas sua implementação é bastante simples, vejamos como fica o arquivo `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
# Código omitido
```

Basta mudar o tipo de objeto criado e aplicar essas mudanças com o mesmo código visto anteriormente (*kubectl apply*) que teremos o Deployment criado. Caso seja feita alguma atualização ou algo que mude o estado dos Pods, o deployment se encarrega de **gradualmente** substituir os Pods antigos pelos novos. Apesar de simples, esta ferramente é muito util e sera utilizada diversas vezes. 

### Rollout e Revisões

Algo muito interessante com o Deployment, é que quando é feito a atualização/alteração no estado dos Pods, além dele criar um novo ReplicaSet ele também mantém o anterior. E graças a isso podemos efetuar um Rollout, que é o processo de retornar o estado para uma versão anterior. Suponhamos que houve algum erro com a nova versão dos Pods, podemos utilizar o seguinte comando:

```bash
kubectl rollout undo deployment nome_do_deployment

kubectl rollout undo deployment nome_do_deployment --to-revision=numero_da_revisao
```

O primeiro comando retorna para a versão anterior, e o segundo para qualquer versão escolhida. Podemos verificar no numero da revisão com o comando `kubectl rollout history deployment nome_do_deployment`.

## Services

O Service é uma abstração lógica para expor as aplicações implementadas dentro do cluster Kubernetes para outras aplicações dentro ou fora do cluster. Ele fornece uma maneira estável de acessar os Pods que estão executando sua aplicação, independentemente de onde esses Pods estejam localizados no cluster. Vejamos algumas de suas características:

- Balanceamento de Carga: Os Services ja possuem por padrão um balanceamento de carga, onde ao receber uma requisição, ele redirecionara para o Pod numero um, então para o numero dois, para o tres e assim sucessivamente.
- Tipos de Services: Existem diferentes tipos de Services no Kubernetes, incluindo:
  - ClusterIP: Expõe o Service somente dentro do cluster.
  - NodePort: Expõe o Service em uma determinada porta que pode ser acessada utilizando o IP de um node. Ou seja, independente de que Node for acessado, ao passar a porta definida, ira cair no Service.
  - LoadBalancer: Provisionar um balanceador de carga externo (no caso de um provedor de nuvem suportado) para expor o Service.
  - ExternalName: Redirecionar solicitações para um nome de serviço externo (por exemplo, um serviço fora do cluster).
- Seleção de Pods: Os Services usam etiquetas (labels) para selecionar os Pods aos quais devem direcionar o tráfego. Eles podem direcionar o tráfego para um conjunto específico de Pods com base nas tags associadas a esses Pods.

### Criando um Service

Ainda dentro da pasta `kube-go` vamos adicionar o arquivo `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-server-service
spec:
  selector:
    app: go-server-label
  type: ClusterIP
  ports:
  - name: go-server-service
    port: 80
    protocol: TCP
```

- spec: Define as especificações do Service.
  - selector: Define para quais Pods o Service deve direcionar o tráfego. Neste caso, os Pods com o rótulo `app: go-server-label` serão selecionados para receber o tráfego.
  - type: Define o tipo do Service.
- ports: Especifica as portas que o Service expõe.
  - name: Apesar de não ser necessário, é bastante recomendado colocar um nome para a porta do Service.
  - port: O número da porta que o Service escuta.
  - protocol: O protocolo de rede usado (geralmente TCP ou UDP).

Para criar o service, utilizamos o mesmo comando `kubectl apply`. Porem mesmo assim não é possível acessar o serviço, e isso ocorre por que o Kubernetes disponibiliza a porta apenas para a rede interna. Podemos utilizar o seguinte comando para fazer um redirecionamento:

```bash
kubectl port-forward svc/nome_do_service 8000:80
```

> O `svc` ou `service` indica que esse processo sera aplicado em um Service.

#### TargetPort

Suponhamos que a porta para acessar o service e o container não fossem as mesmas, poderíamos então utilizar o campo `targetPort` para indicar qual seria a porta do container, por exemplo:

```yaml
- name: go-server-service
  port: 80
  targetPort: 9000
```

Com isso podemos acessar o serviço pela porta 80, enquanto ele ira redirecionar para o container na porta 9000. No arquivo do tópico anterior não precisamos fazer essa passo pois ambas as portas eram iguais, tanto a do container como o do service eram 80.

### NodePort

O NodePort é um tipo de Service que permite acessar o serviço externamente. Para este objeto utilizamos portas altas, ou seja entre `30000` e `32767`. Vejamos um exemplo abaixo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-server-service
spec:
  selector:
    app: go-server-label
  type: NodePort
  ports:
  - name: go-server-service
    port: 80
    protocol: TCP
    nodePort: 30001
```

Dessa forma, ao acessar qualquer um dos Nodes passando a porta `300001`, sera redirecionado para a porta 80 do service, que por sua vez vai redirecionar para o container.

Apesar deste tipo poder ser utilizado, **não é muito recomendado**, principalmente por questões de segurança, e também temos o proximo tipo que acaba sendo mais util.

### LoadBalancer

O LoadBalancer é um dos tipos de Service que oferece um recurso nativo para disponibilizar um balanceador de carga externo para expor um aplicativo para fora do cluster. Geralmente utilizado com algum provedor em nuvem, dessa forma ao criar esse objeto, sera atribuído um ip externo, que nos permite acessar o Service de fora do cluster. Para criar ele basta mudar o campo do tipo:

```yaml
type: LoadBalancer
```

## Acessando a API do Kubernetes através de um proxy

O Kubernetes funciona disponibilizando uma api, e podemos acessa-la diretamente utilizando um proxy, vejamos o comando para isso:

```bash
kubectl proxy --port=8080   
```

Com isso podemos acessar a api utilizando o endereço `http://localhost:8080`. Nele temos todas as rotas disponíveis e diversas informações. Por exemplo, podemos acessar as informações de um service com o endereço `http://localhost:8080/api/v1/namespaces/default/services/nome_do_service`.

## Configurando variáveis de ambiente

Temos algumas formas para configurar variaveis de ambiente no container. Vejamos algumas

- Adicionando diretamente ao arquivo `Deployment`, no campo `env` passamos o nome e o valor.:

```yaml
# código omitido
containers:
  - name: go-server-container
    image: "juliucesar/kube-golang:v3"
    env:
      - name: NAME
        value: "Juliu Cesar"
      - name: AGE
        value: "255"
```
- Criando um ConfigMap para centralizar as variáveis de ambiente. Esta opção é mais recomendada que a anterior, uma vez que configuramos um arquivo separado apenas para estas variáveis. Vamos criar o arquivo `config-env.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: go-server-env
data:
  NAME: "Juliu Cesar"
  AGE: "255"
```

Apos isso é preciso fazer uma referencia à esses campos no arquivo `deployment`:

```yaml
containers:
  - name: go-server-container
    image: "juliucesar/kube-golang:v3"
    env:
      - name: NAME
        valueFrom:
          configMapKeyRef:
            name: go-server-env
            key: NAME
      - name: AGE
        valueFrom:
          configMapKeyRef:
            name: go-server-env
            key: AGE
```

Utilizamos o `valueFrom` para indicar que o valor vira de outro arquivo, e `configMapKeyRef` para informar qual o nome do Map que estamos referenciando e o campo com o valor definido. Em seguida precisamos aplicar o Map e atualizar o Deployment.

> Caso seja atualizado o Map, essas mudanças não serão refletidas automaticamente no Deployment, é necessário atualiza-lo manualmente.

```bash
kubectl apply -f kube-go/config-env.yaml

kubectl apply -f kube-go/deployment.yaml
```
