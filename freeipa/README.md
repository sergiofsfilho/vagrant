FreeIPA com 2 replicas identicas
SO: Fedora 24 cloud base
Versão: FreeIPA 4.4

[Em todos os servidores]
1. Edite o /etc/hosts com os IPs dos FreeIPAs
# vim /etc/hosts

2. Edite o /etc/resolv.conf
`
vim /etc/resolv.conf
search freeipa.lab
nameserver 8.8.8.8
nameserver 8.8.4.4
`

3. Verifique se o FQDN está correto. Se não, corrija-o
# hostnamectl set-hostname ipa1.freeipa.lab

[Instalando o FreeIPA "Master"]
1. Instale o servidor do FreeIPA com o comando abaixo.
ipa-server-install --realm=FREEIPA.LAB --domain=freeipa.lab --ds-password=muxi12345 --admin-password=muxi12345 --mkhomedir --hostname=ipa1.freeipa.lab --ip-address=172.16.101.11 --idstart=2000 --idmax=2999 --ssh-trust-dns --setup-dns --forwarder=8.8.8.8 --forwarder=8.8.4.4 --forward-policy=first --auto-reverse

[Instalando replicas]
1. No FreeIPA master, adicione os endereços PTR para resolução de DNS Reverso.
`kinit admin
ipa dnszone-find
ipa dnsrecord-find freeipa.lab.
ipa dnsrecord-add 
Record name: 12
Zone name: 101.16.172.in-addr.arpa.
Please choose a type of DNS resource record to be added
The most common types for this type of zone are: PTR

DNS resource record type: PTR
PTR Hostname: ipa2.freeipa.lab.
  Record name: 12
    PTR record: ipa2.freeipa.lab.
`

2. Aponte o DNS para o servidor inicial do FreeIPA
`
# vim /etc/resolv.conf
search freeipa.lab
nameserver 172.16.101.11
`
3. Instale a replica com os comandos abaixo
`
ipa-client-install --enable-dns-updates --mkhomedir --ssh-trust-dns --force-ntpd
kinit admin
ipa-replica-install --mkhomedir --ssh-trust-dns --setup-dns --setup-ca --auto-forwarders
`

[Nos clientes]
1. Instale o client com o comando abaixo.
`
yum install ipa-client -y
ipa-client-install --enable-dns-updates --mkhomedir --ssh-trust-dns --force-ntpd
`
