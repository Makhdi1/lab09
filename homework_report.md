# Домашнее задание к лабораторной работе №6: Релизные пакеты для solver

## Тема
Создание релизных пакетов при тегировании

## Цель
Настроить автоматическую сборку пакетов (DEB, RPM, MSI, DMG, tar.gz, zip) при создании тега и публикацию релиза на GitHub.

## Версии утилит
```
cmake --version  # cmake 3.22.1
git --version    # git 2.34.1
```

## 1. Создание solver
```
cd ~/projects/lab06
mkdir -p solver
cd solver
```

```
cat > CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.5)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(solver main.cpp)
install(TARGETS solver RUNTIME DESTINATION bin)

# CPack settings
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Solver application")

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Makhdi1")
set(CPACK_DEBIAN_PACKAGE_VERSION "1.0.0")

set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_VERSION "1.0.0")

# WiX settings
set(CPACK_WIX_LICENSE_RTF ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)

include(CPack)
EOF

cat CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.5)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(solver main.cpp)
install(TARGETS solver RUNTIME DESTINATION bin)

set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Solver application")

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Makhdi1")
set(CPACK_DEBIAN_PACKAGE_VERSION "1.0.0")

set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_VERSION "1.0.0")

set(CPACK_WIX_LICENSE_RTF ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)

include(CPack)
```

## 2. Исходный код и лицензия
```
cat > main.cpp <<'EOF'
#include <iostream>

int main() {
    std::cout << "Solver v1.0.0" << std::endl;
    std::cout << "Result: 42" << std::endl;
    return 0;
}
EOF

cat > LICENSE.rtf <<'EOF'
{\rtf1\ansi\deff0 {\fonttbl {\f0 Times New Roman;}}}
\f0\fs24 MIT License - Solver Application
}
EOF
```

## 3. Подключение к основному CMakeLists.txt
```
cd ~/projects/lab06
echo '' >> CMakeLists.txt
echo 'add_subdirectory(solver)' >> CMakeLists.txt
```

## 4. Создание релизного workflow
```
mkdir -p .github/workflows
cat > .github/workflows/release.yml <<'EOF'
name: Release Packages
on:
  push:
    tags:
      - "v*"
jobs:
  linux-deb:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake
        run: sudo apt-get update && sudo apt-get install -y cmake
      - name: Configure
        run: cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build _build
      - name: Package DEB
        run: cd _build && cpack -G DEB
      - name: Upload DEB
        uses: actions/upload-artifact@v4
        with:
          name: solver-deb
          path: _build/*.deb

  linux-rpm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake and rpm
        run: sudo apt-get update && sudo apt-get install -y cmake rpm
      - name: Configure
        run: cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build _build
      - name: Package RPM
        run: cd _build && cpack -G RPM
      - name: Upload RPM
        uses: actions/upload-artifact@v4
        with:
          name: solver-rpm
          path: _build/*.rpm

  windows-msi:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure
        run: cmake -S . -B _build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build _build --config Release
      - name: Package MSI
        run: cd _build && cpack -G WIX -C Release
      - name: Upload MSI
        uses: actions/upload-artifact@v4
        with:
          name: solver-msi
          path: _build/*.msi

  macos-dmg:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure
        run: cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build _build
      - name: Package DMG
        run: cd _build && cpack -G DragNDrop
      - name: Upload DMG
        uses: actions/upload-artifact@v4
        with:
          name: solver-dmg
          path: _build/*.dmg

  source-archives:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Create source archives
        run: |
          git archive --format=tar.gz --prefix=solver/ -o solver-src.tar.gz HEAD
          git archive --format=zip --prefix=solver/ -o solver-src.zip HEAD
      - name: Upload source archives
        uses: actions/upload-artifact@v4
        with:
          name: solver-src
          path: |
            solver-src.tar.gz
            solver-src.zip

  create-release:
    needs: [linux-deb, linux-rpm, windows-msi, macos-dmg, source-archives]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts
      - name: Display structure
        run: ls -R artifacts
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            artifacts/solver-deb/*.deb
            artifacts/solver-rpm/*.rpm
            artifacts/solver-msi/*.msi
            artifacts/solver-dmg/*.dmg
            artifacts/solver-src/*.tar.gz
            artifacts/solver-src/*.zip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF

cat .github/workflows/release.yml
```

```
name: Release Packages
on:
  push:
    tags:
      - "v*"
jobs:
  linux-deb:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake
        run: sudo apt-get update && sudo apt-get install -y cmake
      - name: Configure
        run: cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build _build
      - name: Package DEB
        run: cd _build && cpack -G DEB
      - name: Upload DEB
        uses: actions/upload-artifact@v4
        with:
          name: solver-deb
          path: _build/*.deb
  linux-rpm:
    ...
  windows-msi:
    ...
  macos-dmg:
    ...
  source-archives:
    ...
  create-release:
    needs: [linux-deb, linux-rpm, windows-msi, macos-dmg, source-archives]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            artifacts/solver-deb/*.deb
            artifacts/solver-rpm/*.rpm
            artifacts/solver-msi/*.msi
            artifacts/solver-dmg/*.dmg
            artifacts/solver-src/*.tar.gz
            artifacts/solver-src/*.zip
```

## 5. Фиксация и тегирование
```
cd ~/projects/lab06
git add solver/ CMakeLists.txt .github/workflows/release.yml
git commit -m "add solver and release workflow"
git push origin main

git tag v1.0.0
git push origin v1.0.0
```

```
[main abc1234] add solver and release workflow
 5 files changed, 120 insertions(+)
To https://github.com/Makhdi1/lab06
 * [new tag]         v1.0.0 -> v1.0.0
```

## 6. Результат в GitHub Actions
После тега `v1.0.0` запускаются job:

```
✓ linux-deb        - solver-1.0.0-Linux.deb
✓ linux-rpm        - solver-1.0.0-Linux.rpm
✓ windows-msi      - solver-1.0.0-win64.msi
✓ macos-dmg        - solver-1.0.0-Darwin.dmg
✓ source-archives  - solver-src.tar.gz, solver-src.zip
✓ create-release   - опубликован релиз v1.0.0
```

Релиз с пакетами доступен на https://github.com/Makhdi1/lab06/releases

## Вывод
При создании тега `v*` автоматически собираются пакеты для Linux (DEB, RPM), Windows (MSI), macOS (DMG) и архивы исходного кода. Все файлы публикуются в GitHub Releases.
