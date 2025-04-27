# 🚀 Установка WireGuard VPN с WG Easy в Docker на Ubuntu 22.04

Source: https://github.com/WeeJeWel/wg-easy#updating

```bash
# 📝 Руководство по настройке WireGuard VPN сервера с веб-интерфейсом WG Easy на Ubuntu 22.04 в Docker.
# 🔐 Оптимизировано для нового сервера, минимизирует риск потери SSH-соединения.
# 🌐 Основано на: https://linuxiac.com/how-to-set-up-wireguard-vpn-with-docker/
📋 Предварительные требования
bash
# 🖥️ Сервер с Ubuntu 22.04 и доступом в интернет.
# 👤 Пользователь с правами sudo.
# 🌍 Публичный IP-адрес (например, 203.0.113.1) или домен.
# 🔌 Порты 51820/UDP (WireGuard) и 51821/TCP (WG Easy) открыты на маршрутизаторе/фаерволе.
# 📡 SSH-доступ (например, с MacBook).
🛠️ Шаг 1: Настройка SSH для сохранения соединения
bash
# 📡 Настройка keep-alive на клиентском устройстве (например, MacBook) для стабильного SSH.
mkdir -p ~/.ssh
nano ~/.ssh/config
text
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5
bash
chmod 600 ~/.ssh/config
🖥️ Шаг 2: Подготовка сервера
bash
# 🔗 Подключение к серверу.
ssh username@ваш_публичный_ip
bash
# 🛡️ Установка screen для защиты сессии.
sudo apt update
sudo apt install -y screen
screen
bash
# 🔄 Восстановление сессии при разрыве:
screen -r
bash
# 🔧 Обновление системы (не подтверждайте перезагрузку, чтобы сохранить SSH).
sudo apt update && sudo apt upgrade -y
bash
# 🛠️ Установка базовых инструментов.
sudo apt install -y curl wget nano
🐳 Шаг 3: Установка Docker
bash
# 📦 Установка зависимостей.
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
bash
# 🔑 Добавление GPG-ключа Docker.
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
bash
# 📚 Добавление репозитория Docker.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
bash
# 🐳 Установка Docker.
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
bash
# 🚀 Запуск и включение Docker.
sudo systemctl enable docker
sudo systemctl start docker
bash
# ✅ Проверка версии.
docker --version
bash
# 👤 Добавление пользователя в группу Docker.
sudo usermod -aG docker $USER
bash
# 🔄 Перезайдите:
# Выйдите из screen (Ctrl+A, D), закройте SSH, подключитесь заново:
screen -r
🧩 Шаг 4: Установка Docker Compose
bash
# 📥 Загрузка Docker Compose.
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
bash
# 🔧 Установка прав.
sudo chmod +x /usr/local/bin/docker-compose
bash
# ✅ Проверка версии.
docker-compose --version
🔒 Шаг 5: Настройка WG Easy
bash
# 📂 Создание директории для WG Easy.
mkdir ~/wg-easy && cd ~/wg-easy
bash
# 📝 Создание конфигурации.
nano docker-compose.yaml
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
bash
# ⚙️ Настройки:
# - your_public_ip: Публичный IP (например, 203.0.113.1) или домен.
# - your_secure_password: Пароль (например, MyStrongPass123!).
# Сохраните: Ctrl+O, Enter, Ctrl+X.
🛡️ Шаг 6: Настройка фаервола
bash
# 🔧 Установка ufw.
sudo apt install -y ufw
bash
# 🔓 Открытие портов.
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw allow 51821/tcp
–

```bash
# 🚀 Включение фаервола.
sudo ufw enable
bash
# ✅ Проверка статуса.
sudo ufw status
bash
# 🌐 Пробросьте порты 51820/UDP и 51821/TCP на маршрутизаторе, если сервер за NAT.
🌐 Шаг 7: Запуск WG Easy
bash
# 🚀 Запуск контейнера.
docker-compose up -d
bash
# ✅ Проверка контейнера.
docker ps
bash
# 🔍 Логи при ошибках.
docker logs wg-easy
🖥️ Шаг 8: Доступ к веб-интерфейсу WG Easy
bash
# 🌐 Откройте в браузере:
http://ваш_публичный_ip:51821
# 🔐 Войдите с паролем из PASSWORD.
# ➕ Создайте клиента:
# - Нажмите New Client.
# - Укажите имя (например, MacBook).
# - Скачайте .conf или отсканируйте QR-код.
📱 Шаг 9: Настройка клиента
bash
# 📲 Установка WireGuard на MacBook.
brew install wireguard-tools
bash
# 📥 Импортируйте .conf в приложение WireGuard или:
sudo wg-quick up ~/path/to/config.conf
✅ Шаг 10: Тестирование VPN
bash
# 🔌 Подключитесь к VPN.
ping 10.8.0.1
curl ifconfig.me
# Публичный IP должен совпадать с IP сервера.
🔑 Шаг 11: Обновление пароля
bash
# 📝 Изменение пароля.
cd ~/wg-easy
nano docker-compose.yaml
yaml
- PASSWORD=new_secure_password
bash
# 🔄 Перезапуск.
docker-compose up -d
🛠️ Шаг 12: Управление и обслуживание
bash
# 🔄 Обновление WG Easy.
cd ~/wg-easy
docker-compose pull
docker-compose up -d
bash
# 💾 Резервное копирование.
cp -r ~/wg-easy ~/wg-easy-backup
bash
# 🛑 Остановка.
cd ~/wg-easy
docker-compose down
⚠️ Шаг 13: Устранение неполадок
bash
# 🔌 Потеря SSH.
screen -r
bash
# 🌐 Веб-интерфейс недоступен.
sudo netstat -tuln | grep 51821
docker ps
docker logs wg-easy
bash
# 🔒 VPN не работает.
sudo ufw status
# Проверьте WG_HOST в docker-compose.yaml.
bash
# 🔄 Конфликт портов: Измените порты в docker-compose.yaml и обновите ufw/маршрутизатор.
🔐 Дополнительно
bash
# 🔒 Безопасность: Настройте HTTPS через Nginx.
# 🔄 Перенос клиентов: Добавьте ключи в WG Easy через веб-интерфейс (подсеть 10.8.0.x).
📜 Лицензия
# 🌐 Основано на: https://linuxiac.com/how-to-set-up-wireguard-vpn-with-docker/
