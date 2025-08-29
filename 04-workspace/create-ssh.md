# 🔐 Создание SSH RSA ключа в Windows

Краткое руководство по генерации SSH-ключа для подключения к GitHub, GitLab и другим сервисам.

## 1. Проверка существующих ключей

Откройте **Git Bash** (или командную строку) и выполните:

```bash
ls -al ~/.ssh
```

Ищите файлы:
*   `id_rsa` (приватный ключ)
*   `id_rsa.pub` (публичный ключ)

Если их нет — создаём новые.

## 2. Генерация нового SSH ключа

Выполните команду, подставив свой email:

```bash
ssh-keygen -t rsa -b 4096 -C "rad@mail.ru"
```

*   `-t rsa`: тип ключа (RSA)
*   `-b 4096`: длина ключа (4096 бит — безопасно)
*   `-C`: комментарий (обычно ваш email)

## 3. Указание пути к файлу

Нажмите `Enter` на вопрос о сохранении файла, чтобы использовать путь по умолчанию:
> `C:\Users\YourUserName\.ssh\id_rsa`

## 4. Создание пароля (опционально, но рекомендуется)

```plaintext
> Enter passphrase (empty for no passphrase): [Введите пароль]
> Enter same passphrase again: [Повторите пароль]
```

Готово! Ключи созданы в папке `~/.ssh/`.

## 5. Добавление ключа в ssh-agent

Запустите ssh-agent в фоне:

```bash
eval "$(ssh-agent -s)"
```

Добавьте ваш приватный ключ в агент:

```bash
ssh-add ~/.ssh/id_rsa
```

## 6. Копирование публичного ключа

Выведите содержимое публичного ключа в терминал и скопируйте его:

```bash
cat ~/.ssh/id_rsa.pub
```

Или используйте `clip` для автоматического копирования в буфер обмена:

```bash
cat ~/.ssh/id_rsa.pub | clip
```

## 7. Добавление ключа на GitHub/GitLab

1.  Перейдите в **Settings** → **SSH and GPG keys** на GitHub/GitLab.
2.  Нажмите **New SSH key**.
3.  Вставьте скопированный ключ в поле.
4.  Сохраните.

## Проверка подключения

Проверьте, что всё работает:

```bash
ssh -T git@github.com
```

Вы должны увидеть:
> `Hi username! You've successfully authenticated...`

---

**Важно!** Никогда никому не передавайте файл `id_rsa` (ваш приватный ключ).

## Копирование ssh в Ubuntu
### Создаём папку .ssh в WSL (если её нет)
```bash
mkdir -p ~/.ssh
```
### Копируем ключи из Windows в WSL
Проверяем права доступа (обязательно!):
```bash
chmod 600 ~/.ssh/id_rsa         # Если используешь id_rsa
chmod 644 ~/.ssh/id_rsa.pub     # Публичный ключ
chmod 700 ~/.ssh                # Сама папка
```
## Проверяем подключение
```bash
ssh -T git@github.com
```
Если видишь:
```text
Hi radekkrab! You've successfully authenticated...
```
→ Всё работает! 🎉
## Настраиваем Git в WSL
```bash
git config --global user.name "radekkrab"
git config --global user.email "rad@mail.ru"
```
