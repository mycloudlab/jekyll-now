---
published: true
layout: post
title: Servidor de DNS dinâmico
---
Um serviço que é necessário em uma infraestrutura é o DNS, este serviço efetua a tradução de nomes para IP's e vice-versa.

Para a montagem de um laboratório mais realístico com uma produção, necessitamos ter um serviço como este.

Portanto criei o projeto[mycloudlab/infra-dynamic-dns](https://github.com/mycloudlab/infra-dynamic-dns) que tem como objetivo criar um servidor dns em docker que pudesse ser usado para fazer forward, multiplas zonas, zonas reversas, e atualizações via nsupdate.

Como o objetivo é ter um serviço de dns funcional para laboratórios, foi ignorado a criação de chaves de segurança para a atualização do dns.

O projeto é bem simples, basta executar a imagem docker abaixo:

```bash
docker run \
    -e ZONES="example.com" \
    -e REVERSE_ZONES="0.168.192" \
    -e FORWARDERS="8.8.8.8; 8.8.4.4;" \
    --publish 53:53/tcp --publish 53:53/udp \
    -d -t mycloudlab/infra-dynamic-dns 
```

E você terá um servidor de dns completamente funcional configurado para o dominio example.com com uma zona reversa para a rede 192.168.0.0/24.

A atualização da zona pode ser feita via nsupdate como abaixo:

```bash
echo "
server localhost

; atualiza a zona example.com 
; criando o endereço test.example.com para o ip 192.168.0.1
zone example.com
update del test.example.com. A
update add test.example.com 1440 A 192.168.0.1


; atualiza a zona reversa 
; criando um apontamento do ip 192.168.0.1 para o endereço test.example.com
zone 0.168.192.in-addr.arpa
update del 1.0.168.192.in-addr.arpa. PTR
update add 1.0.168.192.in-addr.arpa. 300 PTR test.example.com

show
send
" | nsupdate 
```

Para testar a atualização usando dig:
```bash
dig @localhost test.example.com +short
192.168.0.1

dig @localhost  -x 192.168.0.1 +short
test.example.com.
```

Tudo isso com uma imagem de apenas 13.1MB.


