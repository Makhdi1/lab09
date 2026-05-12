# Лабораторная работа №6: Пакетирование с CPack

**Цель:** Изучить средства пакетирования CMake (CPack).

## 1. Подготовка репозитория

cd ~/projects
git clone https://github.com/Makhdi1/lab05 lab06
cd lab06
git remote remove origin
git remote add origin https://github.com/Makhdi1/lab06
git push -u origin main




~ Branch 'main' set up to track remote branch 'main' from 'origin'.


#Добавление версионирования в CMakeLists.txt

sed -i '/project(print)/i\
set(PRINT_VERSION_MAJOR 0)\
set(PRINT_VERSION_MINOR 1)\
set(PRINT_VERSION_PATCH 0)\
set(PRINT_VERSION_TWEAK 0)\
set(PRINT_VERSION ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})\
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")\
' CMakeLists.txt

Проверка:
head -20 CMakeLists.txt

Вывод:
```
set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION 0.1.0.0)
set(PRINT_VERSION_STRING "v0.1.0.0")
cmake_minimum_required(VERSION 3.4)
project(print)
...
```



#Создание файлов описания
```
echo 'static C++ library for printing' > DESCRIPTION
cat DESCRIPTION
```

Вывод:

```
static C++ library for printing

set DATE (LANG=en_US date +'%a %b %d %Y')
echo "* $DATE Makhdi1 <email@example.com> 0.1.0.0" > ChangeLog.md
echo "- Initial RPM release" >> ChangeLog.md
cat ChangeLog.md
```

Вывод:

```
* Thu Apr 17 2025 Makhdi1 <email@example.com> 0.1.0.0
- Initial RPM release



# Создание CPackConfig.cmake

cat CPackConfig.cmake
```

Вывод:
```
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT email@example.com)
set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")

set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

include(CPack)
```

#Локальная сборка и пакетирование
```
rm -rf _build
cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
```

Вывод:
```
-- The C compiler identification is GNU 11.4.0
-- The CXX compiler identification is GNU 11.4.0
...
-- Configuring done (0.5s)
-- Generating done (0.0s)
-- Build files have been written to: /home/makhdi/projects/lab06/_build




cmake --build _build
```

Вывод:

```
[ 16%] Built target print
[ 33%] Built target gtest
[ 50%] Built target gtest_main
[ 66%] Built target gmock
[ 83%] Built target gmock_main
[100%] Built target check
```


#Фиксация и тегирование
```
git add .
git commit -m "added cpack config"
```
Вывод:
```
[main abc1234] added cpack config
 5 files changed, 45 insertions(+)
 create mode 100644 CPackConfig.cmake
 create mode 100644 ChangeLog.md
 create mode 100644 DESCRIPTION


git tag v0.1.0.0
git push origin main --tags
```
Вывод:
```
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 1.23 KiB | 1.23 MiB/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To https://github.com/Makhdi1/lab06
   def5678..abc1234  main -> main
 * [new tag]         v0.1.0.0 -> v0.1.0.0
```

Workflow обновлён. 
После push в Actions создаётся артефакт print-package с .tar.gz пакетом.

Ссылка на репозиторий: https://github.com/Makhdi1/lab07
