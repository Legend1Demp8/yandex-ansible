## Задание 1
### Требования:
установка должна выполняться только на системах семейства Debian;
используйте Ansible-модули;
решение должно быть идемпотентным.
### Выполнение
Тэг задания 1 [v1.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v1.0.0) \
cat [playbook.yaml](https://github.com/Legend1Demp8/yandex-ansible/blob/v1.0.0/playbook.yaml)
```
---
- name: Установка и настройка Redis
  hosts: all
  become: true

  roles:
    - redis
```
cat [roles/redis/tasks/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v1.0.0/roles/redis/tasks/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# tasks file for redis
- name: Установка пакета Redis (только для семейства Debian)
  ansible.builtin.apt:
    name: redis-server
    state: present
    update_cache: true
    cache_valid_time: 3600
  when: ansible_facts['os_family'] == 'Debian'
```
### Идемпотентность
<img width="1637" height="535" alt="image" src="https://github.com/user-attachments/assets/c69da7f6-9dd7-4e12-82d8-451aab7ce35b" />

## Задание 2
### Требования:
используйте соответствующий Ansible-модуль для управления файловой системой;
каталог должен существовать до запуска сервиса.

### Выполнение
Тэг задания 2 [v2.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v2.0.0) \
cat [roles/redis/tasks/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v2.0.0/roles/redis/tasks/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# tasks file for redis
- name: Установка пакета Redis (только для семейства Debian)
  ansible.builtin.apt:
    name: redis-server
    state: present
    update_cache: true
    cache_valid_time: 3600
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание каталога для хранения данных Redis
  ansible.builtin.file:
    path: /var/lib/redis
    state: directory
    owner: redis
    group: redis
    mode: "0750"
  when: ansible_facts['os_family'] == 'Debian'
```

### Идемпотентность
Поскольку стандартная установка из репозитория уже автоматически настраивает папку то по сути нам не обязателен данный шаг
<img width="1634" height="289" alt="image" src="https://github.com/user-attachments/assets/7d6640c0-1396-4883-9c6e-8eb39ff4c841" />

## Задание 3
### Требования:
Создайте конфигурационный файл Redis через шаблон Ansible, чтобы все параметры можно было переопределять через переменные.
```
port <number>
dir <path>
databases <number>
requirepass <password>
appendonly <yes/no> 
```

### Выполнение
Тэг задания 3 [v3.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v3.0.0) \
cat [roles/redis/tasks/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v3.0.0/roles/redis/tasks/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# tasks file for redis
- name: Установка пакета Redis (только для семейства Debian)
  ansible.builtin.apt:
    name: redis-server
    state: present
    update_cache: true
    cache_valid_time: 3600
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание каталога для хранения данных Redis
  ansible.builtin.file:
    path: "{{ redis_dir }}"
    state: directory
    owner: redis
    group: redis
    mode: "0750"
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание конфигурационного файла Redis из шаблона
  ansible.builtin.template:
    src: redis.conf.j2
    dest: /etc/redis/redis.conf
    owner: redis
    group: redis
    mode: "0640"
  when: ansible_facts['os_family'] == 'Debian'
```
cat [roles/redis/defaults/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v3.0.0/roles/redis/defaults/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# defaults file for redis
redis_port: 6379
redis_dir: /var/lib/redis
redis_databases: 16
redis_requirepass: "qwerty123" # Пароль открыт но далее будет задание с Vault
redis_appendonly: "yes"
```

Template [roles/redis/templates/redis.conf.j2](https://github.com/Legend1Demp8/yandex-ansible/blob/v3.0.0/roles/redis/templates/redis.conf.j2)

### Идемпотентность
<img width="1634" height="689" alt="image" src="https://github.com/user-attachments/assets/6d69a5f1-aaa5-445b-8326-4881e1cd019e" />


## Задание 4
### Требования:
сервис Redis должен запускаться автоматически;
должен быть включён в автозагрузку;
Redis перезапускается только при изменении конфигурации.

### Выполнение
Тэг задания 4 [v4.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v4.0.0) \
cat [roles/redis/tasks/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v4.0.0/roles/redis/tasks/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# tasks file for redis
- name: Установка пакета Redis (только для семейства Debian)
  ansible.builtin.apt:
    name: redis-server
    state: present
    update_cache: true
    cache_valid_time: 3600
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание каталога для хранения данных Redis
  ansible.builtin.file:
    path: "{{ redis_dir }}"
    state: directory
    owner: redis
    group: redis
    mode: "0750"
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание конфигурационного файла Redis из шаблона
  ansible.builtin.template:
    src: redis.conf.j2
    dest: /etc/redis/redis.conf
    owner: redis
    group: redis
    mode: "0640"
  notify: Перезапуск сервиса Redis
  when: ansible_facts['os_family'] == 'Debian'

- name: Запуск и добавление сервиса Redis в автозагрузку
  ansible.builtin.service:
    name: redis-server
    state: started
    enabled: true
  when: ansible_facts['os_family'] == 'Debian'
```

cat [roles/redis/handlers/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v4.0.0/roles/redis/handlers/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# handlers file for redis
- name: Перезапуск сервиса Redis
  ansible.builtin.service:
    name: redis-server
    state: restarted
  when: ansible_facts['os_family'] == 'Debian'
```

### Идемпотентность
<img width="937" height="795" alt="image" src="https://github.com/user-attachments/assets/5584b926-37e9-484e-b819-2aec8a109f09" />


## Задание 5
### Требования:
пароль Redis не хранится в открытом виде;
используется Ansible Vault.

### Выполнение
Тэг задания 5 [v5.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v5.0.0) \
[group_vars/all/vault.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v5.0.0/group_vars/all/vault.yml)

cat [roles/redis/defaults/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v5.0.0/roles/redis/defaults/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# defaults file for redis
redis_port: 6379
redis_dir: /var/lib/redis
redis_databases: 16
redis_requirepass: "{{ vault_redis_requirepass }}"
redis_appendonly: "yes"
```

### Идемпотентность
<img width="934" height="821" alt="image" src="https://github.com/user-attachments/assets/3f04015d-86a6-4b73-a65f-9eab4a652cb0" />

## Задание 6
### Требования:
параметры должны отличаться между окружениями;
переключение окружения должно происходить через inventory;
плейбук не должен изменяться.

### Структура
```
.
├── inventories
│   ├── dev
│   │   ├── group_vars
│   │   │   └── all
│   │   │       ├── vars.yml
│   │   │       └── vault.yml
│   │   └── hosts.ini
│   └── prod
│       ├── group_vars
│       │   └── all
│       │       ├── vars.yml
│       │       └── vault.yml
│       └── hosts.ini
├── playbook.yaml
├── README.md
└── roles
    └── redis
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        │   └── redis.conf.j2
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml
```

### Выполнение
Тэг задания 6 [v6.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v6.0.0)
#### dev: 
[inventories/dev/hosts.ini](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/dev/hosts.ini) \
[inventories/dev/group_vars/all/vault.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/dev/group_vars/all/vault.yml) \
cat [inventories/dev/group_vars/all/vars.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/dev/group_vars/all/vars.yml)
```
---
redis_port: 6380
redis_databases: 4
redis_dir: /mnt/redis
redis_appendonly: "no"
```

#### prod: 
[inventories/prod/hosts.ini](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/prod/hosts.ini) \
[inventories/prod/group_vars/all/vault.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/prod/group_vars/all/vault.yml) \
cat [inventories/prod/group_vars/all/vars.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v6.0.0/inventories/prod/group_vars/all/vars.yml)
```
redis_port: 6379
redis_databases: 16
redis_dir: /var/lib/redis
redis_appendonly: "yes"
```

### Идемпотентность
<img width="951" height="827" alt="image" src="https://github.com/user-attachments/assets/bd00f956-014b-4ada-8462-7076590f3bc9" />

## Задание 7
### Требования:
повторный запуск плейбука не должен изменять систему;
Redis не должен переустанавливаться;
конфигурация не должна перезаписываться без изменений;
сервис Redis не должен перезапускаться без необходимости.

### Выполнение
Вот тут возникла сложность - оказывается при установке Redis в стандартном Systemd файле есть подлянка
cat /usr/lib/systemd/system/redis-server.service
```
ReadWritePaths=-/var/lib/redis
ReadWritePaths=-/var/log/redis
ReadWritePaths=-/var/run/redis
```
Она запрещает писать не в стандартные каталоги, и в /mnt/redis ему не записать базу и тд и тп \
Модернизируем нашу роль \
Тэг задания 7 [v7.0.0](https://github.com/Legend1Demp8/yandex-ansible/tree/v7.0.0) \
cat [roles/redis/tasks/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v7.0.0/roles/redis/tasks/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# tasks file for redis
- name: Установка пакета Redis (только для семейства Debian)
  ansible.builtin.apt:
    name: redis-server
    state: present
    update_cache: true
    cache_valid_time: 3600
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание каталога для хранения данных Redis
  ansible.builtin.file:
    path: "{{ redis_dir }}"
    state: directory
    owner: redis
    group: redis
    mode: "0750"
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание директории для systemd override
  ansible.builtin.file:
    path: /etc/systemd/system/redis-server.service.d
    state: directory
    mode: "0755"
  when: ansible_facts['os_family'] == 'Debian'

- name: Добавление redis_dir в разрешенные пути systemd (без затирания старых)
  ansible.builtin.lineinfile:
    path: /etc/systemd/system/redis-server.service.d/override.conf
    create: true
    mode: "0644"
    line: "{{ item }}"
  loop:
    - "[Service]"
    - "ReadWritePaths=-{{ redis_dir }}"
  notify:
    - Перезагрузка systemd
    - Перезапуск сервиса Redis
  when: ansible_facts['os_family'] == 'Debian'

- name: Создание конфигурационного файла Redis из шаблона
  ansible.builtin.template:
    src: redis.conf.j2
    dest: /etc/redis/redis.conf
    owner: redis
    group: redis
    mode: "0640"
  notify: Перезапуск сервиса Redis
  when: ansible_facts['os_family'] == 'Debian'

- name: Запуск и добавление сервиса Redis в автозагрузку
  ansible.builtin.service:
    name: redis-server
    state: started
    enabled: true
  when: ansible_facts['os_family'] == 'Debian'
```

cat [roles/redis/handlers/main.yml](https://github.com/Legend1Demp8/yandex-ansible/blob/v7.0.0/roles/redis/handlers/main.yml)
```
#SPDX-License-Identifier: MIT-0
---
# handlers file for redis
- name: Перезагрузка systemd
  ansible.builtin.systemd:
    daemon_reload: true

- name: Перезапуск сервиса Redis
  ansible.builtin.service:
    name: redis-server
    state: restarted
  when: ansible_facts['os_family'] == 'Debian'
```

### Идемпотентность
<img width="632" height="724" alt="image" src="https://github.com/user-attachments/assets/50a561bf-7553-4356-8575-cbbf654ff012" />

## Задание 8
### Требования:
Напишите README для вашего проекта — так, чтобы другой инженер мог развернуть Redis без дополнительных вопросов.
README должен содержать:
структуру каталогов проекта с кратким пояснением, что где лежит;
инструкцию по запуску: как запустить плейбук для каждого окружения, как передать пароль Ansible Vault;
описание переменных, которые можно переопределять.
### Решение:
[https://github.com/Legend1Demp8/yandex-ansible/USAGE.md](https://github.com/Legend1Demp8/yandex-ansible/blob/main/USAGE.md)





