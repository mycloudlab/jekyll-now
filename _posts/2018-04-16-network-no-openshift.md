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

Quando seu cluster está configurado para usar o plug-in SDN ovs-multitenant, você pode gerenciar a rede para um conjunto de projetos usando o utilitário de linha de comando **oc**.

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





