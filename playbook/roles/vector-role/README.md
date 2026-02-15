vector-role
=========
Роль Ansible создана для установки и настройки Vector,который отправляет логив в базу данных ClickHouse.

Requirements
---------
Ansible версии 2.9 или выше
Ubuntu 20.04 / 22.04 (другие Debian-совместимые дистрибутивы могут работать, но не тестировались)
Целевой хост должен иметь доступ в интернет для загрузки Vector и скрипта добавления репозитория.

Role Variables
---------
Доступные переменные со значениями по умолчанию (см. defaults/main.yml):

Переменная	               Описание	                                                                Значение по умолчанию
vector_version	           Версия Vector для установки	"0.31.0"
clickhouse_host	           IP-адрес или имя хоста ClickHouse (используется в шаблоне конфигурации)	"{{ groups            
                                                                                                    ['clickhouse_servers'][0] | default('clickhouse-01') }}"
clickhouse_port	           HTTP-порт ClickHouse	                                                    8123
vector_config_path	       Полный путь к конфигурационному файлу Vector	                            /etc/vector/vector.yml
vector_repo_script_url	   URL официального скрипта добавления репозитория Vector	                  "https://setup.vector.dev"
vector_repo_script_dest	   Временное расположение скрипта добавления репозитория	                  /tmp/setup-vector.sh
vector_log_file	           Путь к лог-файлу Vector (при запуске без systemd)	                      /var/log/vector.log
vector_pid_file	           Путь к PID-файлу (при запуске без systemd)	                              /var/run/vector.pid


Configuration Template
-----------
Роль разворачивает шаблон конфигурации из templates/vector.yml.j2. Вы можете переопределить этот шаблон или настроить переменные выше под своё окружение. Пример минимальной конфигурации, отправляющей тестовые логи в ClickHouse:
```
sources:
  dummy:
    type: "demo_logs"
    format: "json"
    interval: 1

sinks:
  clickhouse:
    type: "clickhouse"
    inputs: ["dummy"]
    endpoint: "http://{{ clickhouse_host }}:{{ clickhouse_port }}"
    database: "logs"
    table: "log"
    compression: "gzip"
    healthcheck: false
```    

Dependencies
------------
None.

Example Playbook
------------
```
- hosts: vector_servers
  become: true
  vars:
    clickhouse_host: "clickhouse-01"          # or use group from inventory
    clickhouse_port: 8123
    vector_version: "0.31.0"
  roles:
    - vector-role
```

Если у вас несколько хостов ClickHouse, адрес можно вычислить динамически:

```
yaml
- hosts: vector_servers
  become: true
  vars:
    clickhouse_host: "{{ groups['clickhouse_servers'][0] }}"
    clickhouse_ip: "{{ hostvars[clickhouse_host]['ansible_default_ipv4']['address'] }}"
  roles:
    - role: vector-role
      clickhouse_host: "{{ clickhouse_ip }}"
      clickhouse_port: 8123
```

License
---------
MIT

Author
---------
YuryShelukhin



