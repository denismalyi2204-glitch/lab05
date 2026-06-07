## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**
## Report

Настраиваем окружение
```sh
export GITHUB_USERNAME=ваше_имя_пользователя
cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate
```

Копируем репозиторий и подключаем удаленный
```sh
git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05
cd projects/lab05
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab05
```

Вывод 
```sh
Cloning into 'projects/lab05'...
remote: Enumerating objects: 70, done.
remote: Counting objects: 100% (70/70), done.
remote: Compressing objects: 100% (44/44), done.
remote: Total 70 (delta 16), reused 66 (delta 12), pack-reused 0 (from 0)
Receiving objects: 100% (70/70), 9.77 KiB | 1.95 MiB/s, done.
Resolving deltas: 100% (16/16), done
```

Подключаем фреймворк для модульного тестирования Google Test как подмодуль, чтобы можно было писать и запускать тесты.
```sh
mkdir third-party
git submodule add https://github.com/google/googletest third-party/gtest
cd third-party/gtest && git checkout release-1.8.1 && cd ../..
git add third-party/gtest
git commit -m"added gtest framework"
```

Вывод
```sh
Cloning into '/home/ubumba64/denismalyi2204-glitch/workspace/projects/projects/lab05/third-party/gtest'...
remote: Enumerating objects: 28627, done.
remote: Counting objects: 100% (61/61), done.
remote: Compressing objects: 100% (46/46), done.
remote: Total 28627 (delta 32), reused 15 (delta 15), pack-reused 28566 (from 2)
Receiving objects: 100% (28627/28627), 13.74 MiB | 1.28 MiB/s, done.
Resolving deltas: 100% (21273/21273), done.
Note: switching to 'release-1.8.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings
[main b0abb62] added gtest framework
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 third-party/gtest
```

Настраиваем систему сборки CMake для компиляции и запуска модульных тестов
```sh
sed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\
option(BUILD_TESTS "Build tests" OFF)
' CMakeLists.txt
cat >> CMakeLists.txt <<EOF
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```

Создаем первый модульный тест для проверки функции вывода текста в файл с использованием Google Test.
```sh
mkdir tests
cat > tests/test1.cpp <<EOF
#include <print.hpp>
#include <gtest/gtest.h>
TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};
  print(text, out);
  out.close();
  std::string result;
  std::ifstream in{filepath};
  in >> result;
  EXPECT_EQ(result, text);
}
EOF
```

Компилируем проект, запускаем все тесты и выводим результат их выполнения
```sh
cmake -H. -B_build -DBUILD_TESTS=ON
cmake --build _build
cmake --build _build --target test
_build/check
cmake --build _build --target test -- ARGS=--verbose
```

Вывод
```sh
-- The C compiler identification is GNU 13.3.0
-- The CXX compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found GTest: /usr/lib/x86_64-linux-gnu/cmake/GTest/GTestConfig.cmake (found version "1.14.0")  
-- Configuring done (0.8s)
-- Generating done (0.0s)
-- Build files have been written to: /home/ubumba64/denismalyi2204-glitch/workspace/projects/lab05/_build
```
```sh
[ 12%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 25%] Linking CXX static library libprint.a
[ 25%] Built target print
[ 37%] Building CXX object CMakeFiles/example1.dir/examples/example1.cpp.o
[ 50%] Linking CXX executable example1
[ 50%] Built target example1
[ 62%] Building CXX object CMakeFiles/example2.dir/examples/example2.cpp.o
[ 75%] Linking CXX executable example2
[ 75%] Built target example2
[ 87%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[100%] Linking CXX executable check
[100%] Built target check
```
```sh
Running tests...
Test project /home/ubumba64/denismalyi2204-glitch/workspace/projects/lab05/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.01 sec
100% tests passed, 0 tests failed out of 1
Total Test time (real) =   0.01 sec
```
```sh
Running main() from /usr/src/googletest/googletest/src/gtest_main.cc
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (0 ms)
[----------] 1 test from Print (0 ms total)
[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.
```

Настраиваем непрерывную интеграцию (CI) на GitHub Actions, которая автоматически собирает проект и запускает все тесты при каждом изменении в репозитории
```sh
mkdir -p .github/workflows
cat > .github/workflows/ci.yml <<EOF
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
      with:
        submodules: recursive
    
    - name: Configure CMake
      run: cmake -H. -B_build -DBUILD_TESTS=ON
    
    - name: Build
      run: cmake --build _build
    
    - name: Run tests
      run: cmake --build _build --target test
    
    - name: Run tests verbose
      run: cmake --build _build --target test -- ARGS=--verbose
EOF
```

Обновляем ссылки и название проекта в README с 4-й на 5-ю лабу
```sh
sed -i 's/lab04/lab05/g' README.md
```

Вставляем визуальный индикатор, показывающий текущий статус сборки и тестов на странице репозитория
```sh
cat >> README.md <<EOF
## CI Status
[![CI](https://github.com/${GITHUB_USERNAME}/lab05/actions/workflows/ci.yml/badge.svg)](https://github.com/${GITHUB_USERNAME}/lab05/actions/workflows/ci.yml)
EOF
```

Сохраняем все настройки тестирования и непрерывной интеграции в репозиторий
```sh
git add .github/
git add tests
git add README.md
git add CMakeLists.txt
git add third-party/gtest
git add -p
git commit -m"added tests and GitHub Actions"
```

Вывод
```sh
No changes.
[main de88a4c] added tests and GitHub Actions
 5 files changed, 74 insertions(+), 82 deletions(-)
 create mode 100644 tests/test1.cpp
 delete mode 160000 third-party/gtest

```

 Публикуем проект на GitHub и создаем скриншот для отчета 
```sh
git push origin master
mkdir artifacts
sleep 20s && gnome-screenshot --file artifacts/screenshot.png
```

Вывод
```sh
Username for 'https://github.com': denismalyi2204-glitch
Password for 'https://denismalyi2204-glitch@github.com': 
Enumerating objects: 83, done.
Counting objects: 100% (83/83), done.
Delta compression using up to 4 threads
Compressing objects: 100% (49/49), done.
Writing objects: 100% (83/83), 11.73 KiB | 3.91 MiB/s, done.
Total 83 (delta 19), reused 68 (delta 16), pack-reused 0
remote: Resolving deltas: 100% (19/19), done.
To https://github.com/denismalyi2204-glitch/lab05
 * [new branch]      main -> main

```

## CI Status

[![CI](https://github.com/denismalyi2204-glitch/lab05/actions/workflows/ci.yml/badge.svg)](https://github.com/denismalyi2204-glitch/lab05/actions/workflows/ci.yml)
