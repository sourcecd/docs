---
sourcePath: ru/ydb/ydb-docs-core/ru/core/maintenance/_includes/backup_and_recovery/04_fs_backup_1_header.md
---
## Резервное копирование на файловую систему {#filesystem_backup}

Сохранение структуры директории `backup` в базе `$YDB_DB_PATH` в директорию `my_backup_of_basic_example` на файловой системе.
```
{{ ydb-cli }} -e $YDB_ENDPOINT -d $YDB_DB_PATH tools dump -p $YDB_DB_PATH/backup -o my_backup_of_basic_example/
```

Для каждой директории в базе будет создана директория на файловой системе. Для каждой таблицы так же будет создана директория на файловой системе, в которую будет помещён файл с описанием структуры. Данные таблиц будут сохранены в одном или нескольких файлах в формате `csv`, по одной строке в файле для строки таблицы.
