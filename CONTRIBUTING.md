# Правила работы

## Задачи
- Каждая задача — Issue с ключом `SHOP-N` в заголовке. Брать задачу: назначить на себя, перевести в **In Progress**.
- Непонятно — спрашивай в комментарии к задаче (или в чате с лидом) **до** того, как писать код наугад.

## Ветки
- `main` защищена: прямой push запрещён, только через PR.
- Имя ветки: `feature/SHOP-12-order-pagination`, `fix/SHOP-15-n-plus-one`, `chore/SHOP-3-ci`.
- Перед PR подтяни свежий `main`: `git fetch && git rebase origin/main`.

## Коммиты — Conventional Commits
```
feat(order): add pagination to GET /orders
fix(inventory): release reservation on timeout
test(order): cover OrderService.cancel
chore(ci): cache maven dependencies
```
Типы: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`, `ci`. Скоуп — имя сервиса или области.

## Pull Request
- Один PR — одна задача. Заголовок: `SHOP-12: краткое описание`. В описании `Closes #12`.
- Заполни шаблон PR: что сделано, как проверить, что осталось.
- CI должен быть зелёным до запроса ревью.
- На каждый комментарий ревью — исправление или ответ. Резолвит обсуждение автор комментария.
- Вердикт лида — комментарием **APPROVED** или **CHANGES REQUESTED** (GitHub не даёт формально апрувить свой же аккаунт). Merge — только после APPROVED, способ — **Squash and merge**.

## Definition of Done
- [ ] Критерии приёмки из задачи выполнены
- [ ] `./mvnw verify` проходит локально и в CI
- [ ] Новый код покрыт тестами (что именно тестировать — указано в задаче)
- [ ] Нет секретов в коде, конфиги через env / профили
- [ ] Миграции БД — только через Liquibase, старые changeSet'ы не редактируются
- [ ] README / docs обновлены, если поменялся способ запуска или API
