## Задание 1

### Требования:
установка должна выполняться только на системах семейства Debian;
используйте Ansible-модули;
решение должно быть идемпотентным.
### Выполнение
cat playbook.yaml
```
---
- name: Установка и настройка Redis
  hosts: all
  become: true

  roles:
    - redis
```
cat roles/redis/tasks/main.yml
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
<img width="1634" height="477" alt="image" src="https://github.com/user-attachments/assets/e7ff7f42-ab4a-49cd-a179-ce47be3326b3" />
