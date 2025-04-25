
Устанавливаем пакет iptables-persistent, если его еще нет на машине (в ходе установки необходимо будет ответить на вопрос, нужно ли сохранять действующие правила v4 и v6):
```
dpkg -l | grep iptables-persistent

sudo apt update
sudo apt install iptables-persistent -y
```

Разрешаем входящий трафик для портов 80 и 443:
```
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

Проверяем текущие правила (смотрим на новые записи в цепочке INPUT):
```
sudo iptables -L -v -n
```

Сохраняем правила для IPv4:
```
sudo netfilter-persistent save
```

(опционально) Сохраняем правила для IPv6, если таковые имеются:
```
sudo ip6tables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo ip6tables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo netfilter-persistent save
```

Правила будут сохранены в файлах:

- Для IPv4: `/etc/iptables/rules.v4`
- Для IPv6: `/etc/iptables/rules.v6`

Перегружаем сервер:
```
sudo reboot
```

Удостоверяемся, что правила сохранились после перезагрузки:
```
sudo iptables -L -v -n
```

