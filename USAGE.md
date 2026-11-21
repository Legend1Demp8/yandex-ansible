# Развёртывание и настройка Redis с помощью Ansible

Идемпотентная Ansible-роль и плейбук для автоматической установки, настройки и запуска сервера Redis на системах семейства **Debian** (Debian / Ubuntu) с поддержкой разделения окружений (`dev` и `prod`) и шифрованием секретов через **Ansible Vault**.

## Структура проекта

```text
.
├── inventories/                  # Инвентари и переменные для разных окружений
│   ├── dev/                      # Окружение разработки (dev)
│   │   ├── hosts.ini             # Список целевых хостов dev
│   │   └── group_vars/
│   │       └── all/
│   │           ├── vars.yml      # Параметры конфигурации Redis для dev
│   │           └── vault.yml     # Зашифрованный пароль Redis для dev (Ansible Vault)
│   └── prod/                     # Продакшн-окружение (prod)
│       ├── hosts.ini             # Список целевых хостов prod
│       └── group_vars/
│           └── all/
│               ├── vars.yml      # Параметры конфигурации Redis для prod
│               └── vault.yml     # Зашифрованный пароль Redis для prod (Ansible Vault)
├── roles/
│   └── redis/                    # Роль для установки и настройки Redis
│       ├── defaults/
│       │   └── main.yml          # Переменные роли по умолчанию
│       ├── handlers/
│       │   └── main.yml          # Обработчики (перезапуск сервиса при изменении конфига)
│       ├── tasks/
│       │   └── main.yml          # Основные задачи установки, подготовки каталогов и запуска
│       └── templates/
│           └── redis.conf.j2     # Jinja2-шаблон конфигурационного файла /etc/redis/redis.conf
├── playbook.yaml                 # Основной плейбук для запуска роли
├── README.md                     # Документация выполнения задания
└── USAGE.md                      # Документация по проекту
```

## Требования

* **Управляющий узел (Control Node):** установленный Ansible (версии 2.12+).
* **Целевые серверы (Managed Nodes):** ОС семейства Debian (Debian / Ubuntu), доступ по SSH с правами `sudo` (`become: true`).

## Переменные

Все параметры настраиваются через `group_vars` соответствующего окружения (`inventories/<env>/group_vars/all/vars.yml` и `vault.yml`).

| Переменная | Значение по умолчанию (`defaults`) | Значение `dev` | Значение `prod` | Описание |
| :--- | :--- | :--- | :--- | :--- |
| `redis_port` | `6379` | `6380` | `6379` | Порт, на котором Redis принимает подключения |
| `redis_dir` | `/var/lib/redis` | `/mnt/redis` | `/var/lib/redis` | Рабочий каталог для хранения файлов БД на диске |
| `redis_databases` | `16` | `4` | `16` | Количество логических баз данных |
| `redis_appendonly` | `"yes"` | `"no"` | `"yes"` | Включение режима персистентности AOF (`yes` / `no`) |
| `vault_redis_requirepass` | *обязательная* | *в `vault.yml`* | *в `vault.yml`* | Пароль для аутентификации клиентов (хранится в зашифрованном виде) |
| `redis_requirepass` | `{{ vault_redis_requirepass }}` | — | — | Итоговая переменная пароля, передаваемая в шаблон `redis.conf` |

## Работа с секретами (Ansible Vault)

Пароль для подключения к Redis (`vault_redis_requirepass`) хранится в зашифрованных файлах `vault.yml` внутри каждого окружения. Мастер-пароль от Ansible Vault **не хранится в репозитории**.

### Создание или редактирование секретов

* Просмотреть содержимое зашифрованного файла:
  ```bash
  ansible-vault view inventories/dev/group_vars/all/vault.yml
  ```
* Отредактировать секреты окружения:
  ```bash
  ansible-vault edit inventories/prod/group_vars/all/vault.yml
  ```

## Запуск плейбука

Переключение между окружениями происходит указанием пути к нужному инвентарю через флаг `-i`. Сам файл `playbook.yaml` при этом не меняется.

### Способ 1. Интерактивный ввод пароля Vault
Ansible запросит мастер-пароль в терминале перед началом выполнения:

* **Запуск для `dev`:**
  ```bash
  ansible-playbook -i inventories/dev/hosts.ini playbook.yaml --ask-vault-pass
  ```
* **Запуск для `prod`:**
  ```bash
  ansible-playbook -i inventories/prod/hosts.ini playbook.yaml --ask-vault-pass
  ```

### Способ 2. Передача пароля через файл (удобно для автоматизации)
Поместите мастер-пароль от Vault в файл за пределами репозитория (например, `~/.vault_pass.txt` с правами `600`) и передайте его через `--vault-password-file`:

* **Запуск для `dev`:**
  ```bash
  ansible-playbook -i inventories/dev playbook.yaml --vault-password-file ~/.vault_pass.txt
  ```
* **Запуск для `prod`:**
  ```bash
  ansible-playbook -i inventories/prod playbook.yaml --vault-password-file ~/.vault_pass.txt
  ```

## Проверка работоспособности

После успешного выполнения плейбука проверить работу сервиса на целевом сервере можно командой:
```bash
redis-cli -p <порт> -a <пароль> ping
# Ожидаемый ответ: PONG
```
