# Лабораторная работа №1  
Студент: Алина Трифонова  
Группа: НПИ-02-23  
Дата: 2025-09-20  

---

## Цель работы
Поднять сервер и клиент через Vagrant, настроить сеть между ними и создать пользователей с правами администратора.

---

## Ход работы

### 1. Подготовка проекта
Созданы рабочие папки на Windows:

- C:\work\ali_trifonova\vagrant — сервер  
- C:\work\ali_trifonova\vagrant_client — клиент  
- C:\work\ali_trifonova\packer — ISO Rocky Linux (не использовался)

---

### 2. Подключение box-файла
Скачан box-файл bionic-server-cloudimg-amd64-vagrant.box и добавлен в Vagrant:

```powershell
cd C:\work\ali_trifonova\vagrant
vagrant box add bionic64 .\boxes\bionic-server-cloudimg-amd64-vagrant.box
vagrant init bionic64
```
---

### 3. Настройка Vagrantfile

Сервер:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bionic64"
  config.vm.hostname = "server"
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "VagrantServer"
    vb.memory = "1024"
    vb.cpus = 1
  end
end
```

Клиент:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bionic64"
  config.vm.hostname = "client"
  config.vm.network "private_network", ip: "192.168.56.11"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "VagrantClient"
    vb.memory = "1024"
    vb.cpus = 1
  end
end
```

### 4. Запуск виртуальных машин 

```powershell
cd C:\work\ali_trifonova\vagrant
vagrant up   # запуск сервера

cd C:\work\ali_trifonova\vagrant_client
vagrant up   # запуск клиента
```

### 5. Создание пользователей с правами администратора

На сервере:

```bash
sudo adduser trifonovaserver
sudo usermod -aG sudo trifonovaserver
```

На клиенте:

```bash
sudo adduser trifonovaclient
sudo usermod -aG sudo trifonovaclient
```

### 6. Проверка сети и имени хоста

На сервере: 

```bash
ping -c 4 192.168.56.11
hostname
whoami
sudo -l
```

На клиенте:

```bash
ping -c 4 192.168.56.10
hostname
whoami
sudo -l
```

- Пинги прошли успешно, сеть работает.
- Имена хостов совпадают с настройками в Vagrantfile.
- Пользователи имеют права администратора.


### 7. Подготовка файлов для сдачи

Для переноса на другой компьютер подготовлены следующие файлы:

- C:\work\ali_trifonova\vagrant\Vagrantfile — сервер
- C:\work\ali_trifonova\vagrant_client\Vagrantfile — клиент
- C:\work\ali_trifonova\vagrant\boxes\bionic64.box — box-файл

Скопированы в папку VagrantProject.


### Вывод

Все задачи лабораторной работы выполнены:
- Сервер и клиент подняты через Vagrant и работают корректно.
- Созданы пользователи с правами администратора.
- Настроена сеть между сервером и клиентом, проверена связь.
- Подготовлены все необходимые файлы для переноса и повторного развертывания VM.
- Возникали трудности во время выполнения работы, поэтому пришлось выбирать альтернативные варианты решения
