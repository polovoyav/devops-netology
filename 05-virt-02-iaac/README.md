# 05-virt-02-iaac.

## Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.

```
Автоматизация создания инфраструктуры для разработки, то есть упрощение и ускорение этого процесса. 
Уверенность в том, что среда разработки везде, на всех платформах,  одинаковая, исключение дрейфа 
конфигураций. Быстрая и эффективная разработка, тестирование, развёртывание.
```

- Какой из принципов IaaC является основополагающим?

```
Идемпотентность, т.е. одинаковость воспроизведения результата  при повторном выполнении.
```

## Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?

```
С ним проще начать работать, использует ssh, не требуется разворачивать pki, огромное количество модулей.
```

- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

```
Мне кажется push удобнее в плане контроля результатов расталкивания конфигураций. Плюс, удобнее управлять, 
куда и какую конфигурацию отправить.
```

## Задача 3

Установить на личный компьютер:

- VirtualBox
- Vagrant
- Ansible

```
Устанавливал на комп с Windows 10. VirtualBox v6.1.30. Vagrant v2.2.19.

PS C:\Users\andrew\VagrantVM> vagrant --version
Vagrant 2.2.19

Ansible установил с помощью Cygwin.

andrew@AndrewPC ~
$ ansible --version
ansible 2.8.4
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/andrew/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.12 (default, Nov 23 2021, 18:58:07) [GCC 11.2.0]
```

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
ВМ установилась из Vagrantfile с лекции, но Ansible не обнаружился.

The Ansible software could not be found! Please verify
that Ansible is correctly installed on your host system.

Буду делать хост на Ubuntu и ставить VirtualBox, Vagrant, Ansible туда.
```
