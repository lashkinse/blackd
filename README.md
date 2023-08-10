# Blackd [![build](https://github.com/lashkinse/blackd/actions/workflows/blackd_workflow.yml/badge.svg)](https://github.com/lashkinse/blackd/actions/workflows/blackd_workflow.yml)

[Black](https://github.com/psf/black) - инструмент автоматического форматирования кода на Python, придерживающийся стандартов PEP8. Сборка и запуск Docker-образа позволят вам получить функционирующий сервис для форматирования кода на Python с использованием инструмента Black. Этот сервис будет доступен на порту 45484.

### Dockerhub

Этот образ доступен на [Dockerhub](https://hub.docker.com/r/lashkinse/blackd). Вы можете запустить его с помощью следующей команды::

```
docker run -d -p 45484:45484 --name blackd lashkinse/blackd:latest
```

### Dockerfile

В данном Dockerfile описано создание двух контейнеров: первый (builder) используется для сборки необходимых зависимостей, второй - окончательный образ, который содержит только необходимые компоненты и запускает сервис blackd, доступный на порту 45484. Такой подход позволяет уменьшить размер контейнера до приблизительно 160 МБ.

```
FROM python:3.11-slim AS builder
RUN apt update && apt install -y git build-essential \
    && pip install --upgrade pip setuptools wheel black[d]

FROM python:3.11-slim
COPY --from=builder /usr/local/ /usr/local/
EXPOSE 45484
ENTRYPOINT ["blackd", "--bind-host", "0.0.0.0", "--bind-port", "45484"]
```

- `FROM python:3.11-slim AS builder`
  Здесь определяется начальный образ контейнера, на базе которого будет происходить сборка. В данном случае, это образ Python версии 3.11 с минимальным набором компонентов (slim). Он будет использован для сборки промежуточного образа (builder).

- `RUN apt update && apt install -y git build-essential \`
  `&& pip install --upgrade pip setuptools wheel black[d]`
  Этот шаг в инструкции RUN обновляет список доступных пакетов в системе через команду apt update, а затем устанавливает несколько необходимых пакетов, включая git и build-essential (набор инструментов для сборки программ). Далее происходит обновление pip, установка setuptools и wheel для управления зависимостями Python, а также установка инструмента Black с его дополнительными зависимостями.

- `FROM python:3.11-slim`
  Вторая часть кода начинает определение нового образа на основе python:3.11-slim. Это будет окончательный образ, который будет содержать только необходимые компоненты для запуска сервиса.

- `COPY --from=builder /usr/local/ /usr/local/`
  Эта инструкция копирует содержимое директории /usr/local/ из промежуточного образа (builder) в директорию /usr/local/ нового образа. Таким образом, она переносит все установленные зависимости и инструменты из сборочного образа в конечный образ.

- `EXPOSE 45484`
  Эта инструкция копирует содержимое директории /usr/local/ из промежуточного образа (builder) в директорию /usr/local/ нового образа. Таким образом, она переносит все установленные зависимости и инструменты из сборочного образа в конечный образ.

- `ENTRYPOINT ["blackd", "--bind-host", "0.0.0.0", "--bind-port", "45484"]`
  Это указывает Docker, какую команду следует выполнить при запуске контейнера. В данном случае, это команда blackd с опциями --bind-host 0.0.0.0 (привязка к адресу 0.0.0.0, что означает, что контейнер будет доступен с любого IP) и --bind-port 45484 (привязка к порту 45484).

Для сборки контейнера используйте следующую команду:

```
docker build . -t blackd:latest
```

### Pycharm

Установите плагин [BlackConnect](https://plugins.jetbrains.com/plugin/14321-blackconnect) и настройте конфигурацию, как показано на скриншоте ниже.

![BlackConnect config](https://raw.githubusercontent.com/lashkinse/blackd/master/screenshots/BlackConnect.png "BlackConnect config")

Теперь ваш код будет форматироваться каждый раз при его сохранении!
