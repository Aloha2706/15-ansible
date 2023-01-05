# 15-Ansible 


Описание/Пошаговая инструкция выполнения домашнего задания:
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере используя Ansible необходимо развернуть nginx со следующими условиями:

необходимо использовать модуль yum/apt;
конфигурационные файлы должны быть взяты из шаблона jinja2 с перемененными;
после установки nginx должен быть в режиме enabled в systemd;
должен быть использован notify для старта nginx после установки;
сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible.


## Установка  и настройка Ansible
Проводим настройку на сервере где будет установелн Ansible
Добавляем репозитарий.

        yum install epel-release
        yum install ansible

делаем резервную копию файла с хостами чтобы было где подглядыать синтаксис: 

        mv /etc/ansible/hosts /etc/ansible/hosts.default

и правим файл с хостами 

vi /etc/ansible/hosts
[test]
192.168.88.11

 тестируем 

        [root@ansible vagrant]# ansible test -m ping -k -u vagrant
        SSH password:
        192.168.88.11 | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "changed": false,
            "ping": "pong"
        }

Тест прошел успешно, идем настраивать Playbook и все сопутствующие tasks and handlers
        
        [root@localhost ansible]# cat play.yml
        ---
        - hosts: redhat_servers
        become:
            true
        become_method:
            su
        become_user:
            root
        remote_user:
            vagrant
        #password: "{{ password.root | password_hash('sha256') }}"
        roles:
        - epel
        - nginx


        ...

 

notify — обработчик, который будет вызван в случае успешного выполнения задачи. При необходимости, можно задать несколько обработчиков;




Создаем таск для установки репозитория epel

        mkdir -p /etc/ansible/roles/epel/tasks/
        vi /etc/ansible/roles/epel/tasks/main.yml
        ---
        - name: Install EPEL Repo
        yum:
            name=epel-release
            state=present

Создаем таски для установки nginx

        mkdir -p /etc/ansible/roles/nginx/tasks
        sudo vi /etc/ansible/roles/nginx/tasks/main.yml
        ---
        - name: Install Nginx Web Server
        yum:
            name=nginx
            state=latest
        when:
            ansible_os_family == "RedHat"
        notify:
            - nginx systemd


        mkdir -p /etc/ansible/roles/nginx/handlers
        vi /etc/ansible/roles/nginx/handlers/main.yml
        ---
        - name: nginx systemd
        systemd:
            name: nginx
            enabled: yes
            state: started




ansible-playbook /etc/ansible/play.yml -kK 

Для того чтобы изменить порт на котором слушает nginx на 8080 создадим файл конфигураций для замены. 
 roles/nginx/templates/nginx.conf
 и файл с переменными. 
 roles/nginx/vars/main.yml

Добавим в /etc/ansible/roles/nginx/tasks/main.yml 

 - name: Replace nginx.conf
  template:
    src=templates/nginx.conf
    dest=/etc/nginx/nginx.conf
- name: nginx restart
  service:
    name=nginx
    state=restarted
...

запускаем , получаем вот такой вывод 
[root@localhost ansible]# ansible-playbook play.yml --ask-become-pass
BECOME password:

PLAY [redhat_servers] *****************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [192.168.88.11]

TASK [epel : Install EPEL Repo] *******************************************************************
ok: [192.168.88.11]

TASK [nginx : Install Nginx Web Server on RedHat Family] ******************************************
ok: [192.168.88.11]

TASK [Replace nginx.conf] *************************************************************************
ok: [192.168.88.11]

TASK [nginx restart] ******************************************************************************
changed: [192.168.88.11]

PLAY RECAP ****************************************************************************************
192.168.88.11              : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0



проверяем что сервер слушает на порту 8080
curl 192.168.88.11:8080

Все!!!! 





