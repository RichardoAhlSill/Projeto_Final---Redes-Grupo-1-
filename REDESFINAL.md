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

#### 3.1) Instalando o Bind9

```
sudo apt-get install bind9 dnsutils bind9-doc 
```

<p><center> Figura X:  Instalando o Bind9.</center></p>   
<img src="IMAGES/NS1/1.png" alt="Imagens" title="Instalando o Bind9." width="500" height="auto" />

#### 3.2) Verificando o status do serviço Bind9

```
sudo systemctl status bind9 
```

<p><center> Figura X:  Verificando o status do serviço.</center></p>   
<img src="IMAGES/NS1/2.png" alt="Imagens" title="Verificando o status do serviço." width="500" height="auto" />

#### 3.3) Verificando os diretórios do Bind

```
ls /etc/bind
```

<p><center> Figura X:  Verificando os diretórios do Bind.</center></p>   
<img src="IMAGES/NS1/3.png" alt="Imagens" title="Verificando os diretórios do Bind." width="500" height="auto" />

#### 3.4) Criando um diretório para as "zones"

Criando um diretório para armazenar os arquivos das zonas.

```
sudo mkdir /etc/bind/zones
```

<p><center> Figura X:  Criando um diretório para zonas.</center></p>   
<img src="IMAGES/NS1/4.png" alt="Imagens" title="Criando um diretório para zonas." width="500" height="auto" />

#### 3.5) Copiando Banco de Dados para o nosso domínio (Zona Direta) 

Fazendo uma cópia do arquivo db.empty para o db.grupo1.turma913.ifalara.local, isso na Zona Direta

```
sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo1.turma913.ifalara.local
```

<p><center> Figura X:  Copiando Banco de Dados (Zona Direta).</center></p>   
<img src="IMAGES/NS1/5.png" alt="Imagens" title="Copiando Banco de Dados (Zona Direta)" width="500" height="auto" />

#### 3.6) Copiando Banco de Dados para o nosso domínio (Zona Reversa)

Utilizado para quando não se conhece o endereço IP, mas sabe-se o nome do host.
Para isso, faz-se uma cópia do arquivo db.127 para o db.10.9.13.rev, isso na Zona Reversa.

```
sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.13.rev
```

<p><center> Figura X:  Copiando Banco de Dados (Zona Reversa).</center></p>   
<img src="IMAGES/NS1/6.png" alt="Imagens" title="Copiando Banco de Dados (Zona Reversa)." width="500" height="auto" />

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
<img src="IMAGES/NS1/7.png" alt="Imagens" title="Editando Banco de Dados (Zona Direta)." width="500" height="auto" />

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
<img src="IMAGES/NS1/8.png" alt="Imagens" title="Editando Banco de Dados (Zona Reversa)." width="500" height="auto" />

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
<img src="IMAGES/NS1/9.png" alt="Imagens" title="Ativando os arquivos das zonas." width="500" height="auto" />

#### 3.10) Checando a sintaxe do arquivo de ativação

```
sudo named-checkconf
```

<p><center> Figura X:  Checando a sintaxe do arquivo de ativação. </center></p>   
<img src="IMAGES/NS1/10.png" alt="Imagens" title="Checando a sintaxe do arquivo de ativação." width="500" height="auto" />

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
<img src="IMAGES/NS1/11.png" alt="Imagens" title="Checando a sintaxe dos arquivo de dados 1." width="500" height="auto" />

<p><center> Figura X:  Checando a sintaxe dos arquivo de dados 2. </center></p>   
<img src="IMAGES/NS1/12.png" alt="Imagens" title="Checando a sintaxe dos arquivo de dados 2." width="500" height="auto" />

### **4) Instalação e configuração do NS2**

### **5) Instalação e configuração do Serviço WEB**

### **6) Instalação e configuração do BD**
