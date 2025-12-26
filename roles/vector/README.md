# Vector role

Роль устанавливает Vector из официальных rpm-пакетов, разворачивает конфигурацию и unit systemd.

## Requirements

Подходит для дистрибутивов с `yum`/`dnf`. Vector binary должен уметь проходить команду `vector validate`.

## Role Variables

- `vector_version` (default: `0.22.1`) — версия Vector.
- `vector_download_url` — ссылка на rpm-пакет. По умолчанию собирается из `vector_version`.
- `vector_package_dest` (default: `/tmp/vector-<version>-1.x86_64.rpm`) — путь, куда скачивается пакет.
- `vector_config_dir` (default: `/etc/vector`) — каталог для конфигурации.
- `vector_config_owner` / `vector_config_group` — пользователь и группа владельца конфигов, по умолчанию текущий ansible пользователь.
- `vector_config` — словарь с конфигурацией Vector, рендерится в `vector.yml`.
- `vector_systemd_unit` (vars) — путь до unit-файла (по умолчанию `/usr/lib/systemd/system/vector.service`).
- `vector_service_name` (vars) — имя сервиса (`vector`).
- `vector_validate_command` (vars) — команда для валидации конфига.

## Dependencies

Нет.

## Example Playbook

```yaml
- hosts: vector
  roles:
    - role: vector
      vars:
        vector_version: "0.22.1"
        vector_config:
          sources:
            demo_logs:
              type: demo_logs
              format: syslog
          sinks:
            to_clickhouse:
              type: clickhouse
              inputs: [demo_logs]
              database: logs
              endpoint: http://clickhouse-01:8123
              table: vector_table
              compression: gzip
```

## License

MIT

## Author Information

golodniy
