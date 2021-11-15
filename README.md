# Настройка сервера Debian для работы с Django

В этом руководстве мы настроим чистый сервер Debian для проектов Python и Django. Настроим безопасное соединение SSH, установим из репозиториев Debian и из исходников все необходимые пакеты и подготовим их для рабочего сервера Debian Django.

## Создаём пользователя, устанавливаем необходимые пакеты, настраиваем SSH

Подключаемся через SSH к удаленному серверу Debian, обновляем репозитории и устанавливаем несколько начальных необходимых пакетов:

```
sudo apt update && sudo apt upgrade -y ; \
sudo apt-get install -y vim htop git curl wget unzip zip gcc build-essential make mosh tmux tree redis-server nginx libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev python3-lxml gnumeric libpq-dev libxml2-dev libxslt1-dev libjpeg-dev libfreetype6-dev libcurl4-openssl-dev supervisor libgdbm-dev libnss3-dev liblzma-dev ufw zsh
```

Cоздаём нового пользователя, пароль и добавляем его в группу `sudo`, чтобы он мог запускать процессы от имени суперпользователя:

```
adduser www 
usermod -aG sudo www
```
где `www` — это имя пользователя, которое мы будем использовать в дальнейшем.

Настраиваем SSH:

```
sudo vim /etc/ssh/sshd_config
    AllowUsers www
    PermitRootLogin no
    PasswordAuthentication no
```

Перезагружаем SSH-сервер:

```
sudo service ssh restart
```

ВАЖНО: перед выходом с сервера убедитесь, особенно если работали под `root` пользователем, что в файл `~/.ssh/authorized_keys` добавлен необходимый публичный ключ, иначе получите ошибку входа: `www@your_server: Permission denied (publickey).`, где `www` - это имя пользователя,  а `your_server` - ваш сервер.

## ZSH

Устанавливаем [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh):

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" ; \
chsh -s $(which zsh)
```

Настраиваем необходимые алиасы:

```
vim ~/.zshrc
    alias cls="clear"
```

## Устанавливаем Python3.10

Собираем python3.10 из исходников и устанавлеваем его в папку с префиксом `~/.python`:

```
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz ; \
tar xvf Python-3.10.* ; \
cd Python-3.10.0 ; \
mkdir /home/www/.python ; \
./configure --enable-optimizations --prefix=/home/www/.python ; \
make -j 2 ; \
sudo make altinstall
```

Проверяем содержимое переменной `$PATH`:

```
echo $PATH
```

Добавляем путь к интерпретатору в `$PATH`:

```
export PATH=$PATH:/home/www/.python/bin
```

Копируем и помещаем файлы в папку `~/.python/bin` (необязательно):

```
cd ../.python/bin
sudo cp pip3.10 pip
sudo cp python3.10 python
```

Сейчас python3.10 находится в `/home/www/.python/bin/python`. Обновляем `pip`:

```
sudo /home/www/.python/bin/python -m pip install --upgrade pip
```

Удаляем архив и папку:

```
sudo rm -rf Python-3.10.0.tgz Python-3.10.0/
```

Создаем директорию под проекты и переходим в неё:
```
cd
mkdir projects
cd projects
```
## Быстрая настройка чистого шаблона Django проекта
Теперь установим чистый шаблон Django проекта, с которым можно быстро начать разработку. В шаблон входит конфиг systemd, nginx, gunicorn.

Клонируем данный репозиторий:
```
git clone https://github.com/em0ji/debian-setup-for-django.git
```
Переименовываем папку `debian-setup-for-django/` в название своего проекта, например в `site/`:
```
mv debian-setup-for-django site
cd site
```
Настраиваем скрипт для утилиты `venv`:
```
sudo chmod ugo+x install.sh
```
И запускаем скрипт:

```bash
./install.sh
```
Установка представляет собой указание Python интерпретатора и названия домена (либо IP-адреса сервера). В конфиге Django заполняем настройки базы данных (`src/config/settings.py`).

Посмотреть статус gunicorn демона:

```bash
sudo systemctl status gunicorn
```

Логи gunicorn лежат в `gunicorn/access.log` и `gunicorn/error.log`.

После изменения systemd конфига его надо перечитать и затем перезапустить юнит:

```bash
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```
