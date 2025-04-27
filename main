Конечно! Я создам файл в формате Markdown (README.md), который вы сможете использовать на GitHub. Он будет содержать полное руководство по установке WireGuard VPN с WG Easy в Docker на Ubuntu 22.04, основанное на нашей работе. Файл будет структурирован, понятен и готов для публикации в репозитории. Руководство включает все шаги, которые мы прошли, с учётом вашего опыта (работа через SSH с MacBook, новый сервер, сохранение SSH-соединения, переход с WireGuard-UI на WG Easy) и устранения ошибок (--no-restart, ~/.ssh/config).
Содержимое файла README.md
markdown
# Установка WireGuard VPN с WG Easy в Docker на Ubuntu 22.04

Это руководство описывает, как настроить **WireGuard VPN сервер** с веб-интерфейсом **WG Easy** на чистом сервере Ubuntu 22.04, используя Docker. WG Easy предоставляет удобный интерфейс для управления клиентами VPN. Руководство оптимизировано для работы через SSH, минимизирует риск потери соединения и подходит для новых серверов.

## Предварительные требования

- Новый сервер с **Ubuntu 22.04** и доступом в интернет.
- Пользователь с правами **sudo**.
- Публичный IP-адрес сервера (например, `203.0.113.1`) или доменное имя.
- Порты **51820/UDP** (WireGuard) и **51821/TCP** (WG Easy веб-интерфейс) открыты на маршрутизаторе/фаерволе.
- SSH-доступ с клиентского устройства (например, MacBook).

## Шаг 1: Настройка SSH для сохранения соединения

Чтобы избежать разрыва SSH-соединения, настройте keep-alive на клиентском устройстве.

1. Откройте или создайте файл `~/.ssh/config` на клиентском устройстве (например, MacBook):
   ```bash
   mkdir -p ~/.ssh
   nano ~/.ssh/config
Добавьте:
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5
Сохраните (Ctrl+O, Enter, Ctrl+X) и установите права:
bash
chmod 600 ~/.ssh/config
Шаг 2: Подготовка сервера
Подключитесь к серверу по SSH:
bash
ssh username@ваш_публичный_ip
Установите и запустите screen для защиты сессии:
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
Примечание: Не подтверждайте перезагрузку, если предложена, чтобы сохранить SSH.
Установите базовые инструменты:
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
Запустите и включите Docker:
bash
sudo systemctl enable docker
sudo systemctl start docker
Проверьте версию:
bash
docker --version
Добавьте пользователя в группу Docker:
bash
sudo usermod -aG docker $USER
Перезайдите в сессию:
Выйдите из screen (Ctrl+A, D).
Закройте SSH, подключитесь заново, восстановите: screen -r.
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
WG Easy объединяет WireGuard и веб-интерфейс в одном контейнере.
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
Настройки:
your_public_ip: Публичный IP сервера (например, 203.0.113.1) или домен.
your_secure_password: Надёжный пароль для веб-интерфейса (например, MyStrongPass123!).
Порты: 51820/UDP (WireGuard), 51821/TCP (веб-интерфейс).
Сохраните (Ctrl+O, Enter, Ctrl+X).
Шаг 6: Настройка фаервола
Установите ufw, если не установлен:
bash
sudo apt install -y ufw
Разрешите порты:
bash
sudo ufw allow 22/tcp     # SSH
sudo ufw allow 51820/udp  # WireGuard
sudo ufw allow 51821/tcp  # WG Easy веб-интерфейс
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
Проверьте, что контейнер работает:
bash
docker ps
Если возникли ошибки, просмотрите логи:
bash
docker logs wg-easy
Шаг 8: Доступ к веб-интерфейсу WG Easy
Откройте браузер и перейдите:
http://ваш_публичный_ip:51821
Войдите, используя пароль из PASSWORD (например, MyStrongPass123!).
Создайте клиента:
Нажмите New Client.
Укажите имя (например, MacBook).
Скачайте .conf файл или отсканируйте QR-код.
Шаг 9: Настройка клиента
Установите WireGuard на клиентское устройство:
На MacBook: Загрузите WireGuard из Mac App Store или через Homebrew:
bash
brew install wireguard-tools
Импортируйте конфигурацию:
В приложении WireGuard выберите «Import tunnel(s) from file» и загрузите .conf.
Или через терминал:
bash
sudo wg-quick up ~/path/to/config.conf
Активируйте VPN.
Шаг 10: Тестирование VPN
Подключитесь к VPN.
Проверьте:
Пинг: ping 10.8.0.1
Публичный IP: curl ifconfig.me (должен совпадать с IP сервера).
Шаг 11: Обновление пароля
Чтобы изменить пароль веб-интерфейса:
Откройте docker-compose.yaml:
bash
cd ~/wg-easy
nano docker-compose.yaml
Измените PASSWORD:
yaml
- PASSWORD=new_secure_password
Перезапустите контейнер:
bash
docker-compose up -d
Проверьте новый пароль в веб-интерфейсе.
Шаг 12: Управление и обслуживание
Добавление клиентов:
Создавайте новых клиентов через веб-интерфейс WG Easy.
Обновление WG Easy:
bash
cd ~/wg-easy
docker-compose pull
docker-compose up -d
Резервное копирование:
Сохраните ~/wg-easy/config и docker-compose.yaml:
bash
cp -r ~/wg-easy ~/wg-easy-backup
Остановка:
bash
cd ~/wg-easy
docker-compose down
Шаг 13: Устранение неполадок
Потеря SSH:
Подключитесь заново: screen -r.
Веб-интерфейс недоступен:
Проверьте порт: sudo netstat -tuln | grep 51821.
Убедитесь, что контейнер работает: docker ps.
Логи: docker logs wg-easy.
VPN не работает:
Проверьте порт 51820/UDP: sudo ufw status.
Убедитесь, что WG_HOST в docker-compose.yaml правильный.
Конфликт портов:
Измените порты в docker-compose.yaml (например, 51822:51822/udp) и обновите ufw и маршрутизатор.
Дополнительно
Безопасность:
Используйте надёжный пароль для WG Easy.
Настройте HTTPS для веб-интерфейса через обратный прокси (например, Nginx).
Перенос клиентов (если ранее использовался другой интерфейс, например, WireGuard-UI):
Скопируйте приватные и публичные ключи клиентов.
Добавьте их вручную в WG Easy через веб-интерфейс, используя подсеть 10.8.0.x.
Лицензия
Это руководство распространяется под лицензией MIT.
Создано на основе статьи: How to Set Up WireGuard VPN with Docker.

---

### Как использовать файл на GitHub

1. **Создайте репозиторий**:
   - Перейдите на GitHub и создайте новый репозиторий (например, `wireguard-wg-easy`).

2. **Создайте файл `README.md`**:
   - В репозитории нажмите **Create new file**.
   - Назовите файл `README.md`.
   - Скопируйте и вставьте содержимое из приведённого выше кода.

3. **Сохраните файл**:
   - Прокрутите вниз и нажмите **Commit new file**.

4. **(Опционально) Добавьте `docker-compose.yaml`**:
   - Создайте ещё один файл в репозитории, назвав его `docker-compose.yaml`.
   - Скопируйте содержимое из шага 5 руководства:
     ```yaml
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
     ```
   - Сохраните с комментарием, например, `Add docker-compose.yaml`.

5. **Публикация**:
   - Теперь ваше руководство и конфигурация доступны на GitHub. Вы можете клонировать репозиторий или делиться ссылкой.

---

### Дополнительные рекомендации
- **Структура репозитория**:
  Если хотите, можете добавить папку `docs` для дополнительных файлов или скриптов. Например:
  wireguard-wg-easy/
  ├── README.md
  ├── docker-compose.yaml
  └── docs/
      └── troubleshooting.md
- **Troubleshooting файл**:
Если нужен отдельный файл с устранением неполадок, я могу подготовить его.
- **Лицензия**:
В `README.md` указана лицензия MIT. Если нужна другая (например, CC BY-SA), дайте знать.

Если вы хотите внести изменения в файл (например, добавить больше деталей, изменить форматирование или включить дополнительные файлы), напишите, и я обновлю содержимое!
