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



### **2) Instalação e configuração do Samba**

### **3) Instalação e configuração do NS1 (DNS MASTER)**

O Bind9 ou Berkeley Internet Name Domain é um servidor utilizado para o protocólo DNS, na qual tem a serventia de garantir uma maior agilidade na navegação visto que permite que o usuário apenas lembre do hostname de um site ao invés de seu endereço IP, portanto é o Bind9 que irá permitir o uso deste protocolo no Ubuntu.

#### 3.1) Instalando o Bind9

```
sudo apt-get install bind9 dnsutils bind9-doc 
```

<p><center> Figura X:  Instalando o Bind9.</center></p>   
<img src="IMAGES/NS1/1.png" alt="Imagens" title="Figura 1:  Entrando no usuário "redes"." width="500" height="auto" />

### **4) Instalação e configuração do NS2**

### **5) Instalação e configuração do Serviço WEB**

### **6) Instalação e configuração do BD**
