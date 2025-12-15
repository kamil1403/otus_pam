<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Tux.svg/1200px-Tux.svg.png" alt="Linux Logo" width="10%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__pam-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-15.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Запретить всем пользователям, кроме группы `admin`, логиниться в выходные дни (Суббота, Воскресенье).
2. Настроить проверку через PAM `pam_exec`.

### ✅ Результат
- [x] Созданы пользователи и группы.
- [x] Скрипт валидации времени входа.
- [x] PAM настроен.
- [x] Тесты пройдены успешно. Результат см. на скриншотах:
  - 🖼️ [Доступ запрещен](https://github.com/kamil1403/otus_pam/blob/main/screenshots/otus_pam_1.png)
  - 🖼️ [Доступ разрешен](https://github.com/kamil1403/otus_pam/blob/main/screenshots/otus_pam_2.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Подготовка пользователей](#one)
- [🧰 Шаг 2 - Скрипт проверки](#two)
- [🧰 Шаг 3 - Настройка PAM](#three)
- [🧰 Шаг 4 - Тестирование](#four)

---

<a id="one"></a>
## 🧰 Шаг 1 - Подготовка пользователей
Создание тестовых пользователей (`otus`, `otusadm`) и группы `admin`.
```bash
# Создание пользователей
useradd -m -s /bin/bash otus
useradd -m -s /bin/bash otusadm
# Установка паролей
echo "otus:Otus2022!" | chpasswd
echo "otusadm:Otus2022!" | chpasswd
# Настройка группы admin
groupadd -f admin
usermod -aG admin otusadm
```

<a id="two"></a>
🧰 Шаг 2 - Скрипт проверки
Файл /usr/local/bin/login.sh. Проверяет день недели и принадлежность к группе.
```bash
#!/bin/bash
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
  if getent group admin | grep -qw "$PAM_USER"; then
        exit 0 # Разрешить
      else
        exit 1 # Запретить
    fi
  else
    exit 0 # Будни
fi
```

<a id="three"></a>
🧰 Шаг 3 - Настройка PAM
Редактирование /etc/pam.d/sshd. Добавление модуля pam_exec.
```bash
auth required pam_exec.so debug /usr/local/bin/login.sh
```

<a id="four"></a>
🧰 Шаг 4 - Тестирование
Проверка работы блокировки/
```bash
# 1. Смена даты на сервере (Суббота)
sudo date -s "2024-08-24 12:00:00"
# 2. Попытка входа обычным пользователем (результат: доступ запрещен)
ssh otus@192.168.1.21

# 3. Попытка входа администратором (результат: доступ разрешен)
ssh otusadm@192.168.1.21
