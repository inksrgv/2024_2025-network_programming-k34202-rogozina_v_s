University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Rogozina Veronika Sergeevna

Lab: Lab1

Date of create: 26.09.2024

Date of finished: 27.09.2024

# Лабораторная работа №1 "Установка CHR и Ansible, настройка VPN"

## Описание работы

Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов.

## Цель работы

Целью данной работы является развертывание виртуальной машины на базе платформы Yandex.Cloud с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox.

## Ход выполнения работы

1. Была развернута виртуальная машина с помощью сервиса Yandex.Cloud на операционной системе Ubuntu.
   
2. Подключаемся к ВМ по ssh, проверяем версию Ubuntu. 

![photo_2024-09-26_14-41-02](https://github.com/user-attachments/assets/42cf4393-7729-466b-a4a6-47c2df07803e)

3. Устанавливаем python3 и ansible.
   
       sudo apt install python3-pip
       ls -la /usr/bin/python3.6
       sudo pip3 install ansible
       ansible --version
   
4. Проверяем, что все установилось
   
![photo_2024-09-26_14-46-55](https://github.com/user-attachments/assets/b1ece714-ba27-4fa9-af79-8b26fdc65d45)

5. Устанавливаем VirtualBox, а на него CHR. После успешной установки переходим в графический интерфейс по нашему IP (это не получится сделать, если находиться в корпоративной сети :) )
   
![photo_2024-09-26_14-21-09](https://github.com/user-attachments/assets/5a0d200e-dc7d-4482-8746-e8642bdc3b66)

6. Далее создаём Wireguard сервер для организации VPN туннеля между сервером автоматизации где был установлена система контроля конфигураций Ansible и локальным CHR. 

   Установка Wireguard:
   
   ![photo_2024-09-26_14-50-59](https://github.com/user-attachments/assets/18c02d86-6d1c-419f-86c5-08dfcd8c4307)

   Создание приватного и публичного ключа для дальнейшего подключения к серверу:
   
   ![photo_2024-09-26_15-08-42](https://github.com/user-attachments/assets/7b21876c-fff0-41b4-8179-4b4c69220bbc)

   Проверяем, что с правами доступа к ключам проблем нет:
   
   ![photo_2024-09-26_15-09-40](https://github.com/user-attachments/assets/4dfb25e3-ec8a-4770-b1d7-cb728bd3d841)

   Для настройки сервера создаем конфигурационный файл wg0.conf в директории etc/wireguard/. В него прописываем конфигурацию нашего VPN-туннеля, при этом приватный ключ берём из позапрошлого шага, публичный ключ на данном этапе можем прописать любой, потом его нужно будет поменять на сгенерированный сервером ключ:
   
   ![photo_2024-09-26_15-15-33](https://github.com/user-attachments/assets/e530c6fd-d7dd-49d3-926d-5b5b7a16ffc1)

   Запускаем сервер:
   
   ![photo_2024-09-26_15-34-10](https://github.com/user-attachments/assets/e544663f-d545-4cb0-80cb-986023ee7df0)

7. Поднимаем VPN-туннель между VPN сервером на Ubuntu и VPN клиентом на RouterOS (CHR).

  Переходим в графический интерфейс, и идем в WebFig -> Wireguard и добавляем новый: 
  
  ![Снимок экрана 2024-09-27 141742](https://github.com/user-attachments/assets/43706e69-b941-4a8a-a87d-71ce79fdc566)

  Отсюда сразу забираем публичный ключ и прописываем в файле wg0.conf: 

  ![photo_2024-09-26_15-41-59](https://github.com/user-attachments/assets/ea09254a-4ed2-4a66-8338-f5460dcbf730)

  Возвращаемся в графический интерфейс и переходим в Wireguard -> Peers, создаем новую (тут Public Key - это публичный ключ нашего сервера, а Endpoint - Ip адрес сервера, порт ипользуем дефолтный):
  
  ![Снимок экрана 2024-09-27 142128](https://github.com/user-attachments/assets/64b6b2f8-fea2-4033-986c-a02ed89a482f)

  Добавляем IP в IP->Address:

  ![Снимок экрана 2024-09-27 142348](https://github.com/user-attachments/assets/16a3d29e-72df-4172-b24c-cb102df115a2)

  Затем идем в IP -> Firewall -> NAT и создаем там:
  
  ![Снимок экрана 2024-09-27 142945](https://github.com/user-attachments/assets/0e30a977-34e4-4bfc-8757-a44f7b6524f2)


## Результат выполнения работы 

**Важно!** Сначала отправляем запрос с клиента на сервер а не наоборот)

Доступность сервера: 

![photo_2024-09-26_17-00-51](https://github.com/user-attachments/assets/095367a3-95be-4912-aeaa-0f3e6126fece)

Доступность клиента: 

![photo_2024-09-26_17-01-07](https://github.com/user-attachments/assets/4b7efe18-2e48-4f43-a282-f771d7b62c89)


## Вывод 

В ходе выполнения данной лабораторной работы была развернута виртуальная машина на базе платформы Yandex.Cloud с установленной системой контроля конфигураций Ansible и установлен CHR в VirtualBox.

## Полезные ссылки

1. [Установка CHR в VB](https://help.mikrotik.com/docs/display/ROS/CHR%3A+installing+on+VirtualBox)
2. [Установка и настройка Wireguard](https://kubuntu.ru/node/17452)

