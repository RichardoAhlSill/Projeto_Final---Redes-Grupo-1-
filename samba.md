# **2) Instalação e configuração do Samba**


## 2.1) Alterando nome da VM

```
$ sudo hostnamectl set-hostname samba-srv
$ reboot
```
<img src="IMAGES-SAMBA/NameSambaSetado.png" alt="Imagens" title="Figura 2.1:  Alterando nome da VM." width="300" height="auto" />

Nome da VM Samba alterado para samba-srv.

## 2.2) Instalando o servidor Samba na VM samba-srv

```
$ sudo apt update
$ sudo apt install samba
```

## 2.2) Verificar se o Samba está rodando

```
$ whereis samba
$ sudo systemctl status smbd
$ netstat -an | grep LISTEN
```

<img src="IMAGES-SAMBA/systemctl_status.png" alt="Imagens" title="Figura 2.2:  Verificar se a VM está rodando." width="1000" height="auto" />
<img src="IMAGES-SAMBA/netstat.png" alt="Imagens" title="Figura 2.3:  Verificar se a VM está rodando." width="700" height="auto" />

## 2.3) Backup do arquivo de configuração do Samba e criação de um novo arquivos com os comandos necessários.

```
$ sudo cp /etc/samba/smb.conf{,.backup}
$ ls -la
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
$ sudo nano /etc/samba/smb.conf
```
![samba](https://user-images.githubusercontent.com/103438311/210081544-477c9fcf-29e7-4e79-8a86-a2e2895f6040.png)



## 2.4) Editando o arquivo de configuração /etc/samba/smb.conf

```
$ sudo nano /etc/samba/smb.conf
$ sudo systemctl restart smbd
$ sudo systemctl restart smbd
$ cat /etc/samba/smb.conf
```
Linha de interfaces alterada para "127.0.0.1/8 ens160 ens192"
![interfaces](https://user-images.githubusercontent.com/103438311/210081380-92a71d43-c65e-44f9-b742-5985734bacf4.png)


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

## 2.5) Cliente de compartilhamento

* Para o último passo basta em uma máquina digite no Winndows Explorer o endereço IP do servidor samba da seguinte forma:
**\\ip_do_maquina**. Exemplo: \\10.9.13.119

<img src="IMAGES-SAMBA/samba_directories.png" alt="Imagens" title="Figura 2.7:  conectando ao servidor." width="700" height="auto" />
<img src="IMAGES-SAMBA/samba_connecting.png" alt="Imagens" title="Figura 2.8:  conectando ao servidor." width="700" height="auto" />

* Vídeo realizando a conexão e acessando o arquivo de texto criado anteriormente

<img src="IMAGES-SAMBA/samba_connection.gif" alt="Imagens" title="Figura 2.9:  conectando ao servidor." width="700" height="auto" />
