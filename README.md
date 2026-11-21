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

