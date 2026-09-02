# product-factory (набор плагинов)

Переносимая копия пайплайна доставки продукта из проекта `portfolio`: 16 скилов + 5 агентов, которые
проводят продукт от идеи в одну строку до проверенной, протестированной и (опционально) задеплоенной
сдачи. Собрано из `.claude/skills/` и `.claude/agents/` проекта `portfolio` (`E:\Dev\_AI_\portfolio`),
синхронизируется вручную — теперь пакет живёт отдельно (вынесен из того репозитория 2026-08-28), это
не live-симлинк.

Находится в `E:\Dev\_AI_\skills\product-factory` и на GitHub как
[Dmkdok/product-factory](https://github.com/Dmkdok/product-factory) (перенесён с Desktop и переведён
под контроль версий 2026-09-02), чтобы дальше развиваться самостоятельно, а не оставаться разовым
экспортом.

## Что внутри

```
.claude-plugin/plugin.json   — манифест (name: product-factory)
skills/                      — пайплайн (10): orchestrate-product, iterate-product,
                                discover-requirements, draft-product-spec, draft-tech-plan,
                                implement-product, test-product, review-product, deploy-product, pause
                                встроенные зависимости (6): coding-discipline, concise-mode,
                                secure-review, web-design-guidelines, frontend-design, ui-quality-audit
agents/                      — product-planner, architect, implementer, tester, reviewer
templates/                   — шаблоны BRIEF/SPEC/PLAN/TASKS/STATUS/DECISIONS/REVIEW/HANDOFF/RELEASE
CLAUDE.product-factory.md    — блок для вставки в CLAUDE.md нового проекта
SKILLS-GUIDE.md              — шпаргалка на русском: какой скил в какой момент, советы по управлению
                                сессией, подводные камни с именами. Написана для человека, не для Claude.
```

## Как попробовать в другом проекте

```bash
claude --plugin-dir /path/to/product-factory
```

`marketplace.json` для локального использования не нужен — `--plugin-dir` читает `plugin.json`
напрямую. Вызываемые скилы получают неймспейс `/product-factory:orchestrate-product` и т.д., чтобы не
конфликтовать с одноимёнными локальными скилами проекта.

**Разрешение имени плагина — защищено, но не проверено вживую.** Каждая внутренняя ссылка в пакете
(например, `orchestrate-product`, который говорит «читай скил `discover-requirements`») использует
короткое имя. `orchestrate-product/SKILL.md` и `iterate-product/SKILL.md` теперь содержат заметку для
агента — повторить попытку с префиксом `product-factory:`, если короткое имя не сработало, — так что
запуск через плагин в худшем случае стоит один лишний вызов инструмента, а не тихий пропуск. Это
защитная мера, а не подтверждённый тест — первый реальный запуск пакета во внешнем проекте всё ещё
должен это подтвердить и сообщить, если какая-то ссылка потребовала ручной правки.

## Как развернуть в новом проекте с нуля

1. Установить плагин (см. выше) либо просто скопировать `skills/` и `agents/` этой папки в `.claude/`
   нового проекта — больше ничего подтягивать не нужно, шесть скилов-зависимостей ниже уже лежат в
   `skills/`.
2. Вставить содержимое `CLAUDE.product-factory.md` в `CLAUDE.md` нового проекта.
3. Скопировать `templates/` в новый проект как `templates/product-factory/` — Phase 0 скила
   `orchestrate-product` сначала ищет шаблоны там и откатывается на обычный `templates/` только когда
   рабочая директория — сам корень пакета.
4. Один раз прогнать `fewer-permission-prompts` (и `update-config` для хуков/прав проекта), чтобы
   снизить трение от запросов на разрешение до того, как пайплайн начнёт генерировать вызовы
   инструментов в объёме.
5. Начать с `/orchestrate-product` (проект с нуля) или `/iterate-product` (в репозитории уже есть
   `docs/SPEC.md`).

## Зависимости

**Встроены в `skills/`, синхронизируются вручную с `~/.claude/skills/` на этой машине** — та же
ручная модель, что уже используется для остальных 10 скилов, отдельного шага установки нет:
`coding-discipline`, `concise-mode`, `secure-review`, `web-design-guidelines`, `frontend-design`,
`ui-quality-audit`. Phase 0 скилов `orchestrate-product` и `iterate-product` по-прежнему проверяют,
что они разрешаются, перед Phase 4/6 — дешёвая подстраховка на случай, если из пакета скопировали
только часть `skills/`, а не основная защита.

**Вообще не файлы — встроены в сам Claude Code, ничего устанавливать не нужно, доступны на любой
машине с CLI:** `code-review`, `simplify`. В более ранней версии этого документа они ошибочно были
указаны как «установить отдельно» — на диске нет `~/.claude/skills/code-review`, который можно было
бы скопировать, они идут вместе с бинарником `claude`.

## Отличие от копии в проекте portfolio

`skills/pause/SKILL.md` здесь **не** зашивает жёстко `docker compose run --rm tests`, как это делает
`portfolio`'s own `.claude/skills/pause/SKILL.md` — эта команда верна только для того одного
репозитория. Копия в этом пакете просит агента найти настоящую команду тестов в
`CLAUDE.md`/`package.json`/`Makefile` целевого проекта. Всё остальное — прямая копия.
