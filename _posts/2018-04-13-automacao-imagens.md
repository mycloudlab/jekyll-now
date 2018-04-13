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

Crie a seguinte estrutura de pasta:

```bash
packer
  |
  +-- http
```

Primeiro precisamos entender que a instalação tem diversos passos como configurar usuário, setar timezone, configuração de bootloader, partições de disco rede, firewall, etc... 

A redhat criou um projeto chamado [kickstart](https://en.wikipedia.org/wiki/Kickstart_(Linux)), que é usado para definir todos os passos acima em código, e diversos sistemas operacionais são compativeis com o kickstart!

o kickstart é fornecido via rede (http) para o sistema de build da imagem, então na pasta http devemos criar um arquivo chamado **anaconda-ks.cfg**:

```text
#version=DEVEL
# System authorization information
#auth --enableshadow --passalgo=sha512
# Use CDROM installation media
#cdrom
# Use graphical install
#graphical
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=br --xlayouts='br'
# System language
lang en_US.UTF-8


eula --agreed
reboot

# Network information
network  --bootproto=dhcp --device=enp0s3 --onboot=off --noipv6 

# Root password
rootpw --plaintext root

firewall --service=ssh

# System services
services --enabled=NetworkManager,sshd,chronyd

# System timezone
timezone America/New_York --isUtc

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm

# Partition clearing information
clearpart --none --initlabel

%packages
@^minimal
@core
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```

Agora vamos fazer o template e criar o script de provisionamento, na pasta packer crie o arquivo **centos-7-base.json**:



```json
{   
    "builders": [
        {
            "type": "virtualbox-iso",
            "boot_command": [
                "<tab> <wait>",
                "text ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/anaconda-ks.cfg <wait>",
                "<enter> <wait>"
            ],
            "boot_wait": "5s",
            "disk_size": 81920,
            "guest_os_type": "RedHat_64",
            "headless": false,
            "http_directory": "http",
            "iso_urls": [
                "http://centos.pop-es.rnp.br/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso"
            ],
            "iso_checksum_type": "md5",
            "iso_checksum": "5848f2fd31c7acf3811ad88eaca6f4aa",
            "ssh_username": "root",
            "ssh_password": "root",
            "ssh_port": 22,
            "ssh_wait_timeout": "360s",
            "shutdown_command": "echo 'vagrant'|sudo -S /sbin/halt -h -p",
            "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso",
            "virtualbox_version_file": ".vbox_version",
            "vm_name": "centos-7-x86_64-base",
            "output_directory": "output-iso-base",
            "vboxmanage": [
                [
                    "modifyvm",
                    "{{.Name}}",
                    "--memory",
                    "512"
                ],
                [
                    "modifyvm",
                    "{{.Name}}",
                    "--cpus",
                    "2"
                ]
            ]
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "script": "configure-image.sh"
        }
    ]
}
```

Com isso podemos agora criar o arquivo configure-image.sh que será usado para configurar o servidor após instalação do SO:

Crie na pasta packer o arquivo **configure-image.sh**, neste arquivo você pode colocar todos os comandos que desejar para configurar o novo servidor, para uma instalação no virtualbox costumo fazer a instalação do vbox guest additions, pois melhora o suporte do virtualbox ao servidor virtual, abaixo segue o conteúdo que faz a instalação:


Feito isso agora efetuamos o build via packer com o comando abaixo(o packer deve ser instalado antes):
```bash
packer build --force centos-7-base.json
```
Este comando deve ser executado de dentro da pasta do packer.

No final do processo será criado uma pasta **output-iso-base** que conterá a imagem do sistema no formato vmdk, agora basta importar no virtualbox e ser feliz!









