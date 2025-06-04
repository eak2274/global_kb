
В PowerShell выполняем команду
```
ssh-keygen -t ed25519 -C "sparepart74@gmail.com" -f C:\Users\Пользователь\ssh_keys\github\id_ed25519_github
```

Выдать необходимые права:

Пройти аутентификацию с помощью команды:
```
ssh -i "C:\Users\Пользователь\ssh_keys\github\id_ed25519_github" -T git@github.com
```

Установить для всех локальных репозиториев путь к частному ключу:
```
git config --global core.sshCommand "ssh -i C:/Users/Пользователь/ssh_keys/github/id_ed25519_github"
```


