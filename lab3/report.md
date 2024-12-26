University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Rogozina Veronika Sergeevna

Lab: Lab3

Date of create: 17.10.2024

Date of finished: 27.12.2024

# Лабораторная работа №3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

## Описание работы

В данной лабораторной работе вы ознакомитесь с интеграцией Ansible и Netbox и изучите методы сбора информации с помощью данной интеграции.

## Цель работы

С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

## Ход выполнения работы 

### Установка Netbox на дополнительную ВМ

Нетбокс будет развернут на виртуальной машине Ubuntu в VirtualBox. Скачиваем ВМ с образом, ставим и переходим к настройке окружения.

Устанавливаем PostgreSQL 

![Снимок экрана 2024-11-05 160219](https://github.com/user-attachments/assets/564dd345-e6d5-45c9-a984-d3c92a1dfbd5)

Подключаемся к PostgreSQL 

![Снимок экрана 2024-11-05 160723](https://github.com/user-attachments/assets/27f736c6-c375-4674-9cff-7b81a44fbeda)

Создаем базу данных для нетбокса и суперпользователя

![Снимок экрана 2024-11-05 161011](https://github.com/user-attachments/assets/5763e0af-c692-4352-9d5b-a8a93756ffa0)


Далее скачиваем Python

![Снимок экрана 2024-11-05 164717](https://github.com/user-attachments/assets/25a289ce-9ed3-4b6a-b575-af735ea7cb08)

И Redis

![Снимок экрана 2024-11-05 165539](https://github.com/user-attachments/assets/f1b7ee25-661e-4f3f-a889-9276332b385c)

Теперь наконец-то можно поставить и сам Netbox. Сделать это можно с помощью 

    sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/vX.Y.Z.tar.gz
    sudo tar -xzf vX.Y.Z.tar.gz -C /opt
    sudo ln -s /opt/netbox-X.Y.Z/ /opt/netbox
    
![Снимок экрана 2024-11-05 170114](https://github.com/user-attachments/assets/94d7aaef-ca24-4030-90c2-56637def1af4)

Создаем пользователя Netbox и даем ему права 

![Снимок экрана 2024-11-05 170915](https://github.com/user-attachments/assets/1a53fafa-9f5b-4438-9d3c-ac8d3481625d)

После этого проводим еще некоторые манипуляции

![Снимок экрана 2024-11-05 173634](https://github.com/user-attachments/assets/6db61c7d-cb82-4ba3-a672-ec6f2574ae56)

Настраиваем конфиг нетбокса. Также добавляем секретный ключ, сгенерированный в прошлом шаге в этот конфиг

![Снимок экрана 2024-11-05 173602](https://github.com/user-attachments/assets/2d796c42-dc18-4b3b-99d0-f8a4383246e0)

Затем создаем виртуальное окружение с помощью команд

    python3 -m venv /opt/netbox/venv
    source venv/bin/activate
    pip3 install -r requirements.txt

Делаем миграции

![Снимок экрана 2024-11-05 181830](https://github.com/user-attachments/assets/20a23f6a-42c5-45c7-8b95-6dba66737f80)

Создаем суперпользователя и делаем еще некоторые манипуляции

![Снимок экрана 2024-11-05 182208](https://github.com/user-attachments/assets/55b07bc3-0b45-45d0-b35b-a9696b7923ad)


После этого запускаем сервер

![Снимок экрана 2024-11-05 191612](https://github.com/user-attachments/assets/a0f6fb81-1640-407a-b507-77edfefcd1fc)


О неужели! Видим окошко Netbox

![Снимок экрана 2024-11-05 191422](https://github.com/user-attachments/assets/98c18b2a-bbe3-4897-b6f8-87691791affa)

Теперь надо кучу всего создать...

### Заполнение информации в Netbox

Создаем сайт 

![Снимок экрана 2024-11-08 124443](https://github.com/user-attachments/assets/5aad7d56-ad15-4f1c-92e9-1e329b62d8bf)

Мануфактуру

![Снимок экрана 2024-11-08 124726](https://github.com/user-attachments/assets/62523f37-6b73-4c73-8b1b-440c0433368f)

Тип устройства

![Снимок экрана 2024-11-08 124738](https://github.com/user-attachments/assets/24dd8a3c-b676-4bdc-9800-223335304e16)

Роль устройства

![Снимок экрана 2024-11-08 125238](https://github.com/user-attachments/assets/bdd1f7a6-1d52-458e-b961-d5a550fff1df)

И айпи адрес с интерфейсом

![Снимок экрана 2024-11-08 125913](https://github.com/user-attachments/assets/65c9748b-fc36-449e-afa4-fadf046c40c3)

А теперь можно и устройства создать :)))

![Снимок экрана 2024-11-08 130034](https://github.com/user-attachments/assets/d4079b73-fae5-4d4e-8b5c-1b0358012bc6)

### Сохранение данных из Netbox

На этом шаге я осознала, что не прокинула еще один VPN-туннель, поэтому аналогично прошлым работам прокидываю. В сервере добавила еще одну peer, со стороны клиента скачала wg и натсроила интерфейс


После тысячи лет мучений сеть была настроена верно и устройства стали видеть друг друга. 
Затем был создан инвентори файл со следующим содержимым: 

    plugin: netbox.netbox.nb_inventory
    api_endpoint: https://10.0.0.4:8000
    token: *сгенерированный в нетбоксе токен*
    validate_certs: False
    ansible_user: admin
    ansible_ssh_pass: '1234'

Роль была запущена, а результат ее выполнения будет помещен в файл netbox_inventory.yml. Содержимое файла получилось таким:

    all:
      children:
        ungrouped:
          hosts:
            R1:
              *тут куча информации про R1*
            R2:
              *тут куча информации про R2*

Затем в данный файл необходимо было добавить код, представленный ниже. Это было нужно для того, чтобы использовать файл в качестве инвентори файла в следующих пунктах. 

      vars:
      ansible_connection: ansible.netcommon.network_cli
      ansible_network_os: community.routeros.routeros
      ansible_user: admin
      ansible_ssh_pass: admin

### Настройка роутеров по данным из Netbox

Для данной задачи был написан плейбук: 

    - name: setup routers
      hosts: ungrouped
      tasks:
        - name: "change names"
          community.routeros.command:
            commands:
              - /system identity set name="{{ interfaces[0].device.name }}"

        - name: "change ip-addresses"
          community.routeros.command:
            commands:
              - /ip address add address="{{ interfaces[0].ip_addresses[0].address }}" interface="{{ interfaces[0].display }}"

И выполнен

    root@vm-net-prog:/home/nika/netbox# ansible-playbook -i netbox_inventory.yml playbook1.yml

    PLAY [setup routers] ***************************************************************************************************************************************

    TASK [Gathering Facts] *************************************************************************************************************************************
    ok: [R2]
    ok: [R1]

    TASK [change names] *********************************************************************************************************************************
    changed: [R2]
    changed: [R1]

    TASK [change ip-addresses] ***********************************************************************************************************************************
    changed: [R2]
    changed: [R1]

    PLAY RECAP *************************************************************************************************************************************************
    R1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    R2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Проверим, что оно сработало: 

![Снимок экрана 2024-12-26 212625](https://github.com/user-attachments/assets/0233bd19-d8d1-44c5-aff6-7fc1c73c7317)

Имя поменялось, значит всё кайф!

### Сбор и добавление серийного номера в Netbox

Плейбук: 

    - name: get serial numbers
      hosts: ungrouped
      tasks:
        - name: "get number"
          community.routeros.command:
            commands:
              - /system license print
          register: license

        - name: "get name"
          community.routeros.command:
            commands:
              - /system identity print
          register: identity

        - name: add serial number
          netbox_device:
            netbox_url: http://10.0.0.4:8000
            netbox_token: *тут мой токен*
            data:
              name: "{{ identity.stdout_lines[0][0].split()[1] }}"
              serial: "{{ license.stdout_lines[0][0].split()[1] }}"

Вывод:
    
    root@vm-net-prog:/home/nika/netbox# ansible-playbook -i netbox_inventory.yml playbook2.yml

    PLAY [get serial numbers] ***************************************************************************************************************************************

    TASK [Gathering Facts] *************************************************************************************************************************************
    ok: [R2]
    ok: [R1]

    TASK [get number] *********************************************************************************************************************************
    changed: [R2]
    changed: [R1]

    TASK [get name] ***********************************************************************************************************************************
    changed: [R2]
    changed: [R1]

    TASK [add serial number] ***********************************************************************************************************************************
    changed: [R2]
    changed: [R1]

    PLAY RECAP *************************************************************************************************************************************************
    R1                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    R2                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Проверим, что серийный номер изменился:

![Снимок экрана 2024-12-26 222521](https://github.com/user-attachments/assets/3609db4a-a41c-40ce-92cc-246b3e23f0f7)

Ура!

## Вывод

В ходе выполнения данной лабораторной работы была собрана вся возможная информация об устройствах с помощью Ansible и Netbox. 
