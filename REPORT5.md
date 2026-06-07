# Отчёт по лабораторной работе №5

## Выполнил: Артеменко Арина ИУ8-22
**Покрытие кода**: 100% (Coveralls.io)
---

## 1. Настройка репозитория

## 1. Настройка репозитория

`export GITHUB_USERNAME=gest475`  
`cd ~/${GITHUB_USERNAME}/workspace`  
`git clone https://github.com/${GITHUB_USERNAME}/lab004 projects/lab05`  
`cd projects/lab05`  
`git remote remove origin`  
`git remote add origin https://github.com/${GITHUB_USERNAME}/lab05`

Вывод:
```
Cloning into 'projects/lab05'...
remote: Total 72 (delta 28), reused 57 (delta 19)
Receiving objects: 100% (72/72), 17.36 KiB | 1.74 MiB/s, done.
```

## 2. Установка GTest и GMock

`sudo apt update`  
`sudo apt install -y libgtest-dev libgmock-dev`

**Вывод:**
```
libgtest-dev is already the newest version (1.14.0-1)
libgmock-dev is already the newest version (1.14.0-1)
```
## 3. Создание библиотеки banking

Созданы файлы:
- `banking/Account.h`
- `banking/Account.cpp`
- `banking/Transaction.h`
- `banking/Transaction.cpp`
- `banking/CMakeLists.txt`

## 4. Корневой `CMakeLists.txt`

`cat > CMakeLists.txt << ...` (файл с поддержкой C++14, тестов и coverage)

## 5. Тесты

Созданы:
- `tests/test_account.cpp` — 4 теста для `Account`
- `tests/test_transaction.cpp` — 7 тестов для `Transaction`
- `tests/test_transaction_mock.cpp` — 1 тест с `GMock`

## 6. Сборка и запуск тестов

`rm -rf _build`  
`cmake -H. -B_build -DBUILD_TESTS=ON`  
`cmake --build _build`  
`./_build/check`

**Вывод (итог):**
```
[==========] 12 tests from 3 test suites ran.
[ PASSED ] 12 tests.
```
## 7. Настройка CI (GitHub Actions) и Coveralls

Создан файл `.github/workflows/ci.yml` со следующим содержимым (lcov + отправка в Coveralls).

**Команды для публикации:**
`git add .`  
`git commit -m "feat: add banking library with complete tests and coverage"`  
`git push origin main`

**Результат Coveralls.io:**
```
COVERALLS
GEST475 / LAB05
100% coverage
```
