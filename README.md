# Автоматизация администрирования. Ansible

Подготовить стенд на Vagrant как минимум с одним сервером. На сервере используя Ansible необходимо развернуть nginx со следующими условиями:
- использовать модуль yum/apt
- конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть использован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible


# Установка Ansible
(домашнее задание делается на ubuntu)

Ansible требует для своей работы python. Проверить версию python:
```
python -V
```
т.к у меня его нет, устанавливаем:
```
sudo apt install python3
```
Далее переходим к установке Ansible [инструкция](https://docs.ansible.com/ansible/2.7/installation_guide/intro_installation.html#basics-what-will-be-installed)
```
sudo apt-get update
sudo apt-get install software-properties-common -y
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get install ansible -y
```
Убедиться, что ansible установлен корректно:
```
ansible --version
```
# Подготовка окружения


Поднимаем vm командой ```vagrant_up```, для подключения к хосту nginx нам необходимо будет передать множество параметров - это особенность Vagrant. Узнать эти параметры можно с помощью ```vagrant ssh-config```
В ответ получаем:

```
evgeniy@home:~/hw11$ vagrant ssh-config
Host nginx1
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/evgeniy/hw11/.vagrant/machines/nginx1/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

Host nginx2
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/evgeniy/hw11/.vagrant/machines/nginx2/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

```


Создадим отедльную директорию под inventory-файл, и сам файл hosts.
```
mkdir ansible
cd ansible
vi hosts
```
Содержимое файла hosts
```
[web]
nginx2 ansible_ssh_host=127.0.0.1 ansible_port=2200 ansible_private_key_file=/home/evgeniy/hw11/.vagrant/machines/nginx2/virtualbox/private_key

nginx1 ansible_host=127.0.0.1 ansible_port=2222 ansible_private_key_file=/home/evgeniy/hw11/.vagrant/machines/nginx1/virtualbox/private_key
```
Отредактируем файл ```ansible.cfg```

inventory - путь к моему hosts

remote_user = vagrant - эту строку можно не указывать, но тогда в hosts для каждой vm должен быть указан ```ansible_user=vagrant```

host_key_checking = False - отменить проверку пароля
```
[defaults]
inventory = ./hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False

```
# Ansible

После этого убедимся, что Ansible может управлять нашим хостом. Сделать это
можно с помощью команды:
```
ansible all -m ping
```
```
evgeniy@home:~/hw11/ansible$ ansible all -m ping
nginx2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
nginx1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

Установим пакет epel-release на хосты
```
ansible -i /home/evgeniy/hw11/ansible/hosts all -m yum -a "name=epel-release state=present" -b
```
```
evgeniy@home:~/hw11/ansible$ ansible -i /home/evgeniy/hw11/ansible/hosts all -m yum -a "name=epel-release state=present" -b
nginx1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "epel-release"
        ]
    },
    "msg": "warning: /var/cache/yum/x86_64/7/extras/packages/epel-release-7-11.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY\nImporting GPG key 0xF4A80EB5:\n Userid     : \"CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>\"\n Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5\n Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)\n From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7\n",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nDetermining fastest mirrors\n * base: mirrors.datahouse.ru\n * extras: mirrors.datahouse.ru\n * updates: mirror.sale-dedic.com\nResolving Dependencies\n--> Running transaction check\n---> Package epel-release.noarch 0:7-11 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package                Arch             Version         Repository        Size\n================================================================================\nInstalling:\n epel-release           noarch           7-11            extras            15 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 15 k\nInstalled size: 24 k\nDownloading packages:\nPublic key for epel-release-7-11.noarch.rpm is not installed\nRetrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : epel-release-7-11.noarch                                     1/1 \n  Verifying  : epel-release-7-11.noarch                                     1/1 \n\nInstalled:\n  epel-release.noarch 0:7-11                                                    \n\nComplete!\n"
    ]
}
nginx2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "epel-release"
        ]
    },
    "msg": "warning: /var/cache/yum/x86_64/7/extras/packages/epel-release-7-11.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY\nImporting GPG key 0xF4A80EB5:\n Userid     : \"CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>\"\n Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5\n Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)\n From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7\n",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nDetermining fastest mirrors\n * base: mirrors.datahouse.ru\n * extras: mirrors.datahouse.ru\n * updates: mirror.surf\nResolving Dependencies\n--> Running transaction check\n---> Package epel-release.noarch 0:7-11 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package                Arch             Version         Repository        Size\n================================================================================\nInstalling:\n epel-release           noarch           7-11            extras            15 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 15 k\nInstalled size: 24 k\nDownloading packages:\nPublic key for epel-release-7-11.noarch.rpm is not installed\nRetrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : epel-release-7-11.noarch                                     1/1 \n  Verifying  : epel-release-7-11.noarch                                     1/1 \n\nInstalled:\n  epel-release.noarch 0:7-11                                                    \n\nComplete!\n"
    ]
}

```
Удалить пакет epel-release с хостов

```
ansible -i /home/evgeniy/hw11/ansible/hosts all -m yum -a "name=epel-release state=absent" -b
```
```
nginx2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "removed": [
            "epel-release"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nResolving Dependencies\n--> Running transaction check\n---> Package epel-release.noarch 0:7-11 will be erased\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package                Arch             Version        Repository         Size\n================================================================================\nRemoving:\n epel-release           noarch           7-11           @extras            24 k\n\nTransaction Summary\n================================================================================\nRemove  1 Package\n\nInstalled size: 24 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Erasing    : epel-release-7-11.noarch                                     1/1 \n  Verifying  : epel-release-7-11.noarch                                     1/1 \n\nRemoved:\n  epel-release.noarch 0:7-11                                                    \n\nComplete!\n"
    ]
}
nginx1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "removed": [
            "epel-release"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nResolving Dependencies\n--> Running transaction check\n---> Package epel-release.noarch 0:7-11 will be erased\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package                Arch             Version        Repository         Size\n================================================================================\nRemoving:\n epel-release           noarch           7-11           @extras            24 k\n\nTransaction Summary\n================================================================================\nRemove  1 Package\n\nInstalled size: 24 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Erasing    : epel-release-7-11.noarch                                     1/1 \n  Verifying  : epel-release-7-11.noarch                                     1/1 \n\nRemoved:\n  epel-release.noarch 0:7-11                                                    \n\nComplete!\n"
    ]
}

```

# Homework

Создадим плейбук roles_nginx.yml 
```
---
- name: NGINX | Install and configure NGINX
  hosts: all
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: NGINX | Install EPEL Repo package from standart repo
      yum:
        name: epel-release
        state: present
      tags:
        - epel-package
        - packages

    - name: NGINX | Install NGINX package from EPEL Repo
      yum:
        name: nginx
        state: latest
      notify:
        - restart nginx
      tags:
        - nginx-package
        - packages

    - name: NGINX | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
      tags:
        - nginx-configuration

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes
    
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```
Запускаем playbook:
```
evgeniy@home:~/hw11/ansible$ ansible-playbook roles_nginx.yml 

PLAY [NGINX | Install and configure NGINX] **********************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [nginx1]
ok: [nginx2]

TASK [roles_nginx : NGINX | Install EPEL Repo package from standart repo] ***************************************************************************
changed: [nginx2]
changed: [nginx1]

TASK [roles_nginx : NGINX | Install NGINX package from EPEL Repo] ***********************************************************************************
changed: [nginx1]
changed: [nginx2]

TASK [roles_nginx : NGINX | Create NGINX config file from template] *********************************************************************************
changed: [nginx2]
changed: [nginx1]

RUNNING HANDLER [roles_nginx : restart nginx] *******************************************************************************************************
changed: [nginx1]
changed: [nginx2]

RUNNING HANDLER [roles_nginx : reload nginx] ********************************************************************************************************
changed: [nginx2]
changed: [nginx1]

PLAY RECAP ******************************************************************************************************************************************
nginx1                     : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
nginx2                     : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```


проверяем, что работает

http://192.168.56.21:8080/
![Image alt](https://github.com/evgeniy-romanov/hw11/raw/main/1.png)
http://192.168.56.22:8080/
![Image alt](https://github.com/evgeniy-romanov/hw11/raw/main/2.png)

# Роль

Для создания роли
```
ansible-galaxy init nginx
```
Получим подобный каталог папок и файлов (лишнее потом удалим)
![Image alt](https://github.com/Edo1993/otus_10/raw/master/114.png)

Распиливаем наш playbook, перемещая всё по соответствующим директориям.
 - 1 В раздел handlers - перемещаем всё из блока handlers в playbook (самом playbook перенесёное выпиливаем - справедливо и для всего нижеследующего).
Итого, ```/handlers/main.yml``` получит следующую начинку:
```
---
# handlers file for nginx
- name: restart nginx
  systemd:
    name: nginx
    state: restarted
    enabled: yes
    
- name: reload nginx
  systemd:
    name: nginx
    state: reloaded
```
 - 2 В раздел tasks - перемещаем всё из блока tasks в playbook.
```/tasks/main.yml``` :
```
---
# tasks file for nginx
- name: NGINX | Install EPEL Repo package from standart repo
  yum:
    name: epel-release
    state: present
  tags:
    - epel-package
    - packages

- name: NGINX | Install NGINX package from EPEL Repo
  yum:
    name: nginx
    state: latest
  notify:
    - restart nginx
  tags:
    - nginx-package
    - packages

- name: NGINX | Create NGINX config file from template
  template:
    src: templates/nginx.c---
# tasks file for nginx
- name: NGINX | Install EPEL Repo package from standart repo
  yum:
    name: epel-release
    state: present
  tags:
    - epel-package
    - packages

- name: NGINX | Install NGINX package from EPEL Repo
  yum:
    name: nginx
    state: latest
  notify:
    - restart nginx
  tags:
    - nginx-package
    - packages

- name: NGINX | Create NGINX config file from template
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - reload nginx
  tags:
    - nginx-configurationonf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - reload nginx
  tags:
    - nginx-configuration
```
 - 3 В раздел ```templates``` перенесла содержимое текущего ```templates```, выпилив существующую директорию.
 - 4 Раздел ```vars```
```
---
# vars file for nginx
nginx_listen_port: 8080
```

Остальные директории были удалены. Playbook принял следующий вид :
```
---
- name: NGINX | Install and configure NGINX
  hosts: all
  become: true

  roles:
    - nginx
```

(Если честно - то после создания каталога ролей я пошла спать, и на следующий день у меня нихрена не работало. Провозившись 2 часа стало понятно, что после рестарта вм изменили порты. Вылечилось простым изменением портов в inventory-файле. Для проверки - позвать ```vagrant ssh-config``` и проверить, что вывод совпадает с содержимым inventory)

Каталог с ролями должен лежать в той же директории, что и playbook - иначе у меня не стартануло.

Запускаем playbook, чтобы проверить, что всё работает
![Image alt](https://github.com/Edo1993/otus_10/raw/master/115.png)

Итого - всё это хозяйство с ролью лежит [здесь](https://github.com/Edo1993/otus_10/tree/master/2_part)
