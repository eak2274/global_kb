
Сохраняем существующие правила iptables на всякий случай:
```
sudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null
sudo ip6tables-save | sudo tee /etc/iptables/rules.v6 > /dev/null
```

Проверяем содержимое сохраненных файлов:
```
cat /etc/iptables/rules.v4
cat /etc/iptables/rules.v6
```

Устанавливаем ufw:
```
sudo apt update
sudo apt install ufw -y
```

Проверяем, что ufw установлен, смотрим его статус:
```
dpkg -l | grep ufw
sudo ufw status
which ufw
```

Настраиваем базовые правила ufw:
```
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Смотрим список правил, добавленных через ufw:
```
sudo ufw show added
```

Включаем файрвол и применяем правила:
```
sudo ufw enable
```

Проверяем статус:
```
sudo ufw status verbose
```

Подключаемся по ssh из другого окна терминала для проверки.