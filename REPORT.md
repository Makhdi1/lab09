# Лабораторная работа IX

## GitHub Release

### Цель работы
Изучение процесса создания артефактов на примере GitHub Release.

### Выполнение

#### 1. Создание репозитория
Создан репозиторий lab09, склонирован из lab08.

#### 2. Настройка GPG
Сгенерирован GPG ключ RSA 3072, публичный ключ добавлен в GitHub.

#### 3. Сборка пакета
```
cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
cmake --build _build --target package
```
4. Создание тега
```
git tag -s v0.1.0.0
git push origin main --tags
```
5. Создание релиза
```
Релиз создан через github-release:

    Имя: libprint

    Версия: v0.1.0.0

    Артефакт: print-Linux-x86_64.tar.gz (474 kB)
```
Результаты
```
    Освоен процесс создания GitHub Release

    Настроена GPG подпись тегов

    Создан релиз с артефактом
```
#Ссылки
```
    GitHub Releases

    Репозиторий lab09

    Релиз v0.1.0.0' > REPORT.md && git add REPORT.md && git commit -m "Fix lab09 report" && git push origin main
```
