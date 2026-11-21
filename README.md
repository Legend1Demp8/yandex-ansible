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
