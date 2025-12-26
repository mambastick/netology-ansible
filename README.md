# Домашнее задание «08.04 Работа с roles»

Плейбук `site.yml` ставит три продукта через роли: ClickHouse, Vector и VK LightHouse.

## Состав репозитория

- `inventory/prod.yml` — описаны группы `clickhouse`, `vector`, `lighthouse`.
- `group_vars/` — основные переменные ролей.
- `roles/` — локальные роли `vector`, `lighthouse`; внешняя роль `clickhouse` тянется из Git.
- `requirements.yml` — список ролей для ansible-galaxy.

## Требования

- Ansible 2.9+.
- Доступ по SSH к хостам из `inventory/prod.yml`.
- На хостах RHEL/CentOS совместимые менеджеры пакетов (`yum`/`dnf`).

## Установка зависимостей

```bash
ansible-galaxy install -r requirements.yml -p roles
```

## Ключевые переменные

`group_vars/clickhouse.yml`
- `clickhouse_version` — версия ClickHouse.
- `clickhouse_packages` — список rpm.
- `clickhouse_dbs_custom` — создаваемые БД (по умолчанию `logs`).
- `clickhouse_listen_host` — список listen_host (по умолчанию `::`).

`group_vars/vector.yml`
- `vector_version` — версия Vector.
- `vector_download_url` — ссылка на rpm.
- `vector_config_dir` — путь к конфигу.
- `vector_config` — конфигурация источников/приёмников (sink в ClickHouse).
- `vector_clickhouse_host`/`vector_clickhouse_port` — куда писать логи в ClickHouse.

`roles/lighthouse/defaults/main.yml`
- `lighthouse_root` — куда клонируется UI.
- `lighthouse_listen_port` — порт nginx (80).
- `lighthouse_clickhouse_host`/`lighthouse_clickhouse_port` — адрес ClickHouse для UI; nginx проксирует `/clickhouse/` на этот endpoint, чтобы избежать CORS.

Подробнее см. `roles/vector/README.md` и `roles/lighthouse/README.md`.

## Playbook

`site.yml` запускает три play:
- `clickhouse` — устанавливает ClickHouse из внешней роли, создаёт БД `logs`.
- `vector` — скачивает и ставит rpm, рендерит конфиг и unit, запускает сервис.
- `lighthouse` — ставит nginx, клонирует VK LightHouse, настраивает vhost и прокси `/clickhouse/`.

## Быстрый старт

1. Проверьте/правьте `inventory/prod.yml` под свои хосты.
2. Установите роли: `ansible-galaxy install -r requirements.yml -p roles`.
3. Запустите:  
   ```bash
   ansible-playbook -i inventory/prod.yml site.yml
   ```
4. После деплоя:
   - `curl http://<clickhouse-host>:8123/` → должно вернуть `Ok.`.
   - В браузере откройте `http://<lighthouse-host>/` (порт 80).  
     В настройках LightHouse используйте endpoint `http://<lighthouse-host>/clickhouse` — запросы пойдут через nginx на ClickHouse.

## Полезные проверки

- Статус сервисов: `systemctl status clickhouse-server`, `systemctl status vector`, `systemctl status nginx`.
- Доступность прокси: `curl -i http://<lighthouse-host>/clickhouse/` → `Ok.`.
- Логи:
  - ClickHouse: `/var/log/clickhouse-server/clickhouse-server.log`
  - Vector: `/var/log/vector/vector.log` (если включён)
  - nginx: `/var/log/nginx/{access,error}.log`

## Что изменить под себя

- Пересобрать `vector_config` для нужных источников/приёмников.
- Поменять `lighthouse_listen_port` или `lighthouse_server_name`, если нужен нестандартный vhost.
- Добавить авторизацию в ClickHouse и прокинуть её в UI (через настройки LightHouse).
