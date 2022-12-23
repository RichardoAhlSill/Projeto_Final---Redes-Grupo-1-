# Projeto Final - Infraestrutura e Serviços de Redes (Grupo-1)
Projeto final do Grupo 1 da disciplina de Infraestrutura e Serviços de Redes 

```
Instituto Federal de Alagoas - Campus Arapiraca
Professor: Alaelson Jatobá
Turma: 913
Aluno: Daniel Berg Silva Souza | Julio Cesár dos Santos Oliveira | Kelvin Holanda Leão Otilio | Ricardo Alexandre da Silva
```

### **1) Instalação e configuração do GW**

### **2) Instalação e configuração do Samba**

### **3) Instalação e configuração do NS1 (DNS MASTER)**

O Bind9 ou Berkeley Internet Name Domain é um servidor utilizado para o protocólo DNS, na qual tem a serventia de garantir uma maior agilidade na navegação visto que permite que o usuário apenas lembre do hostname de um site ao invés de seu endereço IP, portanto é o Bind9 que irá permitir o uso deste protocolo no Ubuntu.



```
sudo apt-get install bind9 dnsutils bind9-doc 
```

<p><center> Figura X:  Instalando o Bind9.</center></p>   
<img src="IMAGES/NS1/1.png" alt="Imagens" title="Figura 1:  Entrando no usuário "redes"." width="500" height="auto" />

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

Após a edição deve-se salvar o arquivo e dar o comando ```sudo netplan apply``` para aplicar e ```ifconfig``` para verificar.

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/1.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/2.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />

<p><center> Figura X:  Configurando NETPLAN.</center></p>   
<img src="IMAGES/NS2/3.png" alt="Imagens" title="Configurando NETPLAN" width="1000" height="auto" />


#### 4.2) Alterando as configurações do Neplan

Para isso basta utilizar o comando: 

```
sudo nano /etc/netplan/00-instaler-config.yaml 
```

### **5) Instalação e configuração do Serviço WEB**

### **6) Instalação e configuração do BD**
