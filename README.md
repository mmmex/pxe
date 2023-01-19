## PXE

```
Настройка PXE сервера для автоматической установки

Цель:
Отрабатываем навыки установки и настройки DHCP, TFTP, PXE загрузчика и автоматической загрузки

Описание/Пошаговая инструкция выполнения домашнего задания:
1. Следуя шагам из документа https://docs.centos.org/en-US/8-docs/advanced-install/assembly_preparing-for-a-network-install установить и настроить загрузку по сети для дистрибутива CentOS8.
   - В качестве шаблона воспользуйтесь репозиторием https://github.com/nixuser/virtlab/tree/main/centos_pxe.
2. Поменять установку из репозитория NFS на установку из репозитория HTTP.
3. Настройить автоматическую установку для созданного kickstart файла (*) Файл загружается по HTTP.
4. Автоматизировать процесс установки Cobbler cледуя шагам из документа https://cobbler.readthedocs.io/en/latest/quickstart-guide.html.
```

### Запуск проекта

* Клонируем репозиторий: `git clone `

* Переходим в каталог: `cd pxe/centos_pxe`

* Запуск: `vagrant up`

Настройка ВМ `pxeserver` будет выполнена автоматически. Шаг `Download ISO image CentOS 8.4.2105` может занять очень длительное время. После выполнения скрипта ansible для `pxeserver` будет автоматически выполнен запуск ВМ `pxeclient` с опцией `gui = true`, т.е. появится окно virtualbox в котором должны увидеть меню загрузчика и по умолчанию будет выбрана автоматическая установка дистрибутива Centos8.

[Screenshot]()

После установки можно выключить ВМ и выставить в настройках загрузку с жесткого диска или проще, перезагрузить, в момент загрузки нажать клавишу F12 и выбрать загрузку с жесткого диска.

[Screenshot]()

Видим что ОС успешно установлена.

[Screenshot]()

Можем проверить настройки из [kickstart скрипта](centos_pxe/ansible/templates/ks.j2):

[Screenshot]()


### [VagrantFile](centos_pxe/Vagrantfile)

Добавил несколько параметров в раздел конфигурации ВМ `pxeclient`: размер видеопамяти (20MB) и графический контроллер (VMSVGA):

```ruby
      vb.customize [
          'modifyvm', :id,
          '--vram', '20',
          '--graphicscontroller', 'vmsvga', # https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm
          '--natdnshostresolver1', 'on',
          '--nic1', 'intnet',
          '--intnet1', 'pxenet',
          '--nic2', 'nat',
          '--boot1', 'net',
          '--boot2', 'none',
          '--boot3', 'none',
          '--boot4', 'none'
        ]
```

### [Скрипт ansible](centos_pxe/ansible/provision.yml) для настройки pxe сервера

Добавил несколько дополнительных переменных для [ks.cfg](centos_pxe/ansible/templates/ks.j2):

```yml
---
dhcp_network: 10.0.0.0
dhcp_mask: 255.255.255.0
dhcp_range_min: 10.0.0.100
dhcp_range_max: 10.0.0.120
pxe_server: 10.0.0.20
centos_ver: centos8
name_ver: "CentOS 8.4"
timeout_menu: 150
lan_ip: enp0s3
# ks default variables
ks_root_password: "$6$4jRH5ccaw2WWlb6/$iA/8UYASUQKoXmCDVjo9JdBDOywZ42JXQnRJXN1BbbUF4xvBMj9VDV2g22/vVpqEz/CMBngv23HYH0/gFLcYW."
ks_admin_username: val
ks_admin_password: "$6$M.DMs9jPflsgd8K/$antagfnemEBjDj647y4V/XKkoAjE4buB9NSR.NML/hS84gtHN771/Spbw8Z2z5EJZ.WMJSbDtF07yuaVGrcdP."
ks_admin_gecos: val
```

Для генерации своего пароля можно использовать команду:

```python
python3 -c 'import crypt,getpass; print(crypt.crypt(getpass.getpass(), crypt.mksalt(crypt.METHOD_SHA512)))'
```