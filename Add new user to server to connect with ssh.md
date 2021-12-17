# Добавление нового пользователя на сервер для возможности подключения по ssh

1. Пользователя должен сгенерировать ssh ключи:

    `ssh-keygen -t ed25519 -C "your_email@example.com"`

2. Скопировать и отправить ssh pub ключ.

   Linux: `cat ~/.ssh/id_ed25519.pub`

3. Добавить pub ssh ключ пользователя на сервер:

   1. Нужно добавить ключ в файл ~/.ssh/authorized_keys: `sudo nano ~/.ssh/authorized_keys`
