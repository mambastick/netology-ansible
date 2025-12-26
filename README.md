# Домашнее задание к занятию «08.04 Работа с roles»

Плейбук `site.yml` устанавливает ClickHouse, Vector и LightHouse через роли.

## Подготовка окружения

- Установить зависимости из `requirements.yml`: `ansible-galaxy install -r requirements.yml -p roles`.
- Используем инвентарь `inventory/prod.yml`.

## Инвентарь

- `clickhouse`: один хост `clickhouse-01`.
- `vector`: один хост `vector-01`.
- `lighthouse`: один хост `lighthouse-01`.

## Переменные

Основные переменные вынесены в `group_vars`:

- `group_vars/clickhouse.yml`: версия ClickHouse, список пакетов, БД `logs` для демо-логов.
- `group_vars/vector.yml`: версия Vector, ссылка на rpm, директория конфига и параметры sink в ClickHouse.

Дополнительные параметры каждой роли описаны в `roles/*/README.md`.

## Роли и playbook

- `clickhouse` — берётся из внешнего репозитория `AlexeySetevoi/ansible-clickhouse`.
- `vector` — скачивает rpm, ставит и настраивает Vector, заменяет unit systemd.
- `lighthouse` — ставит nginx, разворачивает VK LightHouse из Git и публикует через virtual host.

`site.yml` содержит три play — по одному на каждую группу хостов. После установки ClickHouse создаётся база `logs` через переменные роли.

## Запуск

```bash
ansible-playbook -i inventory/prod.yml site.yml
```
