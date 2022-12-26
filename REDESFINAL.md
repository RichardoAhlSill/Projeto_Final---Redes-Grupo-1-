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
