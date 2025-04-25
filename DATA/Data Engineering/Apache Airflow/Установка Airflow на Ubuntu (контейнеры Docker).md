Подключаемся к удаленному серверу:
`ssh -i "path/to/ssh/private/key.ppk" ubuntu@<server ip address>`

Обновляем пакеты на сервере во избежание проблем с зависимостями:
`sudo apt update && sudo apt upgrade -y`

### Установка docker:
```
# Установка необходимых зависимостей
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Добавление GPG-ключа Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Обновление списка пакетов
sudo apt update

# Установка Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Убеждаемся, что docker установлен и работоспособен:
```
sudo docker --version
sudo systemctl status docker
```

Добавляем своего пользователя в группу docker, чтобы не использовать sudo при работе с docker:
`sudo usermod -aG docker $USER`

Выходим с помощью команды `exit` и снова заходим на сервер.

### Установка Docker Compose:

Скачиваем последнюю версию docker-compose:
`sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

Даем права на выполнение:
`sudo chmod +x /usr/local/bin/docker-compose`

Проверяем, что docker-compose установлен:
`docker-compose --version`

### Настройка Apache Airflow

Создаем рабочий каталог и переходим в него:
```
mkdir ~/airflow
cd ~/airflow
```

Создаем необходимые директории внутри airflow:
```
mkdir -p ~/airflow/dags ~/airflow/logs ~/airflow/plugins ~/airflow/data
```

Предоставляем пользователю ubuntu требуемые права на эти директории:
```
chmod -R u+rwX ~/airflow/dags ~/airflow/logs ~/airflow/plugins ~/airflow/data
chown -R $(id -u):$(id -g) ~/airflow/dags ~/airflow/logs ~/airflow/plugins ~/airflow/data
```

Создаем файл `.env` для хранения переменных окружения (это необходимо для корректной работы контейнеров):
```
echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=$(id -g)" > ~/airflow/.env
cat ~/airflow/.env
```

Вносим корректировки, если требуется, и копируем файл docker-compose.yml на удаленную машину:
`scp -i "D:\Education\Universal\Cloud\Oracle\Instance 3\keys\ssh-key-2025-04-15.ppk" "D:\Education\Universal\Cloud\Oracle\Instance 3\Airflow\docker-compose.yaml" ubuntu@129.159.142.24:~/airflow/`

Инициализируем БД с метаданными Airflow:
`docker-compose up airflow-init`

Дожидаемся окончания выполнения предыдущей команды и поднимаем все сервисы:
