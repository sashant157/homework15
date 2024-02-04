# homework15 Пользователи и группы. Авторизация и аутентификация
Vagrantfile - файл разворачивает ВМ с настроенной PAM аутентификацией. Адрес для ssh подключения 192.168.57.11
По условиям задания пользователям не входящим в группу admin запрещен вход в субботу и воскресенье.
В ВМ созданы пользователи otus и otusadm, пароль для обоих: otus.
Пользователь otusadm входит в группу admin:
[root@pam ~]# cat /etc/group | grep admin
admin:x:1003:otusadm,root,vagrant
[root@pam ~]# 
Скрипт login.sh делает проверку дня недели, если день суббота или воскресенье (если нет, то на выходе 0), то проверяет входит ли пользователь в группу admin (если выполняется, то на выходе 0).
Добавляем в /etc/pam.d/sshd директиву для модуля pam_exec (строка 15)
[root@pam ~]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
auth       required     pam_exec.so /usr/local/bin/login.sh
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin
[root@pam ~]# 
Смотрим текущую дату:
[root@pam ~]# date
Sun Feb  4 10:07:48 UTC 2024
Воскресенье, значит войти получится только учеткой otusadm (она входит в группу admin)
Проверка:
root@otus-ubuntu:/hw/hw15/test# ssh otusadm@192.168.57.11
otusadm@192.168.57.11's password: 
Last failed login: Sun Feb  4 10:10:07 UTC 2024 from 192.168.57.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Sun Feb  4 09:45:49 2024
[otusadm@pam ~]$ exit
logout
Connection to 192.168.57.11 closed.
root@otus-ubuntu:/hw/hw15/test# 
Подключиться удалось.
Проверяем подключение учеткой otus (не входит в группу admin):
root@otus-ubuntu:/hw/hw15/test# ssh otus@192.168.57.11
otus@192.168.57.11's password: 
Permission denied, please try again.
otus@192.168.57.11's password: 

root@otus-ubuntu:/hw/hw15/test# 
Подключиться не удается.
Посмотрим лог (в строке 63 видим ошибку, на выходе скрипта 1):
[root@pam ~]# tail -10 /var/log/secure 
Feb  4 10:10:15 pam sshd[5177]: Connection closed by authenticating user otusadm 192.168.57.1 port 56008 [preauth]
Feb  4 10:10:32 pam sshd[5193]: Accepted password for otusadm from 192.168.57.1 port 34070 ssh2
Feb  4 10:10:32 pam systemd[5203]: pam_unix(systemd-user:session): session opened for user otusadm by (uid=0)
Feb  4 10:10:33 pam sshd[5193]: pam_unix(sshd:session): session opened for user otusadm by (uid=0)
Feb  4 10:10:55 pam sshd[5214]: Received disconnect from 192.168.57.1 port 34070:11: disconnected by user
Feb  4 10:10:55 pam sshd[5214]: Disconnected from user otusadm 192.168.57.1 port 34070
Feb  4 10:10:55 pam sshd[5193]: pam_unix(sshd:session): session closed for user otusadm
Feb  4 10:12:46 pam sshd[5253]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Feb  4 10:12:48 pam sshd[5253]: Failed password for otus from 192.168.57.1 port 40080 ssh2
Feb  4 10:12:53 pam sshd[5253]: Connection closed by authenticating user otus 192.168.57.1 port 40080 [preauth]
[root@pam ~]# 
