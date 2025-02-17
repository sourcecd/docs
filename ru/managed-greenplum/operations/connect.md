# Подключение к базе данных в кластере {{ GP }}

Благодаря тому, что СУБД {{ GP }} основана на {{ PG }}, для подключения к обеим СУБД используются одни и те же инструменты.

Подключение к кластеру {{ mgp-short-name }} производится только через [первичный хост-мастер](../concepts/index.md). Чтобы определить роли хостов, получите [список хостов в кластере](./cluster-hosts.md#list).

К кластеру можно подключиться:

{% include [cluster-connect-note-monolithic](../../_includes/mdb/cluster-connect-note-monolithic.md) %}

## Настройка групп безопасности {#configuring-security-groups}

{% include [sg-rules](../../_includes/mdb/sg-rules-connect.md) %}

Настройки правил будут различаться в зависимости от выбранного способа подключения:

{% list tabs %}

* Через интернет

  Настройте все группы безопасности кластера так, чтобы они разрешали входящий трафик с любых IP-адресов на порт {{ port-mgp }}. Для этого [создайте правило](../../vpc/operations/security-group-update.md#add-rule) для входящего трафика:

  * протокол: `TCP`;
  * диапазон портов: `{{ port-mgp }}`;
  * тип источника: `CIDR`;
  * источник: `0.0.0.0/0`.

* С ВМ в Облаке

  1. Настройте все группы безопасности кластера так, чтобы они разрешали входящий трафик из группы безопасности, в которой находится ВМ, на порт {{ port-mgp }}. Для этого в этих группах [создайте правило](../../vpc/operations/security-group-update.md#add-rule) для входящего трафика:

     * протокол: `TCP`;
     * диапазон портов: `{{ port-mgp }}`;
     * тип источника: `Группа безопасности`;
     * источник: группа безопасности, в которой находится ВМ. Если она совпадает с настраиваемой группой, то укажите **Текущая**.
  
  1. [Настройте группу безопасности](../../vpc/operations/security-group-update.md#add-rule), в которой находится ВМ так, чтобы можно было подключаться к ВМ и был разрешен трафик между ВМ и хостами кластера.
  
     Пример правил для ВМ:
  
     * Для входящего трафика:
       * протокол: `TCP`;
       * диапазон портов: `22`;
       * тип источника: `CIDR`;
       * источник: `0.0.0.0/0`.

       Это правило позволяет подключаться к ВМ по протоколу SSH.
  
     * Для исходящего трафика:
        * протокол: `Any`;
        * диапазон портов: `0-65535`;
        * тип назначения: `CIDR`;
        * назначение: `0.0.0.0/0`.
        
       Это правило разрешает любой исходящий трафик, что позволяет не только подключаться к кластеру, но и устанавливать сертификаты и утилиты, необходимые для подключения к кластеру.

{% endlist %}

{% note info %}

Вы можете задать более детальные правила для групп безопасности, например, разрешающие трафик только в определенных подсетях.

При неполных или некорректных настройках групп безопасности можно потерять доступ к кластеру, если произойдет смена первичного хоста-мастера.

{% endnote %}

Подробнее см. в разделе [{#T}](../concepts/network.md#security-groups).

## Автоматический выбор первичного мастера {#automatic-master-host-selection}

Чтобы выбирать хост для подключения к кластеру автоматически, используйте [особый FQDN первичного мастера](#fqdn-master).

## Получение SSL-сертификата {#get-ssl-cert}

Чтобы использовать SSL-соединение, получите сертификат:

{% list tabs %}

* Linux (Bash)

  
  ```bash
  mkdir ~/.postgresql && \
  wget "https://{{ s3-storage-host }}{{ pem-path }}" -O ~/.postgresql/root.crt && \
  chmod 0600 ~/.postgresql/root.crt
  ```

* Windows (PowerShell)

  
  ```powershell
  mkdir $HOME\AppData\Roaming\postgresql
  curl.exe -o $HOME\AppData\Roaming\postgresql\root.crt https://{{ s3-storage-host }}{{ pem-path }}
  ```

{% endlist %}

{% include [ide-ssl-cert](../../_includes/mdb/mdb-ide-ssl-cert.md) %}

## Подключение из графических IDE {#connection-ide}

{% include [ide-environments](../../_includes/mdb/mdb-ide-envs.md) %}

Подключаться из графических IDE можно только к кластеру с публичным доступом. Перед подключением [подготовьте сертификат](#get-ssl-cert).

{% list tabs %}

* DataGrip

  1. Создайте источник данных:
      1. Выберите в меню **File** → **New** → **Data Source** → **{{ GP }}**.
      1. Укажите параметры подключения на вкладке **General**:
          * **User**, **Password** — имя и пароль пользователя БД;
          * **URL** — строка подключения:
            В строке подключения используйте [особый FQDN первичного мастера](#fqdn-master):
            ```http
            jdbc:postgresql://c-<идентификатор кластера>.rw.{{ dns-zone }}:{{ port-mgp }}/<имя БД>
            ```
          * Нажмите ссылку **Download**, чтобы загрузить драйвер соединения.
      1. На вкладке **SSH/SSL**:
          1. Включите настройку **Use SSL**.
          1. В поле **CA file** укажите путь к файлу [SSL-сертификата для подключения](#get-ssl-cert).
  1. Нажмите ссылку **Test Connection** для проверки подключения. При успешном подключении будет выведен статус подключения, информация о СУБД и драйвере.
  1. Нажмите кнопку **OK**, чтобы сохранить источник данных.

* DBeaver

  1. Создайте новое соединение с БД:
      1. Выберите в меню **База данных** пункт **Новое соединение**.
      1. Выберите из списка БД **{{ GP }}**.
      1. Нажмите кнопку **Далее**.
      1. Укажите параметры подключения на вкладке **Главное**:
          * **Хост** — [особый FQDN первичного мастера](#fqdn-master): `c-<идентификатор кластера>.rw.{{ dns-zone }}`;
          * **Порт** — `{{ port-mgp }}`;
          * **База данных** — имя БД для подключения;
          * В блоке **Аутентификация** укажите имя и пароль пользователя БД.
      1. На вкладке **SSL**:
          1. Включите настройку **Использовать SSL**.
          1. В поле **Корневой сертификат** укажите путь к файлу [SSL-сертификата для подключения](#get-ssl-cert).
  1. Нажмите кнопку **Тест соединения ...** для проверки подключения. При успешном подключении будет выведен статус подключения, информация о СУБД и драйвере.
  1. Нажмите кнопку **Готово**, чтобы сохранить настройки соединения с БД.

{% endlist %}

## Примеры строк подключения {#connection-string}

{% include [conn-strings-environment](../../_includes/mdb/mgp/conn-strings-env.md) %}

При создании кластера {{ GP }} пользовательская база данных не создается. Для проверки подключения используйте служебную базу `postgres`.

Для подключения к кластеру с публичным доступом [подготовьте SSL-сертификат](#get-ssl-cert).

В примерах предполагается, что SSL-сертификат `root.crt` расположен в директории:

* `/home/<домашняя директория>/.postgresql/` для Ubuntu;
* `$HOME\AppData\Roaming\postgresql` для Windows.

Подключиться к кластеру можно как с использованием обычного FQDN первичного мастера, так и его [особого FQDN](#fqdn-master).

{% include [see-fqdn-in-console](../../_includes/mdb/see-fqdn-in-console.md) %}

{% include [mgp-connection-strings](../../_includes/mdb/mgp/conn-strings.md) %}

При успешном подключении и выполнении тестового запроса кластер вернет используемые версии {{ PG }} и {{ GP }}.

## Особый FQDN первичного мастера {#fqdn-master}

Наравне с обычными FQDN, которые можно запросить со списком хостов в кластере, {{ mgp-name }} предоставляет особый FQDN, который также можно использовать для подключения к кластеру.

FQDN вида `c-<идентификатор кластера>.rw.{{ dns-zone }}` всегда указывает на первичный хост-мастер. К этому FQDN разрешено подключаться и выполнять операции чтения и записи.

Пример подключения к первичному хосту-мастеру в кластере с идентификатором `c9qash3nb1v9ulc8j9nm`:

```bash
psql "host=c-c9qash3nb1v9ulc8j9nm.rw.{{ dns-zone }} \
      port={{ port-mgp }} \
      sslmode=verify-full \
      dbname=<имя БД> \
      user=<имя пользователя>"
```

{% include [greenplum-trademark](../../_includes/mdb/mgp/trademark.md) %}
