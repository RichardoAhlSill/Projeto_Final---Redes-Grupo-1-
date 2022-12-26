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
<img src="IMAGES/NS1/1.png" alt="Imagens" title="Figura 1:  Entrando no usuário "redes"." width="500" height="auto" />

### **4) Instalação e configuração do NS2**

### **5) Instalação e configuração do Serviço WEB**

### **6) Instalação e configuração do BD**
