# Projeto Final - Infraestrutura e Serviços de Redes (Grupo-1)
Projeto final do Grupo 1 da disciplina de Infraestrutura e Serviços de Redes 

```
Instituto Federal de Alagoas - Campus Arapiraca
Professor: Alaelson Jatobá
Turma: 913
Aluno: Daniel Berg Silva Souza | Julio Cesár dos Santos Oliveira | Kelvin Holanda Leão Otilio | Ricardo Alexandre da Silva
```

### **1) Instalação e configuração do GW**

#### 1.1) Habilitando o firewall

``` 
$ sudo ufw allow ssh
```

#### 1.2) Dando permissão para que o protocolo SSH funcione através do firewall.
``` 
$ sudo ufw allow ssh
```
![ufw_configs](https://user-images.githubusercontent.com/103438311/209577774-c2f3275b-ea5f-4a15-b690-0d52bc1e1157.png)

#### 1.3) Verificação do status do firewall
``` 
$ sudo ufw status
```
![checking-ufw-status](https://user-images.githubusercontent.com/103438311/209577914-395bdfd1-0502-4f60-b27f-7fdb69d5ee35.png)

#### 1.4) Habilitando a tranferência de pacotes de WAN para LAN
Para habilitar essa funcionalidade de comunicação entre interface externa e interna do
gateway, feita com as interfaces ens160 e ens192, precisa-se configurar o arquivo ```/etc/ufw/sysctl.conf``` 
da seguinte forma: 
```
$ sudo nano /etc/ufw/sysctl.conf
net/ipv4/ip_forwarding=1 #basta descomentar esta linha do arquivo em questão
$ cat /etc/ufw/sysctl.conf
```
![gateway_configs](https://user-images.githubusercontent.com/103438311/209578356-86e87db7-5481-4b39-b217-428cc2699103.png)

#### 1.5) Verificando o nome das interfaces que respondem pela configuração do arquivo acima
```
$ ifconfig
```
![ifconfig](https://user-images.githubusercontent.com/103438311/209578719-5c52fc2b-ca85-46dc-9021-631b6b3f742a.png)
 ```
 Interface WAN: ens160
 Interface LAN: ens192
 Interface LoopBack: lo
```

#### 1.6) Configurando os IPs, DHCPs e Gateyas das interfaces WAN e LAN
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

#### 1.7) Agora, cria-se o arquivo ```rc.local``` no diretório /etc.
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
![rc-local-configs](https://user-images.githubusercontent.com/103438311/209579929-5edf2e54-10f7-413e-872b-9659c16b555d.png)

#### 1.8) Converte-se o arquivo rc.local em executável
```
$ sudo chmod 755 /etc/rc.local
```
![execution_permission](https://user-images.githubusercontent.com/103438311/209580017-51a5143b-0b2d-4e27-9bec-ee71290c3d9c.png)

#### 1.9) Reinicie a máquina para aplicação do script
```
$ sudo reboot
```
![reboot](https://user-images.githubusercontent.com/103438311/209580158-06ecc655-07ea-4f43-8d9a-130277a8464e.png)

#### 1.9.1) Deixando a máquina samba atrás da máquina gw.
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

#### 1.9.2) Fazendo o redirecionamento dos pacotes da rede externa para interna, mediante as portas 445, 139 e 53 do sistema, para deixar os serviços Gaeway, Samba e DNS Master disponíveis externamente

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
iptables -A FORWARD -p udp -d 192.168.13.11 --dport 53 -j ACCEP
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



### **2) Instalação e configuração do Samba**


#### 2.1) Alterando nome da VM

```
$ sudo hostnamectl set-hostname samba-srv
$ reboot
```
<img src="IMAGES-SAMBA/NameSambaSetado.png" alt="Imagens" title="Figura 2.1:  Alterando nome da VM." width="300" height="auto" />

Nome da VM Samba alterado para samba-srv.

#### 2.2) Instalando o servidor Samba na VM samba-srv

```
$ sudo apt update
$ sudo apt install samba
```

#### 2.2) Verificar se o Samba está rodando

```
$ whereis samba
$ sudo systemctl status smbd
$ netstat -an | grep LISTEN
```

<img src="IMAGES-SAMBA/systemctl_status.png" alt="Imagens" title="Figura 2.2:  Verificar se a VM está rodando." width="1000" height="auto" />
<img src="IMAGES-SAMBA/netstat.png" alt="Imagens" title="Figura 2.3:  Verificar se a VM está rodando." width="700" height="auto" />

#### 2.3) Backup do arquivo de configuração do Samba e criação de um novo arquivos com os comandos necessários.

```
$ sudo cp /etc/samba/smb.conf{,.backup}
$ ls -la
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
$ sudo nano /etc/samba/smb.conf
```

<img src="IMAGES-SAMBA/samba_config_part2.png" alt="Imagens" title="Figura 2.5:  Configuração Samba." width="700" height="auto" />

falta imagens.

#### 2.4) Editando o arquivo de configuração /etc/samba/smb.conf

```
$ sudo nano /etc/samba/smb.conf
$ sudo systemctl restart smbd
$ sudo systemctl restart smbd
$ cat /etc/samba/smb.conf
```
Linha de interfaces alterada para "10.9.13.1/24 ens160 ens192 enp0s3"

<img src="IMAGES-SAMBA/samba_config_part1.png" alt="Imagens" title="Figura 2.4:  Configuração Samba." width="700" height="auto" />

* Logo em seguida deve-se criar um usuário do S.O para utilizar o compartilhamento Samba.

```
* usuário: aluno
* senha: alunoifal
```
```
$ sudo adduser aluno
```
* Vinculando o usuário do S.O ao Serviço Samba para acessar o compartilhamento de arquivo. Neste caso repetiremos a senha do usuário aluno.

```
$ sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.
$ sudo usermod -aG sambashare aluno
```
<img src="IMAGES-SAMBA/smbpasswd.png" alt="Imagens" title="Figura 2.5:  Vinculando o usuário do S.O." width="700" height="auto" />

* Agora que o Samba já encontra-se instalado basta criar um diretório para que possamos compartilhá-lo em rede.


```
$ mkdir /home/<username>/sambashare/
$ sudo mkdir -p /samba/public
```
<img src="IMAGES-SAMBA/create_file_at_samba_public_directory.png" alt="Imagens" title="Figura 2.6:  diretório de compartilhamento do Samba." width="700" height="auto" />

* Em seguida devemos configurar as permissões para que qualquer um possa acessar o compartilhamento público.

```
sudo chown -R nobody:nogroup /samba/public
sudo chmod -R 0775 /samba/public
sudo chgrp sambashare /samba/public

```

<img src="IMAGES-SAMBA/samba_permissions.png" alt="Imagens" title="Figura 2.6:  configurando permissões." width="700" height="auto" />

#### 2.5) Cliente de compartilhamento

* Para o último passo basta em uma máquina digite no Winndows Explorer o endereço IP do servidor samba da seguinte forma:
**\\ip_do_maquina**. Exemplo: \\10.9.13.119

<img src="IMAGES-SAMBA/samba_directories.png" alt="Imagens" title="Figura 2.7:  conectando ao servidor." width="700" height="auto" />
<img src="IMAGES-SAMBA/samba_connecting.png" alt="Imagens" title="Figura 2.8:  conectando ao servidor." width="700" height="auto" />

* Vídeo realizando a conexão e acessando o arquivo de texto criado anteriormente

<img src="IMAGES-SAMBA/samba_connection.gif" alt="Imagens" title="Figura 2.9:  conectando ao servidor." width="700" height="auto" />

### **3) Instalação e configuração do NS1 (DNS MASTER)**

O Bind9 ou Berkeley Internet Name Domain é um servidor utilizado para o protocólo DNS, na qual tem a serventia de garantir uma maior agilidade na navegação visto que permite que o usuário apenas lembre do hostname de um site ao invés de seu endereço IP, portanto é o Bind9 que irá permitir o uso deste protocolo no Ubuntu.

#### 3.1) Instalando o Bind9

```
sudo apt-get install bind9 dnsutils bind9-doc 
```

<p><center> Figura X:  Instalando o Bind9.</center></p>   
<img src="IMAGES/NS1/1.png" alt="Imagens" title="Instalando o Bind9." width="1000" height="auto" />

#### 3.2) Verificando o status do serviço Bind9

```
sudo systemctl status bind9 
```

<p><center> Figura X:  Verificando o status do serviço.</center></p>   
<img src="IMAGES/NS1/2.png" alt="Imagens" title="Verificando o status do serviço." width="1000" height="auto" />

#### 3.3) Verificando os diretórios do Bind

```
ls /etc/bind
```

<p><center> Figura X:  Verificando os diretórios do Bind.</center></p>   
<img src="IMAGES/NS1/3.png" alt="Imagens" title="Verificando os diretórios do Bind." width="1000" height="auto" />

#### 3.4) Criando um diretório para as "zones"

Criando um diretório para armazenar os arquivos das zonas.

```
sudo mkdir /etc/bind/zones
```

<p><center> Figura X:  Criando um diretório para zonas.</center></p>   
<img src="IMAGES/NS1/4.png" alt="Imagens" title="Criando um diretório para zonas." width="1000" height="auto" />

#### 3.5) Copiando Banco de Dados para o nosso domínio (Zona Direta) 

Fazendo uma cópia do arquivo db.empty para o db.grupo1.turma913.ifalara.local, isso na Zona Direta

```
sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo1.turma913.ifalara.local
```

<p><center> Figura X:  Copiando Banco de Dados (Zona Direta).</center></p>   
<img src="IMAGES/NS1/5.png" alt="Imagens" title="Copiando Banco de Dados (Zona Direta)" width="1000" height="auto" />

#### 3.6) Copiando Banco de Dados para o nosso domínio (Zona Reversa)

Utilizado para quando não se conhece o endereço IP, mas sabe-se o nome do host.
Para isso, faz-se uma cópia do arquivo db.127 para o db.10.9.13.rev, isso na Zona Reversa.

```
sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.13.rev
```

<p><center> Figura X:  Copiando Banco de Dados (Zona Reversa).</center></p>   
<img src="IMAGES/NS1/6.png" alt="Imagens" title="Copiando Banco de Dados (Zona Reversa)." width="1000" height="auto" />

#### 3.7) Editando o Banco de Dados para o nosso domínio (Zona Direta)

```
sudo nano db.grupo1.turma913.ifalara.local 
```

Editar colocando as informações contidas na planilha.


```
;
; BIND data file for internal network
;
$ORIGIN grupo1.turma913.ifalara.local.
$TTL	3h
@	IN	SOA	ns1.grupo1.turma913.ifalara.local. root.grupo1.turma913.ifalara.local. (
	  	      2022122201	; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.grupo1.turma913.ifalara.local.
@	IN	NS	ns2.grupo1.turma913.ifalara.local.
;hosts
ns1.grupo1.turma913.ifalara.local.	  IN	A	10.9.13.121
ns2.grupo1.turma913.ifalara.local.	  IN	A	10.9.13.129
smb.grupo1.turma913.ifalara.local.	  IN	A	10.9.13.119
gw.grupo1.turma913.ifalara.local.	  IN 	A	10.9.13.107
www.grupo1.turma913.ifalara.local.	  IN 	A	10.9.13.211
db.grupo1.turma913.ifalara.local.	  IN 	A	10.9.13.212
```

<p><center> Figura X:  Editando Banco de Dados (Zona Direta).</center></p>   
<img src="IMAGES/NS1/image.png" alt="Imagens" title="Editando Banco de Dados (Zona Direta)." width="1000" height="auto" />

#### 3.8) Editando o Banco de Dados para o nosso domínio (Zona Reversa)

```
sudo nano db.10.9.13.rev 
```

Editar colocando as informações contidas na planilha.


```
;
; BIND reverse data file of reverse zone for local area network 10.9.13.0/24
;
$TTL    604800
@       IN      SOA     grupo1.turma913.ifalara.local. root.grupo1.turma913.ifalara.local. (
                     2022122200         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
@      IN      NS      ns1.grupo1.turma913.ifalara.local.
@      IN      NS      ns2.grupo1.turma913.ifalara.local.

; PTR Records
121   IN      PTR     ns1.grupo1.turma913.ifalara.local.              ; 10.9.13.121
129   IN      PTR     ns2.grupo1.turma913.ifalara.local.              ; 10.9.13.129
119   IN      PTR     smb.grupo1.turma913.ifalara.local.    	      ; 10.9.13.119
107   IN      PTR     gw.grupo1.turma913.ifalara.local.               ; 10.9.13.107
211   IN      PTR     www.grupo1.turma913.ifalara.local.              ; 10.9.13.211
212   IN      PTR     bd.grupo1.turma913.ifalara.local.               ; 10.9.13.212
```

<p><center> Figura X:  Editando Banco de Dados (Zona Reversa).</center></p>   
<img src="IMAGES/NS1/8.png" alt="Imagens" title="Editando Banco de Dados (Zona Reversa)." width="1000" height="auto" />

#### 3.9) Ativando os arquivos das zonas

```
sudo nano /etc/bind/named.conf.local
```

Inserir as informações de acordo com os arquvios db


```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "grupo1.turma913.ifalara.local" {
	type master;
	file "/etc/bind/zones/db.grupo1.turma913.ifalara.local";
	allow-transfer{ 10.9.13.11; };  
	allow-query{any;};
};

zone "13.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.13.rev";
	allow-transfer{ 10.9.13.11; };
};
```

<p><center> Figura X:  Ativando os arquivos das zonas.</center></p>   
<img src="IMAGES/NS1/9.png" alt="Imagens" title="Ativando os arquivos das zonas." width="1000" height="auto" />

#### 3.10) Checando a sintaxe do arquivo de ativação

```
sudo named-checkconf
```

<p><center> Figura X:  Checando a sintaxe do arquivo de ativação. </center></p>   
<img src="IMAGES/NS1/10.png" alt="Imagens" title="Checando a sintaxe do arquivo de ativação." width="1000" height="auto" />

#### 3.11) Checando a sintaxe dos arquivo de dados

Antes deve-se entrar no diretório das zonas

```
cd /etc/bind/zones
```
Logo após digitar o comando:

```
sudo named-checkzone grupo1.turma913.ifalara.local db.grupo1.turma913.ifalara.local
```

Caso o resultado retornado seja: 

```
zone labredes.ifalarapiraca.local/IN: loaded serial 1
OK
```

Esta tudo funcionando perfeitamente, o mesmo vale para o arquivo seguinte.

```
sudo named-checkzone 13.9.10.in-addr.arpa db.10.9.13.rev
```
Retorno:

```
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK
```

<p><center> Figura X:  Checando a sintaxe dos arquivo de dados 1. </center></p>   
<img src="IMAGES/NS1/11.png" alt="Imagens" title="Checando a sintaxe dos arquivo de dados 1." width="1000" height="auto" />

<p><center> Figura X:  Checando a sintaxe dos arquivo de dados 2. </center></p>   
<img src="IMAGES/NS1/12.png" alt="Imagens" title="Checando a sintaxe dos arquivo de dados 2." width="1000" height="auto" />

#### 3.12) Configuração para resolver apenas endereço IPv4

```
sudo nano /etc/default/named
```
Editando ou adicionando apenas ```OPTIONS="-4 -u bind"``` a última linha.
Ficando dessa forma:

```
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
```

<p><center> Figura X:  Configuração para IPv4.</center></p>   
<img src="IMAGES/NS1/13.png" alt="Imagens" title="Configuração para IPv4." width="1000" height="auto" />

#### 3.13) Restartando ou Iniciando o Bind9

Utilizando o comando ```sudo systemctl restart bind9``` para restartar.
E ```sudo systemctl enable bind9``` caso seja necessáro.


<p><center> Figura X:  Restartando ou Iniciando o Bind9.</center></p>   
<img src="IMAGES/NS1/14.png" alt="Imagens" title="Restartando ou Iniciando o Bind9" width="1000" height="auto" />

#### 3.14) Configuração para os clientes

Para isso, deve-se editar o arquivo:

```
sudo vi /etc/netplan/00-installer-config.yaml
```

Basta adicionar algumas informações para a configuração dos clientes.
Entre elas, alterar o os endereços ns1 e ns2 e o domínio para aqueles definidos na planilha.

```
# This is the network config written by 'subiquity'
network:
  renderer: networkd
  ethernets:
    ens160:
      dhcp4: false
      addresses: [10.9.13.121/24]
      gateway4: 10.9.13.1 
      nameservers:
         addresses:
           - 10.9.13.121 #ns1
           - 10.9.13.129 #ns2
         search: [grupo1.turma913.ifalara.local] #domínio                                                                                                                                
    ens192:
      dhcp4: false
      addresses: [192.168.13.11/28]
      #gateway4: 192.168.13.1 
      #nameservers:
         #addresses:
           #- 8.8.8.8 
           #- 8.8.4.4
         #search: []
  version: 2  
```


<p><center> Figura X:  Configuração para os clientes.</center></p>   
<img src="IMAGES/NS1/15.png" alt="Imagens" title="Configuração para os clientes." width="1000" height="auto" />

#### 3.15) Testes do Servidor

Primeiro iremos testar se os campos DNS servers e DNS Domain estão corretos, ou seja, se estão de acordo com a configuração do cliente que acabamos de editar.
Para isso, basta utilizar o comando:

```
resolvectl status ens160
```

No qual deve retornar os mesmo valores que inserimos no passo anterior, como pode ser observado na figura abaixo.

<p><center> Figura X:  Testando se os campos DNS servers e DNS Domain estão corretos.</center></p>   
<img src="IMAGES/NS1/16.png" alt="Imagens" title="Testando se os campos DNS servers e DNS Domain estão corretos." width="1000" height="auto" />


Após esse teste, iremos testar o serviço para a máquina ns1.
Para isso, basta utilizar o comando:

```
dig ns1.grupo1.turma913.ifalara.local
```

No qual deve retornar o resultado da pesquisa na linha ```ANSWER SECTION```, como pode ser visto na figura abaixo.

<p><center> Figura X:  Testando o servidor na máquina ns1.</center></p>   
<img src="IMAGES/NS1/17.png" alt="Imagens" title="Testando o servidor na máquina ns1." width="1000" height="auto" />

E por fim testar o serviço DNS Reverso.
Primeiramente na máquina com ns1, digitando o comando de acordo com o seu IP da planilha.

```
dig -x 10.9.13.121
```

Retornando este resultado da figura abaixo.

<p><center> Figura X:  Testando o DNS Reverso na máquina ns1.</center></p>   
<img src="IMAGES/NS1/18.png" alt="Imagens" title="Testando o DNS Reverso na máquina ns1." width="1000" height="auto" />

E após, fazer o mesmo para a máquina ns2.

```
dig -x 10.9.13.129
```
Como mostra na figura abaixo.

<p><center> Figura X:  Testando o DNS Reverso na máquina ns2.</center></p>   
<img src="IMAGES/NS1/19.png" alt="Imagens" title="Testando o DNS Reverso na máquina ns2." width="1000" height="auto" />


### **4) Instalação e configuração do NS2 (DSN SLAVE)**

#### 4.1) Alterando as configurações do Neplan

Para isso basta utilizar o comando: 

```
sudo nano /etc/netplan/00-instaler-config.yaml 
```

Inserindo as informações necessárias de acordo com a planilha.

```
network:
    ethernets:
        ens160:                        # interface local
            addresses: [10.9.13.129/24]  # ip/mascara
            gateway4: 10.9.13.1         # ip do gateway
            dhcp4: false               # 'false' para conf. estatica 
            nameservers:               # servidores dns
                addresses:
                - 10.9.13.121            # ip do ns1
                - 10.9.13.129            # ip do ns2
                search: [grupo1.turma913.ifalara.local]  # domínio
    version: 2
```

Após a edição deve-se salvar o arquivo e dar o comando ```sudo netplan apply``` para aplicar e ```ifconfig``` para verificar se funcionou.

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/1.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/2.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/3.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />


#### 4.2) Instalando o Bind9

Agora basta instalar o Bind9. Para isso basta utilizar o comando: 

```
sudo apt-get install bind9 dnsutils bind9-doc -y
```

<p><center> Figura X:  Instalando Bind9.</center></p>   
<img src="IMAGES/NS2/4.png" alt="Imagens" title="Instalando Bind9" width="1000" height="auto" />

#### 4.3) Verificando o status do Bind9

Para isso basta utilizar o comando: 

```
sudo systemctl status bind9
```

E usar o comando ```sudo systemctl status bind9``` para ativar, caso não esteja ativado.

<p><center> Figura X:  Verificando status do Bind9.</center></p>   
<img src="IMAGES/NS2/5.png" alt="Imagens" title="Verificando status do Bind9." width="1000" height="auto" />

#### 4.3) Verificando o status do Bind9

Para isso basta utilizar o comando: 

```
sudo systemctl status bind9
```

E usar o comando ```sudo systemctl status bind9``` para ativar, caso não esteja ativado.

<p><center> Figura X:  Verificando status do Bind9.</center></p>   
<img src="IMAGES/NS2/5.png" alt="Imagens" title="Verificando status do Bind9." width="1000" height="auto" />

<p><center> Figura X:  Ativando o Bind9.</center></p>   
<img src="IMAGES/NS2/6.png" alt="Imagens" title=" Ativando o Bind9." width="1000" height="auto" />

#### 4.4) Configurando as zonas

Para isso basta utilizar o comando ```sudo nano /etc/bind/named.conf.local``` e alterar de acordo com os arquivos necessários.

```
zone "grupo1.turma913.ifalara.local" {
  type slave;
  file "/etc/bind/zones/db.grupo1.turma913.ifalara.local";
  masters { 10.9.13.129; };
};

zone "13.9.10.in-addr.arpa" IN {
  type slave;
  file "/etc/bind/zones/db.10.9.13.rev";
  masters { 10.9.13.129; };
};
```

<p><center> Figura X:  Configurando Zonas.</center></p>   
<img src="IMAGES/NS2/7.png" alt="Imagens" title="Configurando Zonas." width="1000" height="auto" />

<p><center> Figura X:  Configurando Zonas.</center></p>   
<img src="IMAGES/NS2/8.png" alt="Imagens" title="Configurando Zonas." width="1000" height="auto" />

#### 4.5) Checar a sintaxe da configuração
Para isso basta utilizar o comando: 

```
sudo named-checkconf
```

Caso não retorne nenhum erro, significa que está tudo funcionando corretamente.

<p><center> Figura X:  Checando sintaxe.</center></p>   
<img src="IMAGES/NS2/9.png" alt="Imagens" title="Checando sintaxe." width="1000" height="auto" />

#### 4.6) Testes
Iremos usar o comando ```dig``` para fazer o teste. 
Para isso utilizaremos o comando:

```
dig @10.9.13.121 ns1.grupo1.turma913.ifalara.local
```

<p><center> Figura X:  Teste.</center></p>   
<img src="IMAGES/NS2/10.png" alt="Imagens" title="Teste." width="1000" height="auto" />

### **5) Instalação e configuração do Serviço WEB**

### **6) Instalação e configuração do BD**
