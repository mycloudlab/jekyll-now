---
published: false
layout: post
title: Networking no openshift
---
Em um cluster kubernetes, um dos pontos que é necessário atenção é a rede. Uma boa prática é fazer uma correta segregação da rede, pois isso otimiza o tráfego, isola serviços etc...

O Openshift possui 3 plugins para fazer o tratamento de rede, todos baseados no Open vSwitch (OVS).

* **ovs-subnet** - este plugin provê uma rede sobreposta simples, que permite a comunicação entre pods e services.

* **ovs-multitenant** - Este plugin provê uma uma forma de isolar pods e services cada projeto recebe um Virtual Network Id (VNID) para identificar de que projeto a comunicação de rede está ocorrendo. Um pod ou service de um projeto não pode se comunicar com o pod ou service de outro projeto.

Entretanto projetos que recebem o VNID 0 tem maiores privilégios. Eles podem se comunicar com odos os outros pods e os outros podem se comunicar com eles. Em um cluster openshift o projeto default possui o vnid 0, isso é usado por exemplo para criar serviços de infraestrutura como balanceadores de carga, docker registry, monitoria, etc...

* **ovs-networkpolicy** - Este plugin permite aos administradores do cluster fazer o isolamento de rede via Network policy.

\[TODO] verificar a configuração de rede dos nodes.

## DNS

Um ponto fundamental para facilitar o acesso ao cluster é entender a forma de resolução de nomes no cluster. Dentro de um cluster kubernetes, é executado um serviço DNS interno, usado para registrar os services do cluster, este dns é apenas para uso interno do cluster.

O dns é feito da seguinte forma:

```text
<service>.<namespace>.svc.cluster.local
```

Digamos que tenhamos um projeto chamado web e dentro do projeto web temos um service chamado frontend, portanto o dns qualificado dele seria: 
```text
frontend.web.svc.cluster.local
```

## Configurando o tipo de rede do cluster (ansible)

A configuração do tipo de rede do cluster pode ser feito via variável **os_sdn_network_plugin_name**.

Os aceitos os valores:

* redhat/openshift-ovs-subnet
* redhat/openshift-ovs-multitenant
* redhat/openshift-ovs-networkpolicy



O valor default desta variável é 'redhat/openshift-ovs-subnet'.

## Rede do tipo ovs-multitenant

Quando seu cluster está configurado para usar o plug-in SDN **redhat/openshift-ovs-multitenant**, você pode gerenciar a rede para um conjunto de projetos usando o utilitário de linha de comando **oc**.

### Juntando a rede entre projetos

Para permitir a comunicação de rede entre 2 projetos usamos o seguinte comando:

```bash
oc adm pod-network join-projects --to=<projeto1> <projeto2> 
```

Neste exemplo todos os pods e serviços de **<projeto1>** podem acessar os pods e serviços do **<projeto2>** e vice versa. Lembrando que para acessar deve usar o fully-qualified name ou nome completo (**<service>.<namespace>.svc.cluster.local**)

### Isolando a rede

Para isolar a rede use o comando:

```bash
oc adm pod-network isolate-projects <projeto1> <projeto2>
```

No exemplo acima os serviços e pods do <projeto1> <projeto2> não podem acessar outros projetos, ou seja eles estão isolados, apenas serviços globais estão acessíveis.

### Marcando um projeto como global

Um projeto global tem acesso a todos os pods e serviços do cluster e vice versa. Isso é útil para construir serviços globais como balanceadores de carga, registros docker, Builders para ambientes de CI/CD, etc...

Para marcar um projeto como global usamos o seguinte comando:

```bash
oc adm pod-network make-projects-global <project>
```
No exemplo acima os pods do **<project>** acessam todos os serviços e pods de todo o cluster e todos os projetos do cluster acessam os serviços e pods do **<project>**.
  
  
  \[TODO] - falar sobre como fazer o isolamento usando este tipo de rede e executar regras de iptables... Egress

## Rede do tipo ovs-networkpolicy

Kubernetes padronizou as politicas de rede usando um descritor chamado [NetworkPolicy](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/network/network-policy.md).

Openshift da suporte a esta forma de definição de políticas de rede por meio do plugin SDN **redhat/openshift-ovs-networkpolicy**

As políticas de rede deste plugin são criadas pelo administrador do projeto, criando ou removendo objetos do tipo NetworkPolicy.

> IMPORTANTE: Quando não há uma politica definida, os pods e services são acessíveis a todo o cluster, entretanto quando é criado uma política de rede, apenas o que estiver estipulado nas políticas é que será liberado.

### Funcionamento do NetworkPolicy

O NetworkPolicy atua no tráfego de entrada e saida de um pod ou namespace. Para isso ele permite selecionar pods e namespaces pelos seus labels. Isso permite que regras sejam facilmente aplicadas quando um pod ou namespace contém um determinado label.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

**Campos obrigatórios**: Como todos os outros objetos em um cluster kubernetes, uma **NetworkPolicy** precisa dos campos **apiVersion**, **Kind** e **metadata**.

**spec**: esta seção trás informações que são necessárias para definir uma política de rede para um determinado projeto/namespace.

**podSelector**: Cada **NetworkPolicy** inclue um **podSelector** que é usado para agrupar os pods a que a **NetworkPolicy** se aplica. No exemplo acima esta politica de rede está sendo aplicado apenas aos pods que contém o label 'role=db' 

Quando o valor de podSelector está vazio a regra será aplicada a todos os pods no namespace.

**policyTypes**: Cada **NetworkPolicy** inclue uma **policyTypes**, contém uma lista que pode ser do tipo **Ingress**, **Egress** ou ambas. O campo **policyTypes** indica se a política se aplica ou não ao tráfego de entrada (ingress) ou saida (egress) para o pod selecionado.

**ingress**: Cada **NetworkPolicy** pode incluir regra de entrada. Cada regra de entrada permite o tráfego para o que for definido no **from** para as portas especificadas na seção **ports**.
Na política do exemplo contém uma regra única, que corresponde ao tráfego em uma única porta, de uma das três origens, a primeira especificada por meio de um **ipBlock**, a segunda por meio de um **namespaceSelector** e a terceira por meio de um **podSelector**.

**egress**: Cada **NetworkPolicy** pode incluir uma lista de regras de saída. Cada regra permite o tráfego que corresponde para os destinos definidos nas seções **to** e **ports**.

A política do exemplo contém uma única regra, que corresponde ao tráfego em uma única porta para qualquer destino em 10.0.0.0/24. Também é possível definir um **namespaceSelector** e um **podSelector**.

Então no exemplo temos uma **NetworkPolicy**:




Abaixo vemos alguns exemplos de políticas de rede:

* Negar todo tráfego.

Para tornar um projeto "negar por padrão", adicione um objeto NetworkPolicy que corresponda a todos os pods, mas não aceite tráfego.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-by-default
spec:
  podSelector:
  ingress: []
```



Fontes:
https://docs.openshift.org/latest/architecture/networking/networking.html
https://docs.openshift.org/latest/architecture/networking/sdn.html
https://docs.openshift.org/latest/admin_guide/managing_networking.html#admin-guide-networking-networkpolicy
https://kubernetes.io/docs/concepts/services-networking/network-policies/


