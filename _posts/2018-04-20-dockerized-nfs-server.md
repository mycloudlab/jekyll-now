---
published: true
layout: post
title: Servidor de nfs em docker
---
Recentemente precisei fazer um teste de um servidor nfs. Como gosto do docker procurei uma imagem que fizesse isso, e encontrei um excelente projeto, itsthenetwork/nfs-server-alpine, verifiquei o github e clonei o projeto.

Para criar um servidor nfs usei o comando:

```bash
docker run -d \
  --name nfs \
  --net=host \
  --privileged \
  -v <dir>/nfsshare \
  -e SYNC=true \
  -e SHARED_DIRECTORY=/nfsshare \
  itsthenetwork/nfs-server-alpine 
```

A imagem tem apenas 19MB.

E para testar fiz o seguinte:
```bash
# mkdir /mnt/teste
# mount -v <ip local>:/ /mnt/teste
# cd /mnt/teste
# touch arquivo
# mkdir dir
# umount /mnt/teste
```

ao verificar o diretório da imagem que foi compartilhado via nfsshare, os arquivos estavam lá.

Fica a dica!

> A alegria que se tem em pensar e aprender faz-nos pensar e aprender ainda mais. - Aristóteles
