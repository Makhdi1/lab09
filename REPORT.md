# Лабораторная работа 8 (ypa)

## Изучение систем автоматизации развёртывания и управления приложениями на примере Docker

### Цель работы
Изучение систем автоматизации развёртывания и управления приложениями на примере Docker.

### Выполнение работы

#### 1. Настройка окружения
Создан публичный репозиторий lab08 на GitHub. Выполнено клонирование lab07 как основы:
```bash
cd /home/makhdi/Makhdi1/workspace
git clone https://github.com/Makhdi1/lab07 lab08
cd lab08
git submodule update --init
git remote remove origin
git remote add origin https://github.com/Makhdi1/lab08.git
```

#### 2. Создание Dockerfile
Создан Dockerfile для сборки образа приложения print:
```dockerfile
FROM ubuntu:18.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN sed -i "/hunter_add_package(print)/d" CMakeLists.txt

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install -DBUILD_TESTS=OFF
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/logs/log.txt

VOLUME /home/logs

WORKDIR _install/bin

ENTRYPOINT ./demo
```

#### 3. Сборка Docker образа
```bash
docker build -t logger .
```
Образ успешно собран, размер 353MB.

#### 4. Запуск контейнера
```bash
mkdir -p logs
docker run -it -v "$(pwd)/logs/:/home/logs/" logger
```
Введены тестовые данные: text1, text2, text3. Контейнер записал логи в примонтированный том.

#### 5. Просмотр информации об образе
```bash
docker images
docker inspect logger
```

#### 6. Просмотр логов
```bash
cat logs/log.txt
```
Логи успешно записаны в файл на хостовой машине через примонтированный volume.

#### 7. Настройка Travis CI
Создан файл `.travis.yml`:
```yaml
language: cpp

services:
  - docker

script:
  - docker build -t logger .
```

#### 8. Обновление README.md
Заменены ссылки lab07 на lab08:
```bash
sed -i "s/lab07/lab08/g" README.md
```

#### 9. Публикация
```bash
git add Dockerfile .travis.yml
git commit -m "adding Dockerfile"
git push origin main
```

### Проблемы и их решение

1. **Ошибка hunter_add_package(print)**: Внутри Docker-образа Hunter не находил версию пакета print 0.1.0.0. Решение: удалить строку `hunter_add_package(print)` из CMakeLists.txt перед сборкой в Dockerfile.

2. **Ветка master vs main**: Репозиторий использует ветку main, а не master. Решение: использовать `git push origin main`.

### Результаты
- Создан Dockerfile для приложения print
- Образ logger успешно собран (353MB)
- Контейнер запущен и протестирован с записью логов в volume
- Настроен Travis CI для автоматической сборки образа
- Логи контейнера успешно сохраняются на хостовой машине

### Ссылки
- [Docker Get Started](https://docs.docker.com/get-started/)
- [Docker Hub](https://hub.docker.com/)

