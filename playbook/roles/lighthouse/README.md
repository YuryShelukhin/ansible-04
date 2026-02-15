lighthouse-role
===============
Роль Ansible создана для установки и настройки Lighthouse – легковесного веб-интерфейса для ClickHouse. Роль разворачивает Lighthouse с использованием Nginx в качестве веб-сервера и настраивает подключение к базе данных ClickHouse.

Requirements
------------
Ansible версии 2.9 или выше
Ubuntu 20.04 / 22.04 (другие Debian-совместимые дистрибутивы могут работать, но не тестировались)
Целевой хост должен иметь доступ в интернет для загрузки файлов Lighthouse с GitHub.

Role Variables
------------
Доступные переменные со значениями по умолчанию (см. defaults/main.yml):

Переменная	                Описание	                                                                     Значение по умолчанию
lighthouse_root	            Корневая директория, куда помещаются файлы Lighthouse	                         /var/www/lighthouse
lighthouse_version	        Ветка Git или тег Lighthouse для загрузки	                                     master
lighthouse_clickhouse_ip	  IP-адрес сервера ClickHouse (используется для замены 127.0.0.1 в index.html)	 "" (должен быть  
                                                                                                           передан)
lighthouse_clickhouse_port	HTTP-порт ClickHouse	                                                         8123
lighthouse_nginx_site_name	Имя конфигурации сайта Nginx	                                                 lighthouse
lighthouse_nginx_port	Порт, на котором будет слушать Nginx	                                               80

Примечание: Переменная lighthouse_clickhouse_ip обязательна для корректной замены адреса ClickHouse в файле index.html. Если она не передана, задача обновления index.html будет пропущена.

Internal Variables (defined in vars/main.yml)
---------------------------------------------
Variable	                                                              Description
lighthouse_files	List of files to download from the Lighthouse repository (index.html, app.js, jquery.js, LICENSE, README.md)

Configuration Template
----------------------
Роль разворачивает конфигурацию сайта Nginx из шаблона templates/lighthouse.conf.j2. Типичный шаблон выглядит так:
```
jinja2
server {
    listen       {{ lighthouse_nginx_port }};
    server_name  _;
    root         {{ lighthouse_root }};
    index        index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ lighthouse_nginx_site_name }}/access.log;
    error_log  /var/log/nginx/{{ lighthouse_nginx_site_name }}/error.log;
}
```

Dependencies
------------
None.

Example Playbook
----------------
```
yaml
- hosts: lighthouse_servers
  become: true
  vars:
    # Assume clickhouse_servers group exists in inventory
    clickhouse_host: "{{ groups['clickhouse_servers'][0] }}"
    # Get ClickHouse IP address – prefer ansible_host if set, otherwise default IPv4
    clickhouse_ip: >-
      {{ hostvars[clickhouse_host]['ansible_host'] |
         default(hostvars[clickhouse_host]['ansible_default_ipv4']['address']) }}
    clickhouse_port: 8123
  roles:
    - role: lighthouse-role
      lighthouse_clickhouse_ip: "{{ clickhouse_ip }}"
      lighthouse_clickhouse_port: "{{ clickhouse_port }}"
      lighthouse_nginx_site_name: lighthouse
      # optionally override other variables:
      # lighthouse_root: /opt/lighthouse
      # lighthouse_nginx_port: 8080
```

Если Lighthouse устанавливается на том же хосте, что и ClickHouse, можно просто передать lighthouse_clickhouse_ip: 127.0.0.1.

License
-------
MIT

Author
------
YuryShelukhin