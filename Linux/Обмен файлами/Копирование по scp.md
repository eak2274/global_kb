Формат команды:
```
scp -i <path/to/private/key> <path/to/source/file> <user@host:path/to/dest/directory>
```

Пример:
```
scp -i "D:\Education\Universal\Cloud\Oracle\Instance 3\keys\ssh-key-2025-04-15.ppk" "D:\Education\Universal\Cloud\Oracle\Instance 3\Airflow\docker-compose.yaml" ubuntu@129.159.142.24:~/airflow/
```