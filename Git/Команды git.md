<br>


# Инициализация репозитория

```bash
# Установить email для всех локальных репозиториев (глобально)
git config --global user.email "test@user.ru"
```
<br>



```bash
# Инициализировать новый Git-репозиторий в текущей директории
git init
```
<br>


```bash
# Добавить файл README.md в индекс (стaging area) для будущего коммита
git add README.md
```
<br>


```bash
# Зафиксировать изменения в локальном репозитории с сообщением "first commit"
git commit -m "first commit"
```
<br>


```bash
# Переименовать текущую ветку в "main" (устаревший способ, лучше: git branch -m main)
git branch -M main
```
<br>


```bash
# Связать локальный репозиторий с удалённым репозиторием на GitHub
git remote add origin https://github.com/eisenheimus/test.git
```
<br>


```bash
# Отправить локальную ветку main в удалённый репозиторий и установить связь по умолчанию (-u)
git push -u origin main
```
<br>


```bash
# Проверить подключение к GitHub по SSH (должно вернуть приветствие)
ssh -T git@github.com
```
---
<br>


# Базовые операции
```bash
# Проверить статус изменений в рабочей директории
git status
```
<br>


```bash
# Добавить ВСЕ изменённые и новые файлы в индекс
git add .
```
<br>


```bash
# Отменить добавление файла в индекс (но сохранить изменения в файле)
git restore --staged filename
```
<br>


```bash
# Посмотреть историю коммитов с красивым форматированием
git log --oneline --graph --all
```
<br>


```bash
# Посмотреть изменения между текущим состоянием и последним коммитом
git diff
```
<br>


```bash
# Посмотреть изменения, уже добавленные в индекс
git diff --staged
```
---
<br>


# Ветвление и слияние
```bash
# Создать новую ветку и переключиться на неё
git checkout -b feature/login
```
<br>


```bash
# Переключиться на существующую ветку
git checkout main
```
<br>


```bash
# Обновить локальную ветку из удалённого репозитория
git pull origin main
```
<br>


```bash
# Слить изменения из ветки feature в текущую ветку
git merge feature/login
```
<br>


```bash
# Удалить локальную ветку (только если она уже слита)
git branch -d feature/login
```
<br>


```bash
# Удалить ветку принудительно (если не слита)
git branch -D feature/login
```
---
<br>


# Работа с удалённым репозиторием
```bash
# Скачать изменения из удалённого репозитория без слияния
git fetch origin
```
<br>


```bash
# Отправить локальные коммиты в удалённый репозиторий
git push origin feature/login
```
<br>


```bash
# Удалить ветку из удалённого репозитория
git push origin --delete feature/login
```
<br>


```bash
# Посмотреть все удалённые репозитории
git remote -v
```
---
<br>


#  Отмена изменений
```bash
# Отменить изменения в рабочей директории (ВНИМАНИЕ: изменения будут потеряны!)
git restore filename
```
<br>


```bash
# Отменить последний коммит, но сохранить изменения в индексе
git reset --soft HEAD~1
```
<br>


```bash
# Отменить последний коммит и вернуть изменения в рабочую директорию
git reset --mixed HEAD~1
```
<br>


```bash
# Полностью удалить последний коммит и все его изменения (осторожно!)
git reset --hard HEAD~1
```
<br>


```bash
# Создать обратный коммит (отменяющий изменения предыдущего коммита)
git revert HEAD
```
---
<br>

