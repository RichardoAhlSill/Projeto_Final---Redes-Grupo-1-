# **1) Instalação e configuração do GW**

## 1.1) Habilitando o firewall

``` 
$ sudo ufw allow ssh
```

## 1.2) Dando permissão para que o protocolo SSH funcione através do firewall.
``` 
$ sudo ufw allow ssh
```
![ufw_configs](https://user-images.githubusercontent.com/103438311/209577774-c2f3275b-ea5f-4a15-b690-0d52bc1e1157.png)

#### 1.3) Verificação do status do firewall
``` 
$ sudo ufw status
```
![checking-ufw-status](https://user-images.githubusercontent.com/103438311/209577914-395bdfd1-0502-4f60-b27f-7fdb69d5ee35.png)

## 1.4) Habilitando a tranferência de pacotes de WAN para LAN
Para habilitar essa funcionalidade de comunicação entre interface externa e interna do
gateway, feita com as interfaces ens160 e ens192, precisa-se configurar o arquivo ```/etc/ufw/sysctl.conf``` 
da seguinte forma: 
```
$ sudo nano /etc/ufw/sysctl.conf
net/ipv4/ip_forwarding=1 #basta descomentar esta linha do arquivo em questão
$ cat /etc/ufw/sysctl.conf
```
![gateway_configs](https://user-images.githubusercontent.com/103438311/209578356-86e87db7-5481-4b39-b217-428cc2699103.png)

## 1.5) Verificando o nome das interfaces que respondem pela configuração do arquivo acima
```
$ ifconfig
```
![ifconfig](https://user-images.githubusercontent.com/103438311/209578719-5c52fc2b-ca85-46dc-9021-631b6b3f742a.png)
 ```
 Interface WAN: ens160
 Interface LAN: ens192
 Interface LoopBack: lo
```

## 1.6) Configurando os IPs, DHCPs e Gateyas das interfaces WAN e LAN
```
$ sudo nano /etc/netplan/00-installer-config.yaml
$ sudo netplan apply
```
![netplan_apply](https://user-images.githubusercontent.com/103438311/209579503-21d5cc8b-a245-41f0-a503-98cd68d74e4e.png)

Após editar e aplicar as configurações, pode-se:
```
Para visualizar o arquivo: $ cat /etc/netplan/00-installer-config.yaml
```
![cat](https://user-images.githubusercontent.com/103438311/209579576-0ba94311-8101-4de1-a98a-cc1ee89e3268.png)

```
Para visualizar se as configurações foram aplicadas: $ ifconfig
```
![new_ifconfig](https://user-images.githubusercontent.com/103438311/209579639-0af99b8b-888c-4b41-a92e-e74b95ccf14c.png)

## 1.7) Agora, cria-se o arquivo ```rc.local``` no diretório /etc.
Este arquivo receberá um script responsável pelo redirecionamento de portas entre as interfaces, para
o recibimento de pacotes.
```
$ sudo nano /etc/rc.local
```
Em seguida, inseri-se o seguinte script: 
```
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i ens192 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i ens160 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i ens192 -o ens160 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i ens160 -o ens192 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE

# rc.local needs to exit with 0
# rc.local precisa sair com 0

exit 0
```

![rclocal](https://user-images.githubusercontent.com/103438311/210082175-35f99522-b977-4a51-a34e-b6630b7a7ace.png)


## 1.8) Converte-se o arquivo rc.local em executável
```
$ sudo chmod 755 /etc/rc.local
```
![execution_permission](https://user-images.githubusercontent.com/103438311/209580017-51a5143b-0b2d-4e27-9bec-ee71290c3d9c.png)

## 1.9) Reinicie a máquina para aplicação do script
```
$ sudo reboot
```
![reboot](https://user-images.githubusercontent.com/103438311/209580158-06ecc655-07ea-4f43-8d9a-130277a8464e.png)

## 1.9.1) Deixando a máquina samba atrás da máquina gw.
Coloca-se o gateway da interface ens192 da máquina GW no gateway4 da máquina samba
```
$ sudo nano /etc/netplan/00-installer-config.yaml
$ sudo netplan apply
$ cat /etc/netplan/00-installer-config.yaml
```
Em seguida, verifica-se se funcionou observando a tabela de rotas.
```
$ netstat -rn 
```
![samba_behind_VM_gw](https://user-images.githubusercontent.com/103438311/209580619-6e0de7da-6334-455e-9429-59cd87340e6a.png)

## 1.9.2) Fazendo o redirecionamento dos pacotes da rede externa para interna, mediante as portas 445, 139 e 53 do sistema, para deixar os serviços Gaeway, Samba e DNS Master disponíveis externamente

Para isso, é necessário retornar ao arquivo /etc/rc.local e adicionar o seguinte script:
```
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 445 -j DNAT --to 192.168.13.10:445
iptables -A FORWARD -p tcp -d 192.168.13.10 --dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 139 -j DNAT --to 192.168.13.10:139
iptables -A FORWARD -p tcp -d 192.168.13.10 --dport 139 -j ACCEPT

#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master
iptables -A PREROUTING -t nat -i ens160 -p udp --dport 53 -j DNAT --to 192.168.13.11:53
iptables -A FORWARD -p udp -d 192.168.13.11 --dport 53 -j ACCEPT
```
* Testando a conexão com o comando telnet no servidor Gateway, na porta 445, temos: 
```
$ telnet 10.9.13.107 445
```
![telnet](https://user-images.githubusercontent.com/103438311/209857178-4dc7ae9a-7594-4e09-b473-350a1c33be5e.png)

* Testando a conexão com o comando telnet no servidor Gateway, na porta 139, temos: 
```
$ telnet 10.9.13.107 139
```
![telnet139](https://user-images.githubusercontent.com/103438311/209857527-3da8b42b-d925-49e7-9dfd-56d7c546b76b.png)

* Testando a conexão com o comando telnet no servidor Samba, na porta 445, temos: 
```
$ telnet 10.9.13.119 445
```
![smb_service_avaliable_externaly](https://user-images.githubusercontent.com/103438311/209807323-f9de238b-8e0d-4425-b790-5658ee04e574.png)


* Testando a conexão com o comando telnet no servidor Samba, na porta 139, temos: 
```
$ telnet 10.9.13.119 139
```
![sambaaa](https://user-images.githubusercontent.com/103438311/209807887-823f09cf-34b1-4f91-b36e-9370d89487bd.png)


* Testando a conexão com o comando telnet no servidor DNS Master, na porta 53, temos:
```
$ telnet 10.9.13.121 53
```
![dns_master_avaliable_externally](https://user-images.githubusercontent.com/103438311/209807471-4995ae23-7a60-485d-8083-33f43bf42125.png)


## 1.9.3) Acessando o servidor Samba, através do IP do servidor Gateway no host

[gateway_samba.webm](https://user-images.githubusercontent.com/103438311/210828165-64130029-ac13-416f-8b83-cceeccf2bd89.webm)


