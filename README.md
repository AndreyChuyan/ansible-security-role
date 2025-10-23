# 🔒 Ansible Security Baseline Role

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/ansible-%3E%3D2.9-blue.svg)](https://www.ansible.com/)
[![Ubuntu](https://img.shields.io/badge/ubuntu-20.04%20|%2022.04-orange.svg)](https://ubuntu.com/)

Комплексная Ansible-роль для базовой настройки безопасности Linux-серверов. Включает настройку firewall (UFW), SSH-hardening, Fail2Ban и управление пользователями.

## 👤 Автор

```bash
 _     _ 
  \___/    Ansible Security Baseline Role
 ( ^_^ )   
 /| o |\   🐞 Dbgops
 /|___|\   by Andrey Chuyan
 _/  \_    https://chuyana.ru
```

## 📋 Содержание

- [Возможности](#-возможности)
- [Требования](#-требования)
- [Установка](#-установка)
- [Быстрый старт](#-быстрый-старт)
- [Конфигурация](#-конфигурация)
- [Примеры использования](#-примеры-использования)
- [Переменные](#-переменные)
- [Лицензия](#-лицензия)

## ✨ Возможности

- 🔥 **UFW Firewall** — автоматическая настройка правил входящих/исходящих соединений
- 🔐 **SSH Hardening** — усиление безопасности SSH (отключение root, настройка портов, ограничение пользователей)
- 🛡️ **Fail2Ban** — защита от bruteforce-атак с гибкой настройкой jail'ов
- 👥 **Управление пользователями** — контроль доступа к системе
- 📝 **MOTD** — информативное приветствие при входе в систему

## 📦 Требования

- **Ansible**: >= 2.9
- **ОС**: Ubuntu 20.04 / 22.04, Debian 10/11
- **Права**: роль требует sudo/root доступа
- **Python**: >= 3.6 на управляемых хостах

## 🚀 Установка

### Через ansible-galaxy (рекомендуется)

```bash
ansible-galaxy install git+https://github.com/AndreyChuyan/ansible-security-role.git
```

### Через requirements.yml

```yaml
# requirements.yml
roles:
  - name: ansible-security-baseline
    src: https://github.com/AndreyChuyan/ansible-security-role.git
    version: main
```

```bash
ansible-galaxy install -r requirements.yml
```

### Клонирование репозитория

```bash
git clone https://github.com/AndreyChuyan/ansible-security-role.git roles/ansible-security-baseline
```

## ⚡ Быстрый старт

### 1. Минимальный playbook

```yaml
# playbook.yml
---
- hosts: all
  become: yes
  roles:
    - ansible-security-baseline
```

```bash
ansible-playbook -i inventory playbook.yml
```

### 2. С базовой кастомизацией

```yaml
# playbook.yml
---
- hosts: webservers
  become: yes
  roles:
    - role: ansible-security-baseline
      vars:
        ssh:
          port: 2222
          allowed_users:
            - admin
            - deployer
        ufw:
          allowed_ports:
            - { port: 80, proto: tcp, comment: "HTTP" }
            - { port: 443, proto: tcp, comment: "HTTPS" }
            - { port: 2222, proto: tcp, comment: "SSH" }
```

## ⚙️ Конфигурация

### Структура роли

```
ansible-security-baseline/
├── defaults/
│   └── main.yml          # Переменные по умолчанию
├── handlers/
│   └── main.yml          # Обработчики (restart SSH, UFW и т.д.)
├── tasks/
│   ├── main.yml          # Основная точка входа
│   ├── ufw.yml           # Настройка firewall
│   ├── ssh.yml           # Hardening SSH
│   ├── fail2ban.yml      # Настройка Fail2Ban
│   └── users.yml         # Управление пользователями
├── templates/
│   ├── sshd_config.j2    # Шаблон конфига SSH
│   ├── jail.local.j2     # Шаблон Fail2Ban
│   └── motd-updates.j2   # Шаблон MOTD
└── README.md
```

## 🔧 Переменные

### UFW (Firewall)

```yaml
ufw:
  # Разрешенные входящие порты
  allowed_ports:
    - { port: 22, proto: tcp, comment: "SSH" }
    - { port: 80, proto: tcp, comment: "HTTP" }
    - { port: 443, proto: tcp, comment: "HTTPS" }
  
  # Сброс всех правил перед настройкой
  reset: false
  
  # Политика для исходящих соединений
  outgoing_policy: allow  # allow | deny | reject
  
  # Заблокированные исходящие порты (при outgoing_policy: allow)
  blocked_outgoing_ports:
    - { port: 25, proto: tcp, comment: "Block SMTP" }
  
  # Разрешенные исходящие порты (при outgoing_policy: deny)
  allowed_outgoing_ports:
    - { port: 80, proto: tcp, comment: "HTTP" }
    - { port: 443, proto: tcp, comment: "HTTPS" }
    - { port: 53, proto: udp, comment: "DNS" }
    - { port: 123, proto: udp, comment: "NTP" }
```

### SSH

```yaml
ssh:
  # SSH порт
  port: 22
  
  # Разрешенные пользователи (остальные будут заблокированы)
  allowed_users:
    - admin
    - deployer
```

**Автоматические настройки безопасности:**
- ❌ Отключен root логин
- ❌ Отключена аутентификация по паролю
- ✅ Только SSH-ключи
- ✅ Строгая проверка режимов файлов

### Пользователи

```yaml
users:
  # Пользователи с правом логина
  allowed_login:
    - admin
    - deployer
```

### Fail2Ban

```yaml
fail2ban:
  # Глобальные настройки
  global:
    bantime: "1h"           # Время блокировки
    findtime: "10m"         # Временное окно для подсчета попыток
    maxretry: 5             # Максимум попыток
    # ignoreip: "127.0.0.1/8 192.168.1.0/24"
    # destemail: "admin@example.com"
    # sender: "fail2ban@example.com"
    # action: "%(action_mwl)s"  # Действие при бане (с отправкой логов)

  # SSH jail
  ssh:
    enabled: true
    maxretry: 3
    bantime: "24h"
    findtime: "10m"

  # Дополнительные jail'ы
  custom_jails:
    - name: nginx-http-auth
      enabled: true
      filter: nginx-http-auth
      logpath: /var/log/nginx/error.log
      maxretry: 3
      bantime: "1h"
    
    - name: postfix-sasl
      enabled: true
      filter: postfix[mode=auth]
      logpath: /var/log/mail.log
      maxretry: 3
```

## 📚 Примеры использования

### Пример 1: Веб-сервер

```yaml
---
- hosts: webservers
  become: yes
  roles:
    - role: ansible-security-baseline
      vars:
        ufw:
          allowed_ports:
            - { port: 22, proto: tcp, comment: "SSH" }
            - { port: 80, proto: tcp, comment: "HTTP" }
            - { port: 443, proto: tcp, comment: "HTTPS" }
        
        fail2ban:
          global:
            bantime: "2h"
            maxretry: 3
          custom_jails:
            - name: nginx-http-auth
              enabled: true
              filter: nginx-http-auth
              logpath: /var/log/nginx/error.log
```

### Пример 2: Строгий firewall (deny-by-default)

```yaml
---
- hosts: production
  become: yes
  roles:
    - role: ansible-security-baseline
      vars:
        ufw:
          outgoing_policy: deny
          allowed_ports:
            - { port: 22, proto: tcp, comment: "SSH" }
          allowed_outgoing_ports:
            - { port: 80, proto: tcp, comment: "HTTP" }
            - { port: 443, proto: tcp, comment: "HTTPS" }
            - { port: 53, proto: udp, comment: "DNS" }
            - { port: 53, proto: tcp, comment: "DNS TCP" }
            - { port: 123, proto: udp, comment: "NTP" }
```

### Пример 3: Нестандартный SSH порт

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: ansible-security-baseline
      vars:
        ssh:
          port: 2222
          allowed_users:
            - admin
        ufw:
          allowed_ports:
            - { port: 2222, proto: tcp, comment: "SSH Custom" }
```

### Пример 4: Использование через group_vars

```yaml
# group_vars/webservers.yml
ufw:
  allowed_ports:
    - { port: 22, proto: tcp, comment: "SSH" }
    - { port: 80, proto: tcp, comment: "HTTP" }
    - { port: 443, proto: tcp, comment: "HTTPS" }

ssh:
  port: 22
  allowed_users:
    - webadmin
    - deployer

fail2ban:
  ssh:
    enabled: true
    maxretry: 3
    bantime: "24h"
```

```yaml
# playbook.yml
---
- hosts: webservers
  become: yes
  roles:
    - ansible-security-baseline
```

## 🧪 Тестирование

### Проверка синтаксиса

```bash
ansible-playbook playbook.yml --syntax-check
```

### Dry-run

```bash
ansible-playbook playbook.yml --check --diff
```

### Проверка после применения

```bash
# Статус UFW
sudo ufw status verbose

# Статус Fail2Ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# SSH конфигурация
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|allowusers'
```

## ❓ FAQ

<details>
<summary><b>Можно ли использовать роль на production без тестирования?</b></summary>

**Нет!** Всегда тестируйте на dev/staging окружении. Роль изменяет критичные настройки безопасности.
</details>

<details>
<summary><b>Как откатить изменения если что-то пошло не так?</b></summary>

1. Через консоль (IPMI/VNC) отключите UFW: `sudo ufw disable`
2. Восстановите оригинальный sshd_config: `sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config`
3. Перезапустите SSH: `sudo systemctl restart sshd`

Роль автоматически создает бэкапы: `/etc/ssh/sshd_config.bak`
</details>

<details>
<summary><b>Роль идемпотентна?</b></summary>

Да, роль полностью идемпотентна. Повторный запуск не изменит состояние системы если конфигурация не менялась.
</details>

<details>
<summary><b>Какие порты открыты по умолчанию?</b></summary>

По умолчанию открыт только SSH (порт 22). Все остальные порты нужно указывать явно в `ufw.allowed_ports`.
</details>

<details>
<summary><b>Роль работает с RedHat/CentOS?</b></summary>

Сейчас поддерживаются только Debian-based дистрибутивы (Ubuntu/Debian). Поддержка RHEL-based систем планируется.
</details>

## ⚠️ Важные замечания

1. **Бэкапы** — роль автоматически создает бэкапы конфигов (`.bak`)
2. **Тестирование** — обязательно тестируйте на dev-окружении перед production
3. **Доступ** — всегда имейте альтернативный способ доступа (консоль, IPMI)
4. **SSH порт** — при изменении порта не забудьте обновить правила UFW
5. **Fail2Ban логи** — следите за `/var/log/fail2ban.log` для отладки

## 🔒 Безопасность

Роль применяет следующие практики безопасности:

- ✅ Отключение root-логина через SSH
- ✅ Только аутентификация по SSH-ключам
- ✅ Ограничение списка пользователей для SSH
- ✅ Настройка firewall с минимальными правами
- ✅ Защита от bruteforce через Fail2Ban
- ✅ Автоматические обновления безопасности (опционально)

## 🤝 Участие в разработке

Приветствуются Pull Requests! Пожалуйста:

1. Форкните репозиторий
2. Создайте feature-ветку (`git checkout -b feature/AmazingFeature`)
3. Закоммитьте изменения (`git commit -m 'Add some AmazingFeature'`)
4. Запушьте в ветку (`git push origin feature/AmazingFeature`)
5. Откройте Pull Request

## 📝 Лицензия

Распространяется под лицензией MIT. См. `LICENSE` для подробностей.

## 🙏 Благодарности

- [Ansible Documentation](https://docs.ansible.com/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)
- [Fail2Ban Documentation](https://www.fail2ban.org/)

---

**Made with ❤️ by [Dbgops](https://chuyana.ru)**

⭐ Если проект оказался полезным, поставьте звездочку!
