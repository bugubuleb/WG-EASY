# Установка WireGuard VPN с WG Easy в Docker на Ubuntu 22.04

Руководство по настройке **WireGuard VPN сервера** с веб-интерфейсом **WG Easy** на Ubuntu 22.04 в Docker. WG Easy упрощает управление клиентами VPN. Подходит для нового сервера, минимизирует риск потери SSH-соединения.

## Предварительные требования

- Сервер с **Ubuntu 22.04** и доступом в интернет.
- Пользователь с правами **sudo**.
- Публичный IP-адрес (например, `203.0.113.1`) или домен.
- Порты **51820/UDP** (WireGuard) и **51821/TCP** (WG Easy) открыты на маршрутизаторе/фаерволе.
- SSH-доступ (например, с MacBook).

## Шаг 1: Настройка SSH для сохранения соединения

Настройте keep-alive на клиентском устройстве (например, MacBook).


bash
mkdir -p ~/.ssh
nano ~/.ssh/config

Добавьте в файл:

Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5

bash

chmod 600 ~/.ssh/config

Шаг 2: Подготовка сервера
Подключитесь к серверу:

bash

ssh username@ваш_публичный_ip

Установите и запустите screen:

bash

sudo apt update
sudo apt install -y screen
screen

Если соединение прервётся, восстановите:
bash

screen -r

Обновите систему:

bash

sudo apt update && sudo apt upgrade -y

Примечание: Не подтверждайте перезагрузку, чтобы сохранить SSH.

Установите инструменты:

bash

sudo apt install -y curl wget nano

Шаг 3: Установка Docker
Установите зависимости:

bash

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

Добавьте GPG-ключ Docker:

bash

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

Добавьте репозиторий Docker:

bash

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Установите Docker:

bash

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

Запустите Docker:

bash

sudo systemctl enable docker
sudo systemctl start docker

Проверьте версию:

bash

docker --version

Добавьте пользователя в группу Docker:

bash

sudo usermod -aG docker $USER

Перезайдите:
Выйдите из screen (Ctrl+A, D).

Закройте SSH, подключитесь заново:

bash

screen -r

Шаг 4: Установка Docker Compose
Загрузите Docker Compose:

bash

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

Сделайте файл исполняемым:

bash

sudo chmod +x /usr/local/bin/docker-compose

Проверьте версию:

bash

docker-compose --version

Шаг 5: Настройка WG Easy
Создайте директорию:

bash

mkdir ~/wg-easy && cd ~/wg-easy

Создайте файл docker-compose.yaml:

bash

nano docker-compose.yaml

Вставьте конфигурацию, заменив your_public_ip и your_secure_password:

yaml

version: "3.8"
services:
  wg-easy:
    image: weejewel/wg-easy:latest
    container_name: wg-easy
    environment:
      - WG_HOST=your_public_ip
      - PASSWORD=your_secure_password
      - WG_PORT=51820
      - WG_DEFAULT_DNS=8.8.8.8
      - WG_DEFAULT_ADDRESS=10.8.0.x
      - WG_MTU=1420
    volumes:
      - ./config:/etc/wireguard
    ports:
      - 51820:51820/udp
      - 51821:51821/tcp
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

your_public_ip: Публичный IP (например, 203.0.113.1) или домен.

your_secure_password: Пароль для веб-интерфейса (например, MyStrongPass123!).

Сохраните (Ctrl+O, Enter, Ctrl+X).

Шаг 6: Настройка фаервола
Установите ufw:

bash

sudo apt install -y ufw

Разрешите порты:

bash

sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw allow 51821/tcp

Включите фаервол:

bash

sudo ufw enable

Проверьте:

bash

sudo ufw status

Примечание: Пробросьте порты 51820/UDP и 51821/TCP на маршрутизаторе, если сервер за NAT.

Шаг 7: Запуск WG Easy
Запустите контейнер:

bash

docker-compose up -d

Проверьте контейнер:

bash

docker ps

Просмотрите логи при ошибках:

bash

docker logs wg-easy

Шаг 8: Доступ к веб-интерфейсу WG Easy
Откройте браузер:

http://ваш_публичный_ip:51821

Войдите с паролем из PASSWORD.

Создайте клиента:
Нажмите New Client.

Укажите имя (например, MacBook).

Скачайте .conf или отсканируйте QR-код.

Шаг 9: Настройка клиента
Установите WireGuard:
На MacBook: Через Mac App Store или Homebrew:

bash

brew install wireguard-tools

Импортируйте конфигурацию:
В приложении WireGuard: «Import tunnel(s) from file» → выберите .conf.

Или через терминал:

bash

sudo wg-quick up ~/path/to/config.conf

Активируйте VPN.

Шаг 10: Тестирование VPN
Подключитесь к VPN.

Проверьте:

bash

ping 10.8.0.1
curl ifconfig.me

Публичный IP должен совпадать с IP сервера.
Шаг 11: Обновление пароля
Откройте docker-compose.yaml:

bash

cd ~/wg-easy
nano docker-compose.yaml

Измените PASSWORD:

yaml

- PASSWORD=new_secure_password

Перезапустите:

bash

docker-compose up -d

Проверьте новый пароль.

Шаг 12: Управление и обслуживание
Добавление клиентов: Через веб-интерфейс WG Easy.

Обновление:

bash

cd ~/wg-easy
docker-compose pull
docker-compose up -d

Резервное копирование:

bash

cp -r ~/wg-easy ~/wg-easy-backup

Остановка:

bash

cd ~/wg-easy
docker-compose down

Шаг 13: Устранение неполадок
Потеря SSH:

bash

screen -r

Веб-интерфейс недоступен:

bash

sudo netstat -tuln | grep 51821
docker ps
docker logs wg-easy

VPN не работает:

bash

sudo ufw status

Проверьте WG_HOST в docker-compose.yaml.
Конфликт портов: Измените порты в docker-compose.yaml и обновите ufw/маршрутизатор.

