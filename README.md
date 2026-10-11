# Ansible SCM

Домашнее задание: Ansible и SCM.

Роль ставит Redis на Debian и Ubuntu, кладёт конфиг из шаблона и включает сервис.
Повторный запуск ничего не меняет, если переменные те же.
Redis перезапускается только когда меняется конфиг.

## Структура

- `playbook.yaml` — запуск роли `redis`. Окружение здесь не зашито.
- `roles/redis/tasks/main.yml` — установка пакета, каталог данных, конфиг, запуск сервиса.
- `roles/redis/handlers/main.yml` — перезапуск Redis, если конфиг изменился.
- `roles/redis/templates/redis.conf.j2` — шаблон конфига.
- `roles/redis/defaults/main.yml` — значения по умолчанию, их перекрывает inventory.
- `group_vars/all/vault.yml` — пароль Redis, зашифрован Ansible Vault.
- `inventories/dev` — хосты и переменные для dev.
- `inventories/prod` — хосты и переменные для prod.

## Запуск

Пароль Vault в репозиторий не кладётся. Его спрашивает Ansible при запуске.

Dev:

    ansible-playbook playbook.yaml -i inventories/dev --ask-vault-pass --ask-pass

Prod:

    ansible-playbook playbook.yaml -i inventories/prod --ask-vault-pass --ask-pass

`--ask-vault-pass` запрашивает пароль Vault.
`--ask-pass` запрашивает пароль SSH.
Плейбук один и тот же, меняется только inventory.

Если пароль Vault лежит в отдельном файле вне репозитория, вместо `--ask-vault-pass` можно передать `--vault-password-file /путь/к/файлу`.

## Переменные

Их задают в `inventories/<окружение>/group_vars/all.yml`.

| Переменная | Что значит | dev | prod |
|---|---|---|---|
| redis_port | порт | 6380 | 6379 |
| redis_databases | число баз | 4 | 16 |
| redis_dir | каталог данных | /mnt/redis | /var/lib/redis |
| redis_appendonly | запись на диск | no | yes |
| redis_password | пароль Redis | в Vault | в Vault |

`redis_appendonly` пишется в кавычках: `"yes"` или `"no"`.