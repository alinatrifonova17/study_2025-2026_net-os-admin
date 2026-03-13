## Цель работы

Получить навыки работы с программным средством Fail2ban для обеспечения базовой защиты от
атак типа «brute force».

## Ход работы

1. Подготовка виртуальных машин
Сервер: Ubuntu 18.04 (bionic64)
Клиент: Ubuntu 18.04 (bionic64)
Установлен Vagrant + VirtualBox

Проверена работа SSH между клиентом и сервером
# На сервере
hostname
whoami

2. Установка Fail2ban

На сервере выполнено:
sudo apt update
sudo apt install -y fail2ban
Проверка версии:
fail2ban-client --version

3. Запуск и автозапуск сервиса
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
sudo systemctl status fail2ban

4. Создание локальной конфигурации
sudo mkdir -p /etc/fail2ban/jail.d
sudo nano /etc/fail2ban/jail.d/customisation.local
Содержимое файла:
[DEFAULT]
bantime = 3600
ignoreip = 127.0.0.1/8 192.168.56.11
[sshd]
enabled = true
port = ssh
maxretry = 3
Перезапуск Fail2ban:
sudo systemctl restart fail2ban
5. Проверка работы Fail2ban
5.1 Проверка статуса
sudo fail2ban-client status
sudo fail2ban-client status sshd
Вывод:
Status for the jail: sshd |- Filter | |- Currently failed: 0 | |- Total failed: 0 | - File list:
/var/log/auth.log - Actions |- Currently banned: 0 |- Total banned: 0 `- Banned IP list:
5.2 Проверка блокировки
1. На сервере временно убрать IP клиента из ignoreip (если нужно проверить бан).
2. На клиенте сделать 2–3 попытки SSH с неправильным паролем:
ssh vagrant@192.168.56.10
3. На сервере проверить:
sudo fail2ban-client status sshd
IP клиента появляется в Banned IP list.
После проверки IP разблокирован:
sudo fail2ban-client set sshd unbanip 192.168.56.11
6. Размещение конфигурации для Vagrant provisioning
1. Создан каталог для конфигурации:
sudo mkdir -p /vagrant/provision/server/protect/etc/fail2ban/jail.d
2. Скопирован конфигурационный файл:
sudo cat /etc/fail2ban/jail.d/customisation.local | sudo tee /vagrant/provision/server/protec
7. Создание скрипта protect.sh
cd /vagrant/provision/server
nano protect.sh
Содержимое:
#!/bin/bash
echo "Provisioning script: $0"
echo "Install needed packages"
apt update
apt install -y fail2ban
echo "Copy configuration files"
cp -R /vagrant/provision/server/protect/etc/* /etc/
echo "Restart fail2ban service"
systemctl enable fail2ban
systemctl restart fail2ban
Сделать исполняемым:
chmod +x protect.sh
В блоке сервера добавить:
server.vm.provision "shell", path: "provision/server/protect.sh"
Проверка работы provisioning:
vagrant reload --provision
На сервере:
sudo fail2ban-client status
sudo fail2ban-client status sshd

## Контрольные вопросы

1. Поясните принцип работы Fail2ban
Fail2ban отслеживает логи служб (например, SSH, веб-серверов, почты) на наличие неудачных
попыток входа. При превышении заданного числа неудачных попыток (maxretry) он блокирует IP-
адрес на заданное время (bantime), используя встроенные firewall-правила (обычно iptables или
nftables). Таким образом предотвращается атака типа brute-force. ⸻
2. Настройки какого файла более приоритетны: jail.conf или jail.local?
jail.local имеет более высокий приоритет.
jail.conf — системный конфигурационный файл, не рекомендуется редактировать напрямую
jail.local — пользовательский файл, куда вносятся локальные изменения, которые
перекрывают значения из jail.conf. ⸻
3. Как настроить оповещение администратора при срабатывании Fail2ban?
В конфигурации можно использовать параметр destemail и action в jail.local:
[DEFAULT]
destemail = admin@example.com
action = %(action_mwl)s
action_mwl — отправка письма с логами и whois атакующего IP.
После перезапуска Fail2ban администратор будет получать уведомления по электронной
почте. ⸻
4. Поясните построчно настройки по умолчанию в конфигурации /etc/fail2ban/jail.conf,
относящиеся к веб-службе
Например для Apache (apache-auth):
[apache-auth] # Имя «jail» для проверки аутентификации веб-сервера
enabled = false # Включена ли проверка по умолчанию (false — отключена)
port = http,https # Порты, которые мониторятся
filter = apache-auth # Фильтр для анализа логов (см. /etc/fail2ban/filter.d/)
logpath = /var/log/apache*/* # Путь к логам веб-сервера
maxretry = 3 # Количество неудачных попыток до бана
5. Поясните построчно настройки по умолчанию в конфигурации /etc/fail2ban/jail.conf,
относящиеся к почтовой службе
Пример для Postfix (postfix) и Dovecot (dovecot):
[postfix]
enabled = false
port = smtp,ssmtp,submission
filter = postfix
logpath = /var/log/mail.log
maxretry = 5
[dovecot]
enabled = false
port = pop3,pop3s,imap,imaps
filter = dovecot
logpath = /var/log/mail.log
maxretry = 5
enabled — включение проверки
port — порты почтовой службы
filter — фильтр для логов
logpath — путь к логам
maxretry — количество ошибок перед баном
6. Какие действия может выполнять Fail2ban при обнаружении атакующего IP-адреса? Где
посмотреть описание действий?
Fail2ban может:
блокировать IP через iptables или nftables
отправлять уведомление на email (action_mwl)
выполнять произвольный скрипт (action с custom script)
7. Как получить список действующих правил Fail2ban?
sudo fail2ban-client status
8. Как получить статистику заблокированных адресов?
sudo fail2ban-client status sshd
9. Как разблокировать IP-адрес?
На сервере:
sudo fail2ban-client set sshd unbanip <IP-адрес>

## Вывод

Fail2ban установлен и работает
SSH-защита от brute-force настроена
IP клиента корректно блокируется и разблокируется
ignoreip работает
Provisioning скрипт позволяет автоматизировать установку и настройку Fail2ban