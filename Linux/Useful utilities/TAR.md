
Заархивировать один файл и сжать его с помощью gzip:
`tar -czvf my_archive.tar.gz file.txt`

Заархивировать несколько файлов и сжать их с помощью gzip:
`tar -czvf archive.tar.gz file1.txt file2.txt file3.txt`

Заархивировать один каталог и сжать его с помощью gzip:
`tar -czvf backup.tar.gz my_folder/`

Распаковать в текущий каталог:
`tar -xzvf archive.tar.gz`

Распаковать в указанный каталог:
`tar -xzvf archive.tar.gz -C /home/user/extracted_files`

Что означают флаги:
-c - создать новый архив
-z - сжать с использованием gzip
-v - вывести процесс на экран (verbose)
-f - указать имя файла архива
-С - изменить каталог (т.е., указать каталог для распаковки)



