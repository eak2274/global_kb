
Останавливаем запущенные контейнеры:
```
docker-compose down
```

Убеждаемся, что контейнеры удалены:
```
docker ps -a
```

Если старые контейнеры все же есть, удаляем их вручную (если контейнеров нет - команда не сработает):
```
docker rm -f $(docker ps -aq)
```

Удаляем тома:
```
docker volume ls
docker volume prune
```

Удаляем образы Docker:
```
docker images
docker image prune -a
```

Удаляем файлы и подкаталоги в каталоге ~/airflow:
```
cd ~/airflow
rm -rf dags logs plugins data airflow.db
```

Очищаем кэш Docker:
```
docker system prune -a
```

