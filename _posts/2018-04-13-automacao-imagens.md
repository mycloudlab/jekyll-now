---
published: false
layout: post
title: Automatize a criação de templates de imagem
---

O docker representou uma mudança muito grande de paradigma, tanto para o desenvolvimento de aplicações como para a infraestrutura que sustenta estas aplicações.

Uma das facilidades do docker é a criação de uma imagem para execução das aplicações, entretanto o que poderia ser um equivalente na área de virtualização?

Geralmente as ferramentas de virtualização (vmware, virtualbox, kvm, xen, etc...) permitem a criação de máquinas de template, entretanto a criação destes templates, geralmente é feita na mão é com isso não podemos replicar o sucesso ou falha do processo.

Idealmente o melhor seria fazer a instalação do sistema operacional na vm de forma automatizada. Para isso existe uma de fazer isso, o projeto chamasse [packer](https://www.packer.io/).

Packer é um projeto da hashicorp que permite o build automatizado de imagens.

O build é definido em código usando um arquivo em json (template), um template permite a construção de imagens nas mais diversas tecnologias de virtualização, inclusive nuvem (amazon ec2).

Após a criação das imagens pode ser feito o provisionamento do servidor usando bashscript, ansible, puppet, chef, etc...

Pode ser definido um post-processador que após a geração do template, efetue a compactação da imagem, o envio para uma cloud, etc...

Vamos a parte prática:


