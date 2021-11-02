## Для подключения нескольких репозиториев к одному серверу необходимо:
Допустим у меня есть репозиторий *rep1* и *rep2*.
1. Создать ssh ключи для каждого репозитория:
	1. Создаю ключ для репозиторий *rep1*: `ssh-keygen -o -t rsa -C "rep1"`, при указании директории для сохранения, указываю `/home/username/.ssh/id_rsa.rep1`
	2.  Аналогично для второго репозитория
2. Указать где храняться ключи:
	1. Запустить ssh-agent `eval $(ssh-agent -s)`
	2. `ssh-add ~/.ssh/id_rsa.rep1`
	3. `ssh-add ~/.ssh/id_rsa.rep2`
3.  Указать свой ssh ключ для каждого из репозиториев. 
	1. `sudo nano ~/.ssh/config`
	2. После открыватия нужно указать следующие данные:
		`Host rep1`
		`HostName github.com`
		`User git`
		`IdentityFile=~/.ssh/id_rsa.rep1`
		
		`Host rep2`
		`HostName github.com`
		`User git`
		`IdentityFile=~/.ssh/id_rsa.rep2`
4. Добавить созданные ssh ключи в deploy keys к  каждому из репозиториев.
5. Перейти в директорию, в которую будет или уже лежит репозиторий.
6. Если клонируем репозиторий:
	1. `git clone rep1:your_usename/rep1.git`
 	2. `git clone rep2:your_usename/rep2.git`
7. Если репозитории, которые подключены по http, но нужно подкючить по ssh.
	1.  `git remote set-url origin rep1:your_usename/rep1.git`
	2.  `git remote set-url origin rep2:your_usename/rep2.git`

Всё, теперь у вас есть несколько репозиториев на одном сервере, которые подключены по ssh
