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

## Configurando o tipo de rede do cluster (ansible)

A configuração do tipo de rede do cluster pode ser feito via variável **os_sdn_network_plugin_name**.

Os aceitos os valores:

* redhat/openshift-ovs-subnet
* redhat/openshift-ovs-multitenant
* redhat/openshift-ovs-networkpolicy

O valor default desta variável é 'redhat/openshift-ovs-subnet'.

## Rede do tipo ovs-multitenant

Quando seu cluster está configurado para utilizar uma rede ovs-multitenant 
