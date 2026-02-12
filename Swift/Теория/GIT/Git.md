### 1. Что такое Git (коротко и по делу)

Git — это **распределённая** система контроля версий (Distributed Version Control System, DVCS).

Ключевые отличия от централизованных систем (SVN, Perforce):

| Характеристика          | Git (распределённый)                     | SVN (централизованный)                  |
|-------------------------|------------------------------------------|------------------------------------------|
| Где хранится история    | У каждого разработчика полностью         | Только на сервере                        |
| Работа без интернета    | Полноценная (commit, branch, merge)      | Ограниченная (только локальные изменения)|
| Скорость                | Очень высокая (локальные операции)       | Зависит от сервера                       |
| Ветвление               | Дешёвое и быстрое                        | Дорогое и медленное                      |
| Резервная копия         | Каждый клон — полноценный бэкап          | Только сервер — точка отказа             |

### 2. Основная схема работы Git (локальный и удалённый репозиторий)

```mermaid
flowchart TD
    A[Рабочая копия<br>Working Directory] -->|git add| B[Staging Area<br>Index / Stage]
    B -->|git commit| C[Локальный репозиторий<br>.git / Local Repo]
    C -->|git push| D[Удалённый репозиторий<br>Remote Repo<br>GitHub / GitLab / Bitbucket]
    D -->|git fetch| C
    D -->|git pull = fetch + merge| C
    C -->|git checkout| A
    C -->|git switch| A
    C -->|git reset --hard| A
```

### 3. Основные области Git и команды

| Область               | Что хранит                              | Основные команды                              | Схема перехода |
|-----------------------|-----------------------------------------|-----------------------------------------------|----------------|
| **Working Directory** | Текущие файлы на диске                  | `git status`, `git add`, `git rm`, `git mv`   | → Staging      |
| **Staging Area**      | Файлы, подготовленные к коммиту         | `git add`, `git rm --cached`, `git restore --staged` | → Commit       |
| **Local Repository**  | История коммитов, ветки, теги           | `git commit`, `git branch`, `git tag`, `git log` | → Remote       |
| **Remote Repository** | Общая история (GitHub/GitLab/etc.)      | `git push`, `git pull`, `git fetch`, `git clone` | ← Local        |

### 4. Жизненный цикл файла в Git (схема состояний)

```mermaid
stateDiagram-v2
    [*] --> Untracked : новый файл
    Untracked --> Staged : git add
    Staged --> Committed : git commit
    Committed --> Modified : изменили файл
    Modified --> Staged : git add
    Modified --> Unmodified : git restore
    Committed --> Deleted : git rm
    Deleted --> [*]
```

### 5. Ветвление в Git — самая мощная фича

#### Классическая модель веток (Git Flow / GitHub Flow)

```mermaid
gitGraph
   commit id: "Initial commit"
   branch develop
   checkout develop
   commit id: "Feature A"
   branch feature/login
   checkout feature/login
   commit id: "Login screen"
   commit id: "Add validation"
   checkout develop
   merge feature/login id: "Merge login"
   branch release/v1.0.0
   checkout release/v1.0.0
   commit id: "Version bump"
   checkout main
   merge release/v1.0.0 id: "Release v1.0.0"
   checkout develop
   commit id: "Hotfix"
```

#### Современный GitHub Flow (2026 — самый популярный)

```mermaid
gitGraph
   commit id: "main"
   branch feature/add-dark-mode
   checkout feature/add-dark-mode
   commit id: "Dark theme UI"
   commit id: "Fix contrast issues"
   checkout main
   commit id: "Other fixes"
   branch hotfix/fix-crash
   checkout hotfix/fix-crash
   commit id: "Fix crash on iOS 18"
   checkout main
   merge hotfix/fix-crash id: "Hotfix merged"
   checkout feature/add-dark-mode
   commit id: "Final polish"
   checkout main
   merge feature/add-dark-mode id: "PR merged"
```

### 6. Основные команды Git с примерами и схемами

#### Работа с ветками

```bash
git branch feature/login          # создать ветку
git checkout feature/login        # переключиться
# или одной командой:
git switch -c feature/login

git branch -d feature/login       # удалить ветку (локально)
git push origin --delete feature/login  # удалить на удалённом
```

Схема переключения веток:

```mermaid
graph LR
    A[main] -->|git switch feature/login| B[feature/login]
    B -->|git switch main| A
```

#### Слияние (merge)

```bash
git checkout main
git merge feature/login           # fast-forward или 3-way merge
```

```mermaid
gitGraph
   commit id: "main"
   branch feature
   checkout feature
   commit id: "feature work"
   checkout main
   commit id: "main work"
   merge feature id: "Merge feature into main"
```

#### Rebase вместо merge (линейная история)

```bash
git checkout feature/login
git rebase main                   # перенести коммиты feature поверх main
git checkout main
git merge feature/login           # fast-forward
```

```mermaid
gitGraph
   commit id: "main A"
   commit id: "main B"
   branch feature
   checkout feature
   commit id: "feature X"
   commit id: "feature Y"
   checkout main
   commit id: "main C"
   checkout feature
   rebase main id: "Rebased feature on top of main"
```

### 7. Работа с удалёнными репозиториями

```bash
git clone https://github.com/user/repo.git
git remote add origin https://github.com/user/repo.git
git push -u origin main
git fetch origin
git pull origin main
```

### 8. Полезные команды для повседневной работы (2026)

| Задача                                  | Команда                                      | Альтернатива (современная) |
|-----------------------------------------|----------------------------------------------|-----------------------------|
| Статус репозитория                      | `git status`                                 | —                           |
| История коммитов (красиво)              | `git log --oneline --graph --decorate`       | `git log --pretty=graph`    |
| Последние изменения                     | `git diff` / `git diff --staged`             | —                           |
| Отменить git add                        | `git restore --staged file.txt`              | `git reset HEAD file.txt`   |
| Отменить изменения в файле              | `git restore file.txt`                       | `git checkout -- file.txt`  |
| Создать и сразу переключиться на ветку  | `git switch -c feature/login`                | `git checkout -b ...`       |
| Удалить ветку локально и удалённо       | `git branch -d branch` + `git push --delete` | —                           |
| Переименовать ветку                     | `git branch -m old-name new-name`            | —                           |
| Отменить последний коммит (оставить изменения) | `git reset --soft HEAD~1`             | —                           |
| Отменить последний коммит полностью     | `git reset --hard HEAD~1`                    | Опасно!                     |

### 9. Лучшие практики Git 2026

- Одна ветка = одна задача (feature/..., bugfix/..., hotfix/...)
- Именование коммитов: Conventional Commits  
  `feat: добавить тёмную тему`  
  `fix: исправить краш при пустом списке`  
  `chore: обновить зависимости`
- Rebase для feature-веток перед merge (линейная история)
- Squash & [[merge and rebase#Вариант 1 — git merge (рекомендуется для общей ветки)|merge]] или [[merge and rebase#Вариант 2 — git rebase (рекомендуется только для личных веток)|rebase]] & Merge в [[GitHub]]/[[GitLab]]
- Protected branches (main, develop) — требовать PR и ревью
- Git hooks + Husky / lefthook — форматирование, линтинг
- .gitignore — обязательно для [[Xcode]], SwiftPM, Pods, DerivedData
- Git LFS — для больших бинарных файлов (изображения, модели ML)

**Короткий девиз 2026**:
> «Git — это не просто инструмент, это история твоего кода.  
> Делай маленькие коммиты, понятные ветки, линейную историю и пиши хорошие сообщения — и работать будет приятно даже через 5 лет.»
