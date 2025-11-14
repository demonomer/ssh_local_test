#                    Домашнее задание по теме 
#     «Работа с серверами и протокол удалённого управления SSH»

Цель: получить базовые навыки настройкии и использования SSH

Конфигурация ОС: локальная машина с Windows 10 
и удаленный сервер на ее штатном WSL. 


## Кейс 1. Настройка и подключение

обновим WSL-дистрибутив: 
sudo apt update && sudo apt upgrade 
установим OpenSSH-сервер:  
sudo apt install openssh-server  
 
cоздаем и корректируем файл конфигурации sshd_config:  
sudo vim /etc/ssh/sshd_config 
изменим в нем порт с 22 на другой, чтобы не было конфликта: 
	Port 2222 
добавим возможность заходить извне (раскомментируем):  
	ListenAddress 0.0.0.0  
Пока планируем входить по паролю пользователя WSL (раскомментируем): 
	PasswordAuthentication yes  
security-заглушка: 
	PermitRootLogin no 
 
запустим сервер в WSL и проверим состояние: 
sudo service ssh start 
sudo service ssh status 
ssh.service - OpenBSD Secure Shell server 
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled) 
     Active: active (running) since Fri 2025-11-14 20:26:33 MSK; 3min 48s ago 
TriggeredBy:  ssh.socket 
<skipped>
Nov 14 20:26:33 Workstat systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
### итог: cервер установлен и запущен, статус "active (running)"


## Кейс 2. Подключение к удаленному серверу

в ВМ ищем свой IP-адрес (WSL-дистрибутива) командой ip для eth0 или ensXX:
$ ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:4d:47:58 brd ff:ff:ff:ff:ff:ff
    inet 192.168.95.133/20 brd 192.168.95.255 scope global eth0
<skipped>

подключаемся из командной строки или powerShell'а,
запущенные из-под администратора, и попадаем в терминал Ubuntu:
> ssh -p 2222 demonomer@192.168.95.133
The authenticity of host '192.168.95.133 (192.168.95.133)' can't be established.
ED25519 key fingerprint is SHA256:uLYypIhcYU1SblqaynZwN9DmTZ6bP6I5jTGoDn7CfI0.
This host key is known by the following other names/addresses:
    C:\Users\Professional/.ssh/known_hosts:3: localhost
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.95.133' (ED25519) to the list of known hosts.
demonomer@192.168.95.133's password:
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)
<skipped>
Last login: Fri Nov 14 20:42:37 2025 from ::1
(base) demonomer@Workstat:~$ exit
logout
Connection to 192.168.95.133 closed.
### итог: подключились по ssh с использованием логина и IP-адреса


## Кейс 3. Копирование файлов с локальной машины на удаленный сервер:

проверяем наличие файла на ВМ - он отсутствует:
ls -lisa x*
ls: cannot access 'x*': No such file or directory
date 
Fri Nov 14 21:33:29 MSK 2025 
копируем файл с локальной машины на удаленный сервер: 
c:\vboxfolder>scp x_ssh_scp_test.txt demonomer@localhost:/home/demonomer 
demonomer@localhost's password: 
x_ssh_scp_test.txt                        
файл скопирован, проверим: 
ls -lisa x* 
7430 4 -rw-rw-r-- 1 demonomer demonomer 25 Nov 14 21:34 x_ssh_scp_test.txt 

запомним дату исходного файла и удалим его с локальной машины: 
PS C:\vboxfolder> Get-ChildItem x* | Format-Table Name,LastWriteTime 
Name               LastWriteTime 
x_ssh_scp_test.txt 14.11.2025 21:31:01 
PS C:\vboxfolder> rm .\x_ssh_scp_test.txt 
проверка - повторная попытка удаления не работает, т.е. файл удален: 
PS C:\vboxfolder> rm .\x_ssh_scp_test.txt 
rm : Cannot find path 'C:\vboxfolder\x_ssh_scp_test.txt' because it does not exist. 
скопируем файл с удаленного сервера в текущую директорию локальной машины: 
PS C:\vboxfolder> scp demonomer@localhost:/home/demonomer/x_ssh_scp_test.txt .
demonomer@localhost's password:
x_ssh_scp_test.txt                                     
проверим наличие файла и дату изменения - она отличается от исходной:
PS C:\vboxfolder> Get-ChildItem x* | Format-Table Name,LastWriteTime
Name               LastWriteTime
x_ssh_scp_test.txt 14.11.2025 21:51:15

### итог: успешно скопирован файл в обе стороны


## Кейс 4. Работа с ключами

генерируем пару ключей на локальной машине. используем алгоритм ED25519 с флагом -f, 
чтобы имя ключа отличалось от стандартного id_ed25519, т.к. уже есть ключ для GitHub:
PS C:\vboxfolder> ssh-keygen -t ed25519 -f C:\Users\Professional\.ssh\id_wsl_ed25519 -C "WSL Access Key"
Generating public/private ed25519 key pair.
Your identification has been saved in C:\Users\Professional\.ssh\id_wsl_ed25519
Your public key has been saved in C:\Users\Professional\.ssh\id_wsl_ed25519.pub
The key fingerprint is:
SHA256:KVb60yus7OnZ8y1K5ekSJPlxGtJcZO5BB1euiOOu8Pg WSL Access Key
The key's randomart image is: <skipped>
ключи сгенерированы:
PS C:\vboxfolder> dir C:\Users\Professional\.ssh\id_wsl*
    Directory: C:\Users\Professional\.ssh
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        14.11.2025     22:10            411 id_wsl_ed25519
-a----        14.11.2025     22:10             97 id_wsl_ed25519.pub

передаем .pub-ключ на удаленный сервер для беспарольного входа:
PS C:\vboxfolder> scp c:\Users\Professional\.ssh\id_wsl_ed25519.pub demonomer@localhost:~/.ssh
публичный ключ скопирован: 
ls ~/.ssh 
id_ed25519  id_ed25519.pub  id_wsl_ed25519.pub  known_hosts
добавим публичный ключ в authorized_keys (вставим строку в файл) и
установим права для authorized_keys:
chmod 600 ~/.ssh/authorized_keys

добавим приватный ключ в ssh-agent:
PS C:\vboxfolder> ssh-add C:\Users\Professional\.ssh\id_wsl_ed25519
Identity added: C:\Users\Professional\.ssh\id_wsl_ed25519 (WSL Access Key)

теперь ssh при подключении к удаленному серверу не запрашивает пароль:
PS C:\Windows\system32> ssh -p 2222 demonomer@localhost
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)

остается в целях безопасности отключить аутентификацию по паролю,
для этого исправим в /etc/ssh/sshd_config yes на no в строке PasswordAuthentication 
и перезапустим ssh:
sudo service ssh restart

### итог: созданы ключи, публичный ключ добавлен на сервер, входим без пароля
