University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Rogozina Veronika Sergeevna

Lab: Lab2

Date of create: 17.10.2024

Date of finished: 5.11.2024

# Лабораторная работа №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

## Описание работы

В данной лабораторной работе вы на практике ознакомитесь с системой управления конфигурацией Ansible, использующаяся для автоматизации настройки и развертывания программного обеспечения.

## Цель работы

Целью данной работы является настройка нескольких сетевых устройств и сбор информации о них с помощью Ansible.

## Ход выполнения работы

Понимаем, что случайно снесли образ и делаем заново 1 лабу :)

### Установка 2-х CHR и настройка VPN 

Для начала нужно установить 2 образа диска 

![Снимок экрана 2024-10-24 173120](https://github.com/user-attachments/assets/b0c8e351-6154-41b9-b74d-66023f285c3f)

Далее аналогично ЛР1 создаем 2 виртуальных роутера в VirtualBox

![Снимок экрана 2024-10-24 173320](https://github.com/user-attachments/assets/04cde6ab-ec08-45a7-9c47-7c11503d0c1e)

Запускаем их и переходим в графический интерфейс по IP для дальнейшей работы.

![Снимок экрана 2024-11-03 141655](https://github.com/user-attachments/assets/e9c0b109-867f-4e1a-bd45-93ad690c128b)

Далее создаем новую пару приватного и публичного ключа для подключения к клиентам. 

![Снимок экрана 2024-10-24 184601](https://github.com/user-attachments/assets/c281e35e-1aec-4711-aa1f-4eb644282747)

Для настройки сервера создаем конфигурационный файл wg0.conf в директории etc/wireguard/. В него прописываем конфигурацию нашего VPN-туннеля.

![Снимок экрана 2024-10-24 205649](https://github.com/user-attachments/assets/14bbd930-5132-4115-bbb1-ab02e12d3b78)

Дальше настраиваем клиента. Создаем новый интерфейс:

![Снимок экрана 2024-10-24 205854](https://github.com/user-attachments/assets/0b819c83-7bed-4174-9ef9-5568dc66d32b)

Создаем новый peer:

![Снимок экрана 2024-10-24 205841](https://github.com/user-attachments/assets/608bbff6-d89e-41a1-b3a4-70b0cdeac734)

После этого проверяем доступность устройств. 

* CHR1 -> CHR2, CHR1 -> Server
  
![Снимок экрана 2024-10-24 211704](https://github.com/user-attachments/assets/743f939f-3d04-4b51-bd36-15ff9416ab56)

* CHR2 -> CHR1, CHR2 -> Server
  
![Снимок экрана 2024-10-24 211716](https://github.com/user-attachments/assets/6ce73d58-acc5-444c-9036-093685231262)

Все доступно, значит можно переходить к следующему шагу.

### Настройка Ansible

Используя Ansible настроим на 2-х CHR:
* логин/пароль;
* NTP Client;
* OSPF с указанием Router ID;
* сбор данных по OSPF топологии и полный конфиг устройства.

<details>
  <summary> Перед тем как настраивать Ansible... </summary>
  
  
По умолчанию Ansible использует SSH для подключения к удаленным хостам, однако очень часто при подключении возникает проблема с учетными записями и пароями. Чтобы этого избежать, импортируем публичный ключ сервера и добавим его в конфиг пользователя в качестве переменной ssh-keys.

**НО!** Пока что импортировать мне нечего. Создаем ключ:

![Снимок экрана 2024-10-24 213149](https://github.com/user-attachments/assets/160d2265-71b5-4b96-bf84-bfc987fa3d4c)

Далее импортируем публичный ключ на клиента (у меня не получилось сделать с помощью scp, поэтому импорт осуществлен с помощью ftp)

![Снимок экрана 2024-11-02 225316](https://github.com/user-attachments/assets/acca1443-0921-4bef-9a19-886d944bc3c6)

![Снимок экрана 2024-11-02 225436](https://github.com/user-attachments/assets/10e16f84-2670-4660-86d5-282b3fb60025)

Проверим, что теперь пароль необязателен:

![Снимок экрана 2024-11-02 224143](https://github.com/user-attachments/assets/5b0e50a5-abd6-4e3c-a223-f50cd6d1f182)

Ну и выполним то же самое для второго CHR :)

</details>

Перейдем к настройке Ansible. Для начала нужно создать несколько конфиг файлов, чтобы все работало корректно. 

*  inventory/hosts
  
        [routers]
        chr1 ansible_host=10.0.0.2
        chr2 ansible_host=10.0.0.3

        [routers:vars]
        ansible_connection=ansible.netcommon.network_cli
        ansible_network_os=community.routeros.routeros
        ansible_user=admin
        ansible_ssh_private_key_file=/home/nika/.ssh/new-key
   
*  ansible.cfg
  
В целом конфигурация довольно дефолтная, однако у меня были проблемы с интерпритатором, поэтому он указан в конфиге, и с доступом по ssh, поэтому путь к приватному ключу также добавлен в файл. Итоговое содержание файла:

       [defaults]
        inventory=./inventory/hosts
        ansible_remote_tmp = /tmp
        ansible_python_interpreter = /home/nika/myenv/bin/python3.12
        ansible_ssh_private_key_file: /home/nika/.ssh/new-key

Далее пропишем playbook:

    - name: Config chrs with login/password, ntp-client and ospf
      hosts: routers
      gather_facts: no
      vars:
          router_ospf_ip: "{{ '10.255.255.1/32' if ansible_host == '10.0.0.2' else '10.255.255.2/32' }}"
      tasks:
        - name: Config user login and password
          community.routeros.command:
            commands:
              - /user add name=user group=read password=1234
          register: user_config

        - name: Config ntp-client
          community.routeros.command:
            commands:
              - /system ntp client set enabled=yes servers=0.ru.pool.ntp.org
          register: ntp_client_config

        - name: Config ospf
          community.routeros.command:
            commands:
              - /routing ospf instance add name=default
              - /interface bridge add name=loopback
              - /ip address add address={{ router_ospf_ip }} interface=loopback
              - /routing ospf instance set 0 router-id={{ router_ospf_ip }}
              - /routing ospf area add instance=default name=backbone
              - /routing ospf interface-template add area=backbone interfaces=ether1 type=ptp
          register: ospf_config

        - name: Get config
          community.routeros.command:
            commands:
              - /export
          register: router_config

        - name: Show config
          debug:
            var: router_config.stdout_lines

И выполним его 

![Снимок экрана 2024-11-02 233755](https://github.com/user-attachments/assets/ade9afcf-1e19-4afc-be39-dec690d0bf37)

Полный вывод плейбука:

    (myenv) nika@vm-net-prog:~$ ansible-playbook config-chr.yml

    PLAY [Config chrs with login/password, ntp-client and ospf] ************************************************************************************************

    TASK [Config user login and password] **********************************************************************************************************************
    changed: [chr2]
    changed: [chr1]

    TASK [Config ntp-client] ***********************************************************************************************************************************
    changed: [chr2]
    changed: [chr1]

    TASK [Config ospf] *****************************************************************************************************************************************
    changed: [chr2]
    changed: [chr1]

    TASK [Get config] ******************************************************************************************************************************************
    changed: [chr2]
    changed: [chr1]

    TASK [Show config] *****************************************************************************************************************************************
    ok: [chr1] => {
    "router_config.stdout_lines": [
        [
            "# 2024-11-02 18:40:18 by RouterOS 7.16.1",
            "# software id = ",
            "#",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=51820 mtu=1420 name=wg1",
            "/routing ospf instance",
            "add disabled=no name=default",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=89.169.157.26 endpoint-port=\\",
            "    51820 interface=wg1 name=peer2 persistent-keepalive=25s public-key=\\",
            "    \"3en/b4SwZGBafftZg3OfCktHiHSn7MVXrvNWsTH+Oj0=\"",
            "/ip address",
            "add address=10.0.0.2/24 interface=wg1 network=10.0.0.0",
            "add address=10.255.255.1 interface=loopback network=10.255.255.1",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip firewall nat",
            "add action=masquerade chain=srcnat",
            "/ip ssh",
            "set always-allow-password-login=yes forwarding-enabled=both host-key-type=\\",
            "    ed25519",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=0.ru.pool.ntp.org"
        ]
    ]
    }
    ok: [chr2] => {
    "router_config.stdout_lines": [
        [
            "# 2024-11-02 20:35:17 by RouterOS 7.15.3",
            "# software id = ",
            "#",
            "/disk",
            "set slot1 media-interface=none media-sharing=no slot=slot1",
            "set slot2 media-interface=none media-sharing=no slot=slot2",
            "set slot3 media-interface=none media-sharing=no slot=slot3",
            "set slot4 media-interface=none media-sharing=no slot=slot4",
            "set slot5 media-interface=none media-sharing=no slot=slot5",
            "set slot6 media-interface=none media-sharing=no slot=slot6",
            "set slot7 media-interface=none media-sharing=no slot=slot7",
            "set slot8 media-interface=none media-sharing=no slot=slot8",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=51820 mtu=1420 name=wg2",
            "/routing ospf instance",
            "add disabled=no name=default",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=89.169.157.26 endpoint-port=\\",
            "    51820 interface=wg2 name=peer3 persistent-keepalive=25s public-key=\\",

    "    \"3en/b4SwZGBafftZg3OfCktHiHSn7MVXrvNWsTH+Oj0=\"",
            "/ip address",
            "add address=10.0.0.3/24 interface=wg2 network=10.0.0.0",
            "add address=10.255.255.2 interface=loopback network=10.255.255.2",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip firewall nat",
            "add action=masquerade chain=srcnat",
            "/ip ssh",
            "set always-allow-password-login=yes forwarding-enabled=both host-key-type=\\",
            "    ed25519",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=0.ru.pool.ntp.org"
        ]
    ]
    }

    PLAY RECAP *************************************************************************************************************************************************
    chr1                       : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    chr2                       : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Посмотрим, что полученные конфигурации корректны.
* CHR1
  
![Снимок экрана 2024-11-02 234628](https://github.com/user-attachments/assets/eb070e32-6ca4-4dd3-942c-bddb4e05a610)

* CHR2
  
![Снимок экрана 2024-11-02 234501](https://github.com/user-attachments/assets/db249e75-2b72-4662-a147-9607d76606f1)

Все настроилось корректно, ура)


## Вывод
В ходе выполнения данной лабораторной работы с помощью Ansible были настроены два сетевых устройства и собрана информациия о них.
