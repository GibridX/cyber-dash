# Техническая спецификация для команды разработки  
## Веб‑приложение — тренажёр слепой печати с геймификацией  
**Проект:** Киберрывок / Осень 2026  
**Тип продукта:** веб‑приложение, обучение + игра  
**Основной стек:** Front-end: HTML, CSS, JavaScript; Back-end: Python; БД: PostgreSQL  

---

## 1. Цель проекта

Необходимо создать веб‑приложение — тренажёр для обучения слепой печати с элементами геймификации.

Пользователь печатает появляющиеся слова, предложения, фрагменты кода или символы. Система отслеживает:

- скорость печати;
- точность;
- ошибки;
- комбо;
- опыт;
- уровни;
- достижения;
- лидерборд;
- прогресс по буквам и сочетаниям.

Главная продуктовая цель — сделать тренировку интересной, чтобы пользователь возвращался ежедневно.

---

## 2. Скоуп MVP

### Обязательно в MVP

1. Регистрация / авторизация пользователя.
2. Гостевой режим с локальным сохранением прогресса.
3. Режимы печати:
   - слова;
   - предложения;
   - код;
   - символы.
4. Переключение языка: русский / английский.
5. Отображение текста посимвольно.
6. Подсветка текущего символа.
7. Подсветка правильного / неправильного ввода.
8. Виртуальная клавиатура с подсказкой пальцев.
9. Обработка ошибок:
   - неправильный символ не засчитывается как прогресс;
   - ошибка фиксируется;
   - комбо сбрасывается, если нет активного бонуса.
10. Система очков.
11. Система комбо:
   - x1, x2, x3, x5.
12. Система XP и уровней.
13. Достижения.
14. Случайные бонусы во время тренировки.
15. Статистика сессии.
16. Общая статистика пользователя.
17. Детальный разбор сложных символов.
18. Графики прогресса.
19. Таблица лидеров.
20. Визуальные эффекты:
   - искры;
   - огонь;
   - комбо-счётчик;
   - всплывающие +XP;
   - уведомления о достижениях.

### Не обязательно в MVP, но желательно заложить архитектуру

- Социальные сети / OAuth.
- Пуш-уведомления.
- Мобильная виртуальная клавиатура.
- Кастомные тексты пользователя.
- Админ-панель.
- Достижения с прогресс-барами.
- Ежедневные серии / streak.
- Квесты и ежедневные задания.

---

## 3. Технологический стек

### Front-end

- HTML5.
- CSS3.
- Vanilla JavaScript ES2022+.
- Возможно использовать сборщик Vite, но без обязательного фреймворка.
- Chart.js или D3.js для графиков.
- LocalStorage / IndexedDB для гостевого прогресса.
- Fetch API для взаимодействия с backend.

### Back-end

- Python 3.11+.
- FastAPI.
- Pydantic v2.
- SQLAlchemy 2.0.
- Alembic для миграций.
- PostgreSQL 15+.
- Redis опционально для кэша, rate limiting, активных сессий.
- Celery / ARQ / встроенные фоновые задачи — опционально для достижений и лидерборда.
- Docker / docker-compose для разработки.

### Аутентификация

- Email + пароль.
- JWT access token.
- Refresh token.
- OAuth провайдеры: Google, Yandex, GitHub, VK — опционально.

---

# 4. Front-end: что нужно сделать

## 4.1 Структура приложения

Рекомендуемые страницы / роуты:

| Route | Назначение |
|---|---|
| `/` | Главная страница, быстрый старт, описание режима |
| `/login` | Вход |
| `/register` | Регистрация |
| `/recover` | Восстановление пароля, опционально |
| `/train` | Основной экран тренировки |
| `/stats` | Статистика пользователя |
| `/achievements` | Достижения |
| `/leaderboard` | Таблица лидеров |
| `/profile` | Профиль и настройки |
| `/settings` | Язык, раскладка, звук, эффекты |
| `/admin` | Опционально, управление словарями и достижениями |

---

## 4.2 HTML

### Требования

1. Семантическая разметка:
   - `<main>`, `<header>`, `<nav>`, `<section>`, `<article>`, `<footer>`.
2. Доступность:
   - `aria-label` для кнопок;
   - `aria-live` для уведомлений;
   - корректная tab-навигация;
   - контрастность не ниже WCAG AA.
3. Экран тренировки должен иметь стабильную высоту, чтобы не было jump-эффектов при появлении нового текста.
4. Текст для печати должен быть разбит на элементы, например:

```html
<div class="typing-text" aria-label="Текст для печати">
  <span class="char current">h</span>
  <span class="char pending">e</span>
  <span class="char pending">l</span>
  <span class="char pending">l</span>
  <span class="char pending">o</span>
</div>
```

5. Виртуальная клавиатура:

```html
<div class="keyboard" aria-hidden="true">
  <div class="keyboard-row">
    <button class="key" data-key-code="KeyQ">q</button>
    <button class="key" data-key-code="KeyW">w</button>
  </div>
</div>
```

6. Панель статистики сессии:

```html
<div class="session-stats">
  <div class="stat">WPM: <span id="wpm">0</span></div>
  <div class="stat">CPM: <span id="cpm">0</span></div>
  <div class="stat">Accuracy: <span id="accuracy">100%</span></div>
  <div class="stat">Combo: <span id="combo">x1</span></div>
</div>
```

---

## 4.3 CSS

### Дизайн-система

Необходимо задать CSS variables:

```css
:root {
  --bg-main: #0b1020;
  --bg-panel: #151b31;
  --text-main: #e5e7eb;
  --text-muted: #9ca3af;
  --accent: #22d3ee;
  --success: #22c55e;
  --error: #ef4444;
  --warning: #f59e0b;
  --combo-fire: #fb923c;
  --radius-md: 12px;
  --shadow-soft: 0 10px 30px rgba(0, 0, 0, 0.25);
}
```

### Обязательные состояния символов

| Класс | Значение |
|---|---|
| `.char.pending` | ещё не напечатан |
| `.char.current` | текущий символ |
| `.char.correct` | напечатан правильно |
| `.char.error` | была ошибка |
| `.char.extra-error` | лишнее нажатие, если реализовано |

### Виртуальная клавиатура

Нужно подсветить:

- текущую клавишу;
- палец, который должен нажать клавишу;
- левую / правую руку;
- Shift, если символ требует регистра.

Пример цветов пальцев:

| Палец | Цвет |
|---|---|
| Левый мизинец | `#60a5fa` |
| Левый безымянный | `#34d399` |
| Левый средний | `#facc15` |
| Левый указательный | `#fb923c` |
| Правый указательный | `#f472b6` |
| Правый средний | `#a78bfa` |
| Правый безымянный | `#22d3ee` |
| Правый мизинец | `#94a3b8` |
| Большой палец | `#cbd5e1` |

### Анимации

Нужны:

1. Плавная смена текущего символа.
2. Анимация ошибки:
   - shake;
   - красная вспышка.
3. Анимация комбо:
   - рост счётчика;
   - огонь при x3 / x5.
4. Всплывающие очки:
   - `+1`, `+2`, `+5 XP`.
5. Уведомление о достижении:
   - slide-in;
   - glow.
6. Бонусы:
   - иконка бонуса;
   - таймер действия.

Важно: поддерживать `prefers-reduced-motion`.

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 4.4 JavaScript: ключевые модули

### Модуль `TypingEngine`

Отвечает за основной игровой процесс.

#### Ответственности

1. Получить упражнение с backend.
2. Отрендерить текст посимвольно.
3. Отслеживать `keydown`.
4. Сравнить нажатый символ с ожидаемым.
5. Посчитать:
   - correct chars;
   - incorrect chars;
   - combo;
   - score;
   - XP;
   - WPM;
   - CPM;
   - accuracy.
6. Отправить события на backend.
7. Завершить сессию.

#### Пример логики ввода

```js
function handleKeyDown(event) {
  if (isModifierKey(event)) return;

  event.preventDefault();

  const expectedChar = getCurrentExpectedChar();
  const actualChar = event.key;
  const keyCode = event.code;

  if (actualChar === expectedChar) {
    markCorrect();
    increaseCombo();
    addScoreAndXP();
    advanceCursor();
    sendEvent({
      type: "keypress",
      keyCode,
      expectedChar,
      actualChar,
      isCorrect: true
    });
  } else {
    markError();
    resetComboUnlessShielded();
    sendEvent({
      type: "error",
      keyCode,
      expectedChar,
      actualChar,
      isCorrect: false
    });
  }
}
```

#### Правила обработки ошибок

Для MVP рекомендуется режим «не пропускаем ошибку»:

- пользователь обязан нажать правильный символ;
- каждая неправильная клавиша увеличивает `incorrect_chars`;
- текущий ожидаемый символ не меняется;
- комбо сбрасывается, если нет бонуса `combo_keep` или `error_shield`.

Альтернативный режим, можно добавить позже:

- ошибка отмечает символ красным;
- пользователь может продолжить;
- точность падает;
- пропущенный символ засчитывается как `missed_chars`.

---

### Модуль `KeyboardHint`

Отвечает за подсказку пальцев.

#### Источник данных

Backend возвращает карту раскладки:

```json
{
  "layout": "en-qwerty",
  "char": "A",
  "keyCode": "KeyA",
  "shift": true,
  "finger": 1,
  "hand": "L"
}
```

#### Что делать на фронтенде

1. Получить текущий ожидаемый символ.
2. Найти соответствующую клавишу.
3. Подсветить клавишу.
4. Подсветить палец.
5. Если `shift=true`, дополнительно подсветить Shift.

---

### Модуль `GamificationUI`

Отвечает за:

- комбо-метр;
- очки;
- XP;
- бонусы;
- достижения;
- уровни.

#### Пример отображения комбо

| Комбо | Множитель | Визуал |
|---:|---:|---|
| 0–9 | x1 | обычный текст |
| 10–24 | x2 | синее свечение |
| 25–49 | x3 | оранжевый огонь |
| 50+ | x5 | красный огонь + частицы |

---

### Модуль `Effects`

Реализует:

- искры;
- огонь;
- конфетти;
- всплывающие `+XP`;
- ripple при правильном нажатии;
- shake при ошибке.

Рекомендация:

- простые эффекты делать на CSS;
- тяжёлые частицы — на Canvas;
- отключать при `reduced_motion` или настройке пользователя.

---

### Модуль `StatsCharts`

Использует Chart.js или D3.js.

Нужные графики:

1. Динамика WPM за последние 7 / 30 / 90 дней.
2. Динамика точности.
3. Столбчатая диаграмма самых сложных символов.
4. Heatmap клавиатуры по ошибкам.
5. Распределение времени по режимам.

---

### Модуль `AuthClient`

Хранение токенов:

- refresh token — лучше в `HttpOnly` cookie;
- access token — в памяти JS;
- при 401 автоматически обновлять токен.

Для гостя:

- прогресс в LocalStorage / IndexedDB;
- при регистрации предложить синхронизировать гостевой прогресс.

---

### Модуль `ApiClient`

Единый клиент для backend.

Пример:

```js
async function api(path, options = {}) {
  const response = await fetch(`/api/v1${path}`, {
    headers: {
      "Content-Type": "application/json",
      ...(options.headers || {})
    },
    credentials: "include",
    ...options
  });

  if (!response.ok) {
    throw await response.json();
  }

  return response.json();
}
```

---

## 4.5 Front-end: критерии приёмки

1. Пользователь может начать тренировку без регистрации.
2. После регистрации прогресс сохраняется.
3. Текст печатается посимвольно.
4. Текущий символ всегда подсвечен.
5. Ошибка визуально отличается от правильного символа.
6. Виртуальная клавиатура подсвечивает нужную клавишу и палец.
7. Комбо растёт без ошибок и сбрасывается при ошибке.
8. Бонусы отображаются с таймером.
9. Достижения показываются в профиле.
10. Статистика обновляется после завершения сессии.
11. Лидерборд фильтруется по периоду и метрике.
12. Приложение адаптировано под desktop и tablet.
13. Основные анимации не снижают FPS ниже 50 на среднем ноутбуке.
14. Поддержаны русская и английская раскладки.

---

# 5. Back-end: что нужно сделать

## 5.1 Архитектура backend

Рекомендуемая структура:

```text
app/
  main.py
  core/
    config.py
    security.py
    deps.py
  api/
    v1/
      auth.py
      users.py
      sessions.py
      exercises.py
      stats.py
      leaderboard.py
      achievements.py
      bonuses.py
      keyboard.py
  models/
    user.py
    session.py
    gamification.py
    dictionary.py
    keyboard.py
  schemas/
    auth.py
    session.py
    stats.py
    gamification.py
  services/
    auth_service.py
    exercise_service.py
    typing_service.py
    gamification_service.py
    achievement_service.py
    bonus_service.py
    stats_service.py
    leaderboard_service.py
  db/
    session.py
    base.py
  repositories/
    user_repo.py
    session_repo.py
    stats_repo.py
  workers/
    achievements_worker.py
    leaderboard_worker.py
alembic/
tests/
docker-compose.yml
Dockerfile
```

---

## 5.2 Основные сервисы

### `AuthService`

Ответственен за:

- регистрацию;
- вход;
- выход;
- обновление refresh token;
- OAuth;
- восстановление пароля;
- проверку прав.

Требования:

- пароль хранить в Argon2id или bcrypt;
- access token короткоживущий, например 15 минут;
- refresh token ротация;
- rate limiting на login/register.

---

### `ExerciseService`

Генерирует упражнения.

#### Режимы

| Режим | Источник данных |
|---|---|
| `words` | таблица `words` |
| `sentences` | таблица `sentences` |
| `code` | таблица `code_snippets` |
| `symbols` | таблица `symbol_sets` или генерация из набора символов |

#### Пример генерации слов

```python
def generate_words_session(language: str, level: int, target_chars: int = 200):
    min_len, max_len = get_length_range_by_level(level)
    words = repository.get_random_words(
        language=language,
        min_length=min_len,
        max_length=max_len,
        limit=50
    )
    text = build_text_from_words(words, target_chars)
    return text, metadata
```

#### Зависимость сложности от уровня

| Уровень | Слова | Предложения | Код | Символы |
|---:|---|---|---|---|
| 1 | короткие, частые | недоступны | недоступен | простые |
| 3 | средние | доступны | недоступен | простые + цифры |
| 5 | любые | доступны | доступен | средние |
| 8 | любые | сложные | доступен | сложные |
| 10+ | все | все | все | все |

---

### `SessionService`

Управляет жизненным циклом тренировки.

#### Состояния сессии

| Статус | Значение |
|---|---|
| `active` | пользователь печатает |
| `completed` | сессия завершена |
| `abandoned` | пользователь вышел / таймаут |

#### Методы

1. `start_session(user_id, mode, language, layout)`
2. `receive_events(session_id, events[])`
3. `complete_session(session_id)`
4. `abandon_session(session_id)`
5. `get_session_state(session_id)`

#### Безопасность

Backend должен считать итоговую статистику, а не доверять клиенту полностью.

Проверки:

- `seq` событий уникален и возрастает;
- время событий не уходит в будущее;
- интервал между событиями не отрицательный;
- слишком высокая CPM помечается как подозрительная;
- сессия привязана к пользователю или анонимному ID;
- повторная отправка того же `session_id + seq` идемпотентна.

---

### `TypingScoringService`

Считает очки, XP, комбо и бонусы.

#### Базовая формула

```text
base_xp = 1
combo_multiplier = f(combo)
level_multiplier = levels.bonus_multiplier
bonus_xp_multiplier = active bonuses

xp_delta = round(base_xp * combo_multiplier * level_multiplier * bonus_xp_multiplier)
score_delta = xp_delta * 10
```

#### Комбо

| Серия правильных нажатий | Множитель |
|---:|---:|
| 0–9 | x1 |
| 10–24 | x2 |
| 25–49 | x3 |
| 50+ | x5 |

#### Ошибки

По умолчанию:

- `incorrect_chars += 1`;
- combo = 0;
- XP не начисляется;
- событие сохраняется.

Если активен `error_shield`:

- ошибка не сбрасывает комбо;
- ошибка может не увеличивать `incorrect_chars`, если это предусмотрено эффектом;
- событие сохраняется с `metadata.shielded = true`.

Если активен `combo_keep`:

- одна ошибка не сбрасывает комбо;
- ошибка всё равно может учитываться в точности.

---

### `AccuracyService`

Формулы:

```text
accuracy = correct_chars / (correct_chars + incorrect_chars) * 100
cpm = correct_chars / duration_minutes
wpm = cpm / 5
```

Если длительность равна 0:

```text
cpm = 0
wpm = 0
accuracy = 100, если correct_chars > 0, иначе 0
```

---

### `LevelService`

Уровень определяется по суммарному XP.

```sql
SELECT level
FROM levels
WHERE xp_required <= total_xp
ORDER BY level DESC
LIMIT 1;
```

При повышении уровня:

- обновляется `user_progress.level`;
- открываются новые режимы;
- может начисляться бонус XP;
- пользователь получает уведомление.

---

### `AchievementService`

Проверяет достижения после завершения сессии.

Источники метрик:

- `user_progress`;
- поля завершённой сессии;
- количество сессий по режимам;
- языки;
- streak;
- уровень.

Пример условия:

```json
{
  "metric": "longest_combo",
  "operator": ">=",
  "value": 1000
}
```

Сервис должен быть идемпотентным: одно достижение нельзя выдать дважды.

---

### `BonusService`

Управляет случайными и активными бонусами.

#### Триггер бонуса

После каждого правильного символа:

1. Проверить cooldown.
2. Проверить `max_per_session`.
3. Проверить минимальный уровень.
4. Бросить random против `trigger_chance`.
5. Если выпал — активировать бонус.

#### Типы эффектов

| effect_type | Значение |
|---|---|
| `xp_multiplier` | умножает XP |
| `score_multiplier` | умножает очки |
| `cpm_multiplier` | увеличивает бонусную скорость для начисления |
| `error_shield` | защищает от ошибок |
| `combo_keep` | сохраняет комбо при ошибке |
| `instant_xp` | мгновенно даёт XP |
| `instant_score` | мгновенно даёт очки |
| `hint_boost` | улучшает подсказку, опционально |

Важно: бонус `+50% к скорости` не должен менять реальную измеренную скорость для честного лидерборда. Он влияет на бонусные очки / XP и визуальный отображаемый boost.

---

### `StatsService`

Формирует статистику.

#### Текущая сессия

- WPM;
- CPM;
- accuracy;
- errors;
- combo;
- score;
- XP.

#### Общая статистика

- количество тренировок;
- общее время;
- суммарные символы;
- лучший WPM;
- лучший CPM;
- лучшая точность;
- самый длинный комбо;
- XP;
- уровень.

#### Детальный разбор

- самые частые ошибки по символам;
- среднее время до нажатия;
- проблемные сочетания;
- разница между русским и английским;
- разница между режимами.

---

### `LeaderboardService`

Возвращает топ пользователей.

Метрики:

- WPM;
- CPM;
- accuracy;
- XP.

Периоды:

- today;
- week;
- month;
- all time.

Для производительности можно использовать:

- материализованные представления;
- Redis sorted sets;
- кэш на 1–5 минут.

---

## 5.3 API endpoints

Базовый префикс: `/api/v1`

### Auth

| Метод | URL | Описание |
|---|---|---|
| POST | `/auth/register` | регистрация |
| POST | `/auth/login` | вход |
| POST | `/auth/refresh` | обновить access token |
| POST | `/auth/logout` | выход |
| GET | `/auth/oauth/{provider}` | OAuth redirect |
| POST | `/auth/oauth/{provider}/callback` | OAuth callback |

### Users

| Метод | URL | Описание |
|---|---|---|
| GET | `/users/me` | профиль |
| PATCH | `/users/me` | обновить профиль |
| GET | `/users/me/settings` | настройки |
| PATCH | `/users/me/settings` | обновить настройки |
| GET | `/users/me/progress` | прогресс |

### Exercises

| Метод | URL | Описание |
|---|---|---|
| GET | `/exercises/next` | получить новое упражнение |
| GET | `/exercises/modes` | список режимов |
| GET | `/exercises/languages` | список языков |

Пример:

```http
GET /api/v1/exercises/next?mode=words&language=en&level=3
```

Ответ:

```json
{
  "exercise_id": "uuid",
  "mode": "words",
  "language": "en",
  "text": "the quick brown fox jumps",
  "metadata": {
    "word_ids": [1, 2, 3]
  }
}
```

### Sessions

| Метод | URL | Описание |
|---|---|---|
| POST | `/sessions/start` | начать сессию |
| POST | `/sessions/{id}/events` | отправить пакет событий |
| POST | `/sessions/{id}/complete` | завершить сессию |
| POST | `/sessions/{id}/abandon` | прервать сессию |
| GET | `/sessions/{id}` | состояние сессии |

Пример отправки событий:

```json
{
  "events": [
    {
      "seq": 1,
      "occurred_at": "2026-10-20T10:00:01.123Z",
      "type": "keypress",
      "key_code": "KeyT",
      "expected_char": "t",
      "actual_char": "t",
      "is_correct": true
    },
    {
      "seq": 2,
      "occurred_at": "2026-10-20T10:00:01.300Z",
      "type": "error",
      "key_code": "KeyR",
      "expected_char": "h",
      "actual_char": "r",
      "is_correct": false
    }
  ]
}
```

### Stats

| Метод | URL | Описание |
|---|---|---|
| GET | `/stats/summary` | общая статистика |
| GET | `/stats/timeline` | динамика по дням |
| GET | `/stats/characters` | сложные символы |
| GET | `/stats/modes` | статистика по режимам |

### Achievements

| Метод | URL | Описание |
|---|---|---|
| GET | `/achievements` | список всех достижений |
| GET | `/achievements/me` | мои достижения |
| POST | `/achievements/{code}/claim` | опционально, если требуется ручное получение |

### Bonuses

| Метод | URL | Описание |
|---|---|---|
| GET | `/bonuses/catalog` | список бонусов |
| GET | `/bonuses/active` | активные бонусы пользователя |

### Leaderboard

| Метод | URL | Описание |
|---|---|---|
| GET | `/leaderboard` | таблица лидеров |

Пример:

```http
GET /api/v1/leaderboard?metric=wpm&period=week&limit=50
```

### Keyboard

| Метод | URL | Описание |
|---|---|---|
| GET | `/keyboard/layouts` | список раскладок |
| GET | `/keyboard/hint` | подсказка для символа |

Пример:

```http
GET /api/v1/keyboard/hint?layout=en-qwerty&char=A
```

Ответ:

```json
{
  "char": "A",
  "key_code": "KeyA",
  "shift": true,
  "finger": 1,
  "finger_name": "left_pinky",
  "hand": "L"
}
```

---

# 6. Геймификация: уровни, достижения, бонусы

## 6.1 Уровни

Пример системы уровней:

| Уровень | Название | XP | Что открывает | Множитель |
|---:|---|---:|---|---:|
| 1 | Новичок | 0 | слова, простые символы | x1.00 |
| 2 | Ученик | 100 | слова, символы | x1.00 |
| 3 | Практикант | 250 | предложения | x1.00 |
| 4 | Оператор | 500 | предложения, слова повышенной сложности | x1.05 |
| 5 | Скоростной | 900 | режим «Код» | x1.05 |
| 6 | Виртуоз | 1500 | все базовые режимы | x1.10 |
| 7 | Мастер | 2500 | сложные символы | x1.10 |
| 8 | Эксперт | 4000 | расширенные наборы символов | x1.15 |
| 9 | Профи | 6500 | повышенные бонусы за точность | x1.15 |
| 10 | Легенда | 10000 | все режимы | x1.20 |
| 11 | Киберпалец | 15000 | специальные достижения | x1.25 |
| 12 | Печатный маг | 22000 | редкие бонусы | x1.30 |
| 13 | Виртуоз кода | 32000 | сложные code-сессии | x1.35 |
| 14 | Мастер потока | 45000 | максимальные комбо-бонусы | x1.40 |
| 15 | Легенда клавиатуры | 60000 | все возможности | x1.50 |

---

## 6.2 Достижения

Примеры достижений:

| Код | Название | Условие | Награда |
|---|---|---|---:|
| `first_steps` | Первые шаги | завершить 1 тренировку | 10 XP |
| `hundred_chars` | Сто символов | напечатать 100 верных символов суммарно | 25 XP |
| `steel_finger` | Стальной палец | 1000 правильных нажатий подряд | 250 XP |
| `printing_genius` | Печатный гений | достичь 300 WPM | 500 XP |
| `speed_demon` | Демон скорости | 200 CPM в сессии от 100 символов | 150 XP |
| `laser_focus` | Лазерный фокус | точность 98%+ при 200+ символах | 150 XP |
| `flawless_run` | Безупречный пробег | сессия 100+ символов без ошибок | 100 XP |
| `combo_master` | Мастер комбо | достичь комбо x5, 50 подряд | 75 XP |
| `code_ninja` | Кодовый ниндзя | завершить 10 сессий в режиме «Код» | 200 XP |
| `polyglot` | Полиглот | тренироваться на русском и английском | 100 XP |
| `marathoner` | Марафонец | 10 часов общего времени тренировок | 300 XP |
| `weekly_streak` | Серия недели | 7 дней подряд с тренировкой | 175 XP |
| `symbol_slayer` | Покоритель символов | сессия Symbols 95%+ и 150+ символов | 120 XP |
| `level_10` | Десятый уровень | достичь 10 уровня | 250 XP |

---

## 6.3 Бонусы

Примеры бонусов:

| Код | Название | Эффект | Длительность | Шанс |
|---|---|---|---:|---:|
| `double_xp_10s` | Двойной опыт | XP x2 | 10 сек | 8% |
| `score_surge_15s` | Всплеск очков | очки x1.5 | 15 сек | 7% |
| `speed_boost_20s` | Ускорение | бонусная скорость +50% для начисления | 20 сек | 5% |
| `error_shield_10s` | Щит ошибок | ошибки не сбрасывают комбо, до 5 ошибок | 10 сек | 6% |
| `combo_keep_15s` | Сохранение комбо | одна ошибка не прерывает серию | 15 сек | 4% |
| `xp_injection` | Инъекция опыта | мгновенно +100 XP | мгновенно | 3% |

Важно:

- бонусы не должны ломать честность лидерборда;
- реальные WPM/CPM/accuracy сохраняются без бонусных искажений;
- бонусы влияют на XP, очки и визуальные эффекты.

---

# 7. Модель данных PostgreSQL

## 7.1 Основные таблицы

| Таблица | Назначение |
|---|---|
| `languages` | языки: ru, en |
| `keyboard_layouts` | раскладки: QWERTY, ЙЦУКЕН |
| `keys` | физические клавиши |
| `key_variants` | символы, регистр, палец, рука |
| `exercise_modes` | режимы тренировки |
| `words` | словарь слов |
| `sentences` | предложения |
| `code_snippets` | фрагменты кода |
| `symbol_sets` | наборы символов |
| `users` | пользователи |
| `oauth_accounts` | OAuth-аккаунты |
| `refresh_tokens` | refresh tokens |
| `user_settings` | настройки пользователя |
| `levels` | уровни и XP-пороги |
| `user_progress` | прогресс пользователя |
| `sessions` | тренировки |
| `session_events` | события ввода |
| `character_stats` | статистика по символам |
| `achievements` | справочник достижений |
| `user_achievements` | полученные достижения |
| `bonus_definitions` | справочник бонусов |
| `user_active_bonuses` | активные бонусы |
| `session_bonus_activations` | активации бонусов в сессии |

---

# 8. SQL-скрипт для PostgreSQL

Ниже приведён готовый скрипт для создания базы данных, таблиц, индексов, функций, представлений и стартовых данных.

Скрипт рассчитан на PostgreSQL 15+.

```sql
-- ============================================================================
-- typing_trainer PostgreSQL schema and seed data
-- Target: PostgreSQL 15+
-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

-- ============================================================================
-- Utility function
-- ============================================================================

CREATE OR REPLACE FUNCTION public.set_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$;

-- ============================================================================
-- Reference: languages
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.languages (
    code TEXT PRIMARY KEY CHECK (code IN ('ru', 'en')),
    name TEXT NOT NULL,
    native_name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================================
-- Reference: keyboard layouts
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.keyboard_layouts (
    id TEXT PRIMARY KEY,
    code TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    language_code TEXT NOT NULL REFERENCES public.languages(code) ON DELETE RESTRICT,
    is_default BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================================
-- Physical keyboard keys
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.keys (
    key_code TEXT PRIMARY KEY,
    row_no SMALLINT NOT NULL,
    col_no NUMERIC NOT NULL,
    width_units NUMERIC NOT NULL DEFAULT 1,
    key_type TEXT NOT NULL DEFAULT 'printable'
        CHECK (key_type IN ('printable', 'space', 'enter', 'tab', 'modifier', 'backspace')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================================
-- Key variants: char, shift, finger, hand
--
-- finger:
-- 1 = left pinky
-- 2 = left ring
-- 3 = left middle
-- 4 = left index
-- 5 = right index
-- 6 = right middle
-- 7 = right ring
-- 8 = right pinky
-- 9 = thumb
-- 0 = unknown / special
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.key_variants (
    layout_id TEXT NOT NULL REFERENCES public.keyboard_layouts(id) ON DELETE CASCADE,
    key_code TEXT NOT NULL REFERENCES public.keys(key_code) ON DELETE CASCADE,
    char TEXT NOT NULL CHECK (char <> ''),
    shift BOOLEAN NOT NULL DEFAULT FALSE,
    altgr BOOLEAN NOT NULL DEFAULT FALSE,
    finger SMALLINT NOT NULL CHECK (finger BETWEEN 0 AND 9),
    hand CHAR(1) NOT NULL CHECK (hand IN ('L', 'R', 'T')),
    PRIMARY KEY (layout_id, key_code, shift, altgr)
);

CREATE INDEX IF NOT EXISTS idx_key_variants_char
    ON public.key_variants (layout_id, char, shift, altgr);

COMMENT ON COLUMN public.key_variants.finger IS
    '1 LP, 2 LR, 3 LM, 4 LI, 5 RI, 6 RM, 7 RR, 8 RP, 9 thumb, 0 unknown';

-- ============================================================================
-- Exercise modes
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.exercise_modes (
    code TEXT PRIMARY KEY CHECK (code IN ('words', 'sentences', 'code', 'symbols')),
    name TEXT NOT NULL,
    description TEXT,
    min_level SMALLINT NOT NULL DEFAULT 1 CHECK (min_level >= 1),
    default_language_code TEXT REFERENCES public.languages(code) ON DELETE SET NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================================
-- Dictionaries
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.words (
    id BIGSERIAL PRIMARY KEY,
    language_code TEXT NOT NULL REFERENCES public.languages(code) ON DELETE CASCADE,
    word TEXT NOT NULL,
    length INTEGER GENERATED ALWAYS AS (char_length(word)) STORED,
    frequency INTEGER NOT NULL DEFAULT 0,
    difficulty SMALLINT NOT NULL DEFAULT 1 CHECK (difficulty BETWEEN 1 AND 5),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (language_code, word)
);

CREATE INDEX IF NOT EXISTS idx_words_lang_diff_length
    ON public.words (language_code, difficulty, length);

CREATE TABLE IF NOT EXISTS public.sentences (
    id BIGSERIAL PRIMARY KEY,
    language_code TEXT NOT NULL REFERENCES public.languages(code) ON DELETE CASCADE,
    text TEXT NOT NULL,
    difficulty SMALLINT NOT NULL DEFAULT 1 CHECK (difficulty BETWEEN 1 AND 5),
    char_count INTEGER GENERATED ALWAYS AS (char_length(text)) STORED,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (language_code, text)
);

CREATE TABLE IF NOT EXISTS public.code_snippets (
    id BIGSERIAL PRIMARY KEY,
    programming_language TEXT NOT NULL
        CHECK (programming_language IN ('python', 'javascript', 'html_css')),
    code TEXT NOT NULL,
    difficulty SMALLINT NOT NULL DEFAULT 1 CHECK (difficulty BETWEEN 1 AND 5),
    char_count INTEGER GENERATED ALWAYS AS (char_length(code)) STORED,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (programming_language, code)
);

CREATE TABLE IF NOT EXISTS public.symbol_sets (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL UNIQUE,
    chars TEXT NOT NULL,
    difficulty SMALLINT NOT NULL DEFAULT 1 CHECK (difficulty BETWEEN 1 AND 5),
    language_code TEXT REFERENCES public.languages(code) ON DELETE SET NULL,
    char_count INTEGER GENERATED ALWAYS AS (char_length(chars)) STORED,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================================
-- Users and auth
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email CITEXT NOT NULL UNIQUE,
    password_hash TEXT,
    display_name TEXT NOT NULL DEFAULT 'Printer',
    avatar_url TEXT,
    timezone TEXT NOT NULL DEFAULT 'UTC',
    locale TEXT NOT NULL DEFAULT 'en',
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    is_email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    role TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('user', 'admin')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_login_at TIMESTAMPTZ
);

CREATE INDEX IF NOT EXISTS idx_users_email ON public.users (email);

CREATE OR REPLACE TRIGGER trg_users_updated
BEFORE UPDATE ON public.users
FOR EACH ROW
EXECUTE FUNCTION public.set_updated_at();

CREATE TABLE IF NOT EXISTS public.oauth_accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    provider TEXT NOT NULL CHECK (provider IN ('google', 'yandex', 'github', 'vk')),
    provider_user_id TEXT NOT NULL,
    access_token TEXT,
    refresh_token TEXT,
    expires_at TIMESTAMPTZ,
    scopes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (provider, provider_user_id)
);

CREATE INDEX IF NOT EXISTS idx_oauth_user ON public.oauth_accounts (user_id);

CREATE TABLE IF NOT EXISTS public.refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    token_hash TEXT NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip INET,
    user_agent TEXT
);

CREATE INDEX IF NOT EXISTS idx_refresh_user
    ON public.refresh_tokens (user_id, expires_at);

CREATE TABLE IF NOT EXISTS public.user_settings (
    user_id UUID PRIMARY KEY REFERENCES public.users(id) ON DELETE CASCADE,
    keyboard_layout_id TEXT REFERENCES public.keyboard_layouts(id) ON DELETE SET NULL,
    sound_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    visual_effects BOOLEAN NOT NULL DEFAULT TRUE,
    reduced_motion BOOLEAN NOT NULL DEFAULT FALSE,
    theme TEXT NOT NULL DEFAULT 'dark' CHECK (theme IN ('light', 'dark', 'system')),
    show_keyboard_hint BOOLEAN NOT NULL DEFAULT TRUE,
    highlight_style TEXT NOT NULL DEFAULT 'char' CHECK (highlight_style IN ('char', 'word')),
    daily_goal_minutes INTEGER NOT NULL DEFAULT 15 CHECK (daily_goal_minutes BETWEEN 0 AND 480),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE OR REPLACE TRIGGER trg_user_settings_updated
BEFORE UPDATE ON public.user_settings
FOR EACH ROW
EXECUTE FUNCTION public.set_updated_at();

-- ============================================================================
-- Levels
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.levels (
    level SMALLINT PRIMARY KEY CHECK (level >= 1),
    title TEXT NOT NULL,
    xp_required BIGINT NOT NULL CHECK (xp_required >= 0),
    unlocks_modes TEXT[] NOT NULL DEFAULT ARRAY[]::TEXT[],
    bonus_multiplier NUMERIC(4,2) NOT NULL DEFAULT 1.00 CHECK (bonus_multiplier > 0),
    color TEXT,
    icon TEXT,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (xp_required)
);

-- ============================================================================
-- User progress
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.user_progress (
    user_id UUID PRIMARY KEY REFERENCES public.users(id) ON DELETE CASCADE,
    total_xp BIGINT NOT NULL DEFAULT 0 CHECK (total_xp >= 0),
    level SMALLINT NOT NULL DEFAULT 1 REFERENCES public.levels(level) ON DELETE RESTRICT,
    total_sessions INTEGER NOT NULL DEFAULT 0,
    total_time_seconds BIGINT NOT NULL DEFAULT 0,
    total_chars INTEGER NOT NULL DEFAULT 0,
    correct_chars INTEGER NOT NULL DEFAULT 0,
    incorrect_chars INTEGER NOT NULL DEFAULT 0,
    best_wpm NUMERIC(7,2) NOT NULL DEFAULT 0 CHECK (best_wpm >= 0),
    best_cpm NUMERIC(7,2) NOT NULL DEFAULT 0 CHECK (best_cpm >= 0),
    best_accuracy NUMERIC(5,2) NOT NULL DEFAULT 0 CHECK (best_accuracy BETWEEN 0 AND 100),
    longest_combo INTEGER NOT NULL DEFAULT 0 CHECK (longest_combo >= 0),
    streak_days INTEGER NOT NULL DEFAULT 0 CHECK (streak_days >= 0),
    last_session_date DATE,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_user_progress_xp ON public.user_progress (total_xp DESC);
CREATE INDEX IF NOT EXISTS idx_user_progress_wpm ON public.user_progress (best_wpm DESC);
CREATE INDEX IF NOT EXISTS idx_user_progress_cpm ON public.user_progress (best_cpm DESC);
CREATE INDEX IF NOT EXISTS idx_user_progress_accuracy ON public.user_progress (best_accuracy DESC);

-- ============================================================================
-- Sessions
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE SET NULL,
    anonymous_id TEXT,
    mode_code TEXT NOT NULL REFERENCES public.exercise_modes(code) ON DELETE RESTRICT,
    language_code TEXT NOT NULL REFERENCES public.languages(code) ON DELETE RESTRICT,
    keyboard_layout_id TEXT REFERENCES public.keyboard_layouts(id) ON DELETE SET NULL,
    started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    ended_at TIMESTAMPTZ,
    duration_seconds INTEGER NOT NULL DEFAULT 0 CHECK (duration_seconds >= 0),
    status TEXT NOT NULL DEFAULT 'active'
        CHECK (status IN ('active', 'completed', 'abandoned')),
    total_chars INTEGER NOT NULL DEFAULT 0 CHECK (total_chars >= 0),
    correct_chars INTEGER NOT NULL DEFAULT 0 CHECK (correct_chars >= 0),
    incorrect_chars INTEGER NOT NULL DEFAULT 0 CHECK (incorrect_chars >= 0),
    missed_chars INTEGER NOT NULL DEFAULT 0 CHECK (missed_chars >= 0),
    accuracy NUMERIC(5,2) NOT NULL DEFAULT 0 CHECK (accuracy BETWEEN 0 AND 100),
    wpm NUMERIC(7,2) NOT NULL DEFAULT 0 CHECK (wpm >= 0),
    cpm NUMERIC(7,2) NOT NULL DEFAULT 0 CHECK (cpm >= 0),
    max_combo INTEGER NOT NULL DEFAULT 0 CHECK (max_combo >= 0),
    score BIGINT NOT NULL DEFAULT 0 CHECK (score >= 0),
    xp_earned BIGINT NOT NULL DEFAULT 0 CHECK (xp_earned >= 0),
    level_before SMALLINT,
    level_after SMALLINT,
    anti_cheat JSONB NOT NULL DEFAULT '{}'::jsonb,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (user_id IS NOT NULL OR anonymous_id IS NOT NULL),
    CHECK (total_chars >= correct_chars + incorrect_chars)
);

CREATE INDEX IF NOT EXISTS idx_sessions_user_status
    ON public.sessions (user_id, status, ended_at DESC);

CREATE INDEX IF NOT EXISTS idx_sessions_mode_ended
    ON public.sessions (mode_code, ended_at DESC);

CREATE INDEX IF NOT EXISTS idx_sessions_completed_ended
    ON public.sessions (ended_at DESC)
    WHERE status = 'completed';

CREATE OR REPLACE TRIGGER trg_sessions_updated
BEFORE UPDATE ON public.sessions
FOR EACH ROW
EXECUTE FUNCTION public.set_updated_at();

-- ============================================================================
-- Session events
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.session_events (
    id BIGSERIAL PRIMARY KEY,
    session_id UUID NOT NULL REFERENCES public.sessions(id) ON DELETE CASCADE,
    seq INTEGER NOT NULL CHECK (seq >= 0),
    occurred_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    event_type TEXT NOT NULL
        CHECK (event_type IN (
            'keypress',
            'error',
            'combo',
            'bonus',
            'achievement',
            'word_complete',
            'line_complete',
            'session_start',
            'session_end',
            'pause',
            'resume'
        )),
    key_code TEXT REFERENCES public.keys(key_code) ON DELETE SET NULL,
    expected_char TEXT,
    actual_char TEXT,
    is_correct BOOLEAN,
    combo_before INTEGER,
    combo_after INTEGER,
    score_delta INTEGER NOT NULL DEFAULT 0,
    xp_delta INTEGER NOT NULL DEFAULT 0,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    UNIQUE (session_id, seq)
);

CREATE INDEX IF NOT EXISTS idx_session_events_session_time
    ON public.session_events (session_id, occurred_at);

CREATE INDEX IF NOT EXISTS idx_session_events_type
    ON public.session_events (event_type);

-- ============================================================================
-- Character stats
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.character_stats (
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    char TEXT NOT NULL CHECK (char <> ''),
    attempts INTEGER NOT NULL DEFAULT 0 CHECK (attempts >= 0),
    correct INTEGER NOT NULL DEFAULT 0 CHECK (correct >= 0),
    incorrect INTEGER NOT NULL DEFAULT 0 CHECK (incorrect >= 0),
    total_time_ms BIGINT NOT NULL DEFAULT 0 CHECK (total_time_ms >= 0),
    avg_delay_ms NUMERIC(10,2) NOT NULL DEFAULT 0,
    last_seen_at TIMESTAMPTZ,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (user_id, char),
    CHECK (correct <= attempts),
    CHECK (incorrect <= attempts)
);

CREATE INDEX IF NOT EXISTS idx_character_stats_errors
    ON public.character_stats (user_id, incorrect DESC);

CREATE OR REPLACE TRIGGER trg_character_stats_updated
BEFORE UPDATE ON public.character_stats
FOR EACH ROW
EXECUTE FUNCTION public.set_updated_at();

-- ============================================================================
-- Achievements
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.achievements (
    code TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    category TEXT NOT NULL
        CHECK (category IN ('speed', 'accuracy', 'combo', 'volume', 'mode', 'social', 'special')),
    condition_type TEXT NOT NULL
        CHECK (condition_type IN (
            'metric_threshold',
            'session_condition',
            'count_condition',
            'multi_mode_condition',
            'streak_condition',
            'level_condition'
        )),
    condition_value JSONB NOT NULL,
    xp_reward BIGINT NOT NULL DEFAULT 0 CHECK (xp_reward >= 0),
    score_reward BIGINT NOT NULL DEFAULT 0 CHECK (score_reward >= 0),
    badge_icon TEXT,
    badge_color TEXT,
    is_hidden BOOLEAN NOT NULL DEFAULT FALSE,
    unlock_level SMALLINT REFERENCES public.levels(level) ON DELETE SET NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE IF NOT EXISTS public.user_achievements (
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    achievement_code TEXT NOT NULL REFERENCES public.achievements(code) ON DELETE CASCADE,
    progress_value NUMERIC NOT NULL DEFAULT 0 CHECK (progress_value >= 0),
    target_value NUMERIC NOT NULL DEFAULT 1 CHECK (target_value > 0),
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    unlocked_at TIMESTAMPTZ,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (user_id, achievement_code)
);

CREATE INDEX IF NOT EXISTS idx_user_achievements_completed
    ON public.user_achievements (user_id, completed, unlocked_at DESC);

CREATE OR REPLACE TRIGGER trg_user_achievements_updated
BEFORE UPDATE ON public.user_achievements
FOR EACH ROW
EXECUTE FUNCTION public.set_updated_at();

-- ============================================================================
-- Bonuses
-- ============================================================================

CREATE TABLE IF NOT EXISTS public.bonus_definitions (
    code TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT NOT NULL,
    effect_type TEXT NOT NULL
        CHECK (effect_type IN (
            'xp_multiplier',
            'score_multiplier',
            'cpm_multiplier',
            'error_shield',
            'combo_keep',
            'instant_xp',
            'instant_score',
            'hint_boost'
        )),
    effect_value JSONB NOT NULL,
    duration_seconds INTEGER NOT NULL DEFAULT 0 CHECK (duration_seconds >= 0),
    trigger_chance NUMERIC(5,4) NOT NULL DEFAULT 0 CHECK (trigger_chance BETWEEN 0 AND 1),
    cooldown_seconds INTEGER NOT NULL DEFAULT 0 CHECK (cooldown_seconds >= 0),
    max_per_session INTEGER NOT NULL DEFAULT 1 CHECK (max_per_session >= 1),
    min_level SMALLINT NOT NULL DEFAULT 1 REFERENCES public.levels(level) ON DELETE RESTRICT,
    icon TEXT,
    color TEXT,
    is_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE IF NOT EXISTS public.user_active_bonuses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    bonus_code TEXT NOT NULL REFERENCES public.bonus_definitions(code) ON DELETE CASCADE,
    session_id UUID REFERENCES public.sessions(id) ON DELETE CASCADE,
    started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at TIMESTAMPTZ,
    effect_snapshot JSONB NOT NULL,
    status TEXT NOT NULL DEFAULT 'active'
        CHECK (status IN ('active', 'expired', 'consumed', 'cancelled')),
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    CHECK (expires_at IS NULL OR expires_at > started_at)
);

CREATE INDEX IF NOT EXISTS idx_user_active_bonuses
    ON public.user_active_bonuses (user_id, status, expires_at);

CREATE TABLE IF NOT EXISTS public.session_bonus_activations (
    id BIGSERIAL PRIMARY KEY,
    session_id UUID NOT NULL REFERENCES public.sessions(id) ON DELETE CASCADE,
    bonus_code TEXT NOT NULL REFERENCES public.bonus_definitions(code) ON DELETE CASCADE,
    activated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    duration_seconds INTEGER NOT NULL DEFAULT 0,
    effect JSONB NOT NULL,
    source TEXT NOT NULL DEFAULT 'random'
        CHECK (source IN ('random', 'achievement', 'manual', 'admin', 'level_up')),
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb
);

CREATE INDEX IF NOT EXISTS idx_session_bonus_session
    ON public.session_bonus_activations (session_id, activated_at);

-- ============================================================================
-- Simple updated_at triggers for tables that use default function
-- ============================================================================

-- Already created above for users, user_settings, sessions, character_stats,
-- user_achievements. Additional triggers can be added here if needed.

-- ============================================================================
-- SEED: languages
-- ============================================================================

INSERT INTO public.languages (code, name, native_name)
VALUES
    ('ru', 'Russian', 'Русский'),
    ('en', 'English', 'English')
ON CONFLICT (code) DO UPDATE
SET name = EXCLUDED.name,
    native_name = EXCLUDED.native_name;

-- ============================================================================
-- SEED: keyboard layouts
-- ============================================================================

INSERT INTO public.keyboard_layouts (id, code, name, language_code, is_default)
VALUES
    ('en-qwerty', 'qwerty', 'English QWERTY', 'en', TRUE),
    ('ru-yucuen', 'yucuen', 'Russian ЙЦУКЕН', 'ru', FALSE)
ON CONFLICT (id) DO UPDATE
SET code = EXCLUDED.code,
    name = EXCLUDED.name,
    language_code = EXCLUDED.language_code,
    is_default = EXCLUDED.is_default;

-- ============================================================================
-- SEED: exercise modes
-- ============================================================================

INSERT INTO public.exercise_modes (code, name, description, min_level, default_language_code, metadata)
VALUES
    ('words', 'Слова', 'Случайные слова из словаря', 1, NULL, '{"min_length":2,"max_length":10}'::jsonb),
    ('sentences', 'Предложения', 'Связные предложения на русском и английском', 3, NULL, '{}'::jsonb),
    ('code', 'Код', 'Фрагменты кода Python, JavaScript, HTML/CSS', 5, NULL, '{}'::jsonb),
    ('symbols', 'Символы', 'Случайные буквы, цифры и спецсимволы', 1, NULL, '{}'::jsonb)
ON CONFLICT (code) DO UPDATE
SET name = EXCLUDED.name,
    description = EXCLUDED.description,
    min_level = EXCLUDED.min_level,
    default_language_code = EXCLUDED.default_language_code,
    metadata = EXCLUDED.metadata;

-- ============================================================================
-- SEED: keyboard mapping
--
-- This uses a temporary table to avoid repeating a large VALUES list.
-- Run the whole script in one session.
-- ============================================================================

DROP TABLE IF EXISTS tmp_layout_map;

CREATE TEMP TABLE tmp_layout_map (
    layout_id TEXT,
    key_code TEXT,
    row_no SMALLINT,
    col_no NUMERIC,
    width_units NUMERIC,
    unshifted TEXT,
    shifted TEXT,
    finger SMALLINT,
    hand CHAR(1)
);

INSERT INTO tmp_layout_map
    (layout_id, key_code, row_no, col_no, width_units, unshifted, shifted, finger, hand)
VALUES
-- EN QWERTY row 0
('en-qwerty','Backquote',0,0,1,'`','~',1,'L'),
('en-qwerty','Digit1',0,1,1,'1','!',1,'L'),
('en-qwerty','Digit2',0,2,1,'2','@',2,'L'),
('en-qwerty','Digit3',0,3,1,'3','#',3,'L'),
('en-qwerty','Digit4',0,4,1,'4','$',4,'L'),
('en-qwerty','Digit5',0,5,1,'5','%',4,'L'),
('en-qwerty','Digit6',0,6,1,'6','^',5,'R'),
('en-qwerty','Digit7',0,7,1,'7','&',5,'R'),
('en-qwerty','Digit8',0,8,1,'8','*',6,'R'),
('en-qwerty','Digit9',0,9,1,'9','(',7,'R'),
('en-qwerty','Digit0',0,10,1,'0',')',8,'R'),
('en-qwerty','Minus',0,11,1,'-','_',8,'R'),
('en-qwerty','Equal',0,12,1,'=','+',8,'R'),

-- EN QWERTY row 1
('en-qwerty','KeyQ',1,1,1,'q','Q',1,'L'),
('en-qwerty','KeyW',1,2,1,'w','W',2,'L'),
('en-qwerty','KeyE',1,3,1,'e','E',3,'L'),
('en-qwerty','KeyR',1,4,1,'r','R',4,'L'),
('en-qwerty','KeyT',1,5,1,'t','T',4,'L'),
('en-qwerty','KeyY',1,6,1,'y','Y',5,'R'),
('en-qwerty','KeyU',1,7,1,'u','U',5,'R'),
('en-qwerty','KeyI',1,8,1,'i','I',6,'R'),
('en-qwerty','KeyO',1,9,1,'o','O',7,'R'),
('en-qwerty','KeyP',1,10,1,'p','P',8,'R'),
('en-qwerty','BracketLeft',1,11,1,'[','{',8,'R'),
('en-qwerty','BracketRight',1,12,1,']','}',8,'R'),
('en-qwerty','Backslash',1,13,1.5,E'\\','|',8,'R'),

-- EN QWERTY row 2
('en-qwerty','KeyA',2,1,1,'a','A',1,'L'),
('en-qwerty','KeyS',2,2,1,'s','S',2,'L'),
('en-qwerty','KeyD',2,3,1,'d','D',3,'L'),
('en-qwerty','KeyF',2,4,1,'f','F',4,'L'),
('en-qwerty','KeyG',2,5,1,'g','G',4,'L'),
('en-qwerty','KeyH',2,6,1,'h','H',5,'R'),
('en-qwerty','KeyJ',2,7,1,'j','J',5,'R'),
('en-qwerty','KeyK',2,8,1,'k','K',6,'R'),
('en-qwerty','KeyL',2,9,1,'l','L',7,'R'),
('en-qwerty','Semicolon',2,11,1,';',':',7,'R'),
('en-qwerty','Quote',2,12,1,chr(39),'"',8,'R'),

-- EN QWERTY row 3
('en-qwerty','KeyZ',3,1,1,'z','Z',1,'L'),
('en-qwerty','KeyX',3,2,1,'x','X',2,'L'),
('en-qwerty','KeyC',3,3,1,'c','C',3,'L'),
('en-qwerty','KeyV',3,4,1,'v','V',4,'L'),
('en-qwerty','KeyB',3,5,1,'b','B',4,'L'),
('en-qwerty','KeyN',3,6,1,'n','N',5,'R'),
('en-qwerty','KeyM',3,7,1,'m','M',5,'R'),
('en-qwerty','Comma',3,8,1,',','<',6,'R'),
('en-qwerty','Period',3,9,1,'.','>',7,'R'),
('en-qwerty','Slash',3,10,1,'/','?',8,'R'),

-- EN space
('en-qwerty','Space',4,3,6.25,' ',NULL,9,'T'),

-- RU ЙЦУКЕН row 0
('ru-yucuen','Backquote',0,0,1,'ё','Ё',1,'L'),
('ru-yucuen','Digit1',0,1,1,'1','!',1,'L'),
('ru-yucuen','Digit2',0,2,1,'2','"',2,'L'),
('ru-yucuen','Digit3',0,3,1,'3','№',3,'L'),
('ru-yucuen','Digit4',0,4,1,'4',';',4,'L'),
('ru-yucuen','Digit5',0,5,1,'5','%',4,'L'),
('ru-yucuen','Digit6',0,6,1,'6',':',5,'R'),
('ru-yucuen','Digit7',0,7,1,'7','?',5,'R'),
('ru-yucuen','Digit8',0,8,1,'8','*',6,'R'),
('ru-yucuen','Digit9',0,9,1,'9','(',7,'R'),
('ru-yucuen','Digit0',0,10,1,'0',')',8,'R'),
('ru-yucuen','Minus',0,11,1,'-','_',8,'R'),
('ru-yucuen','Equal',0,12,1,'=','+',8,'R'),

-- RU ЙЦУКЕН row 1
('ru-yucuen','KeyQ',1,1,1,'й','Й',1,'L'),
('ru-yucuen','KeyW',1,2,1,'ц','Ц',2,'L'),
('ru-yucuen','KeyE',1,3,1,'у','У',3,'L'),
('ru-yucuen','KeyR',1,4,1,'к','К',4,'L'),
('ru-yucuen','KeyT',1,5,1,'е','Е',4,'L'),
('ru-yucuen','KeyY',1,6,1,'н','Н',5,'R'),
('ru-yucuen','KeyU',1,7,1,'г','Г',5,'R'),
('ru-yucuen','KeyI',1,8,1,'ш','Ш',6,'R'),
('ru-yucuen','KeyO',1,9,1,'щ','Щ',7,'R'),
('ru-yucuen','KeyP',1,10,1,'з','З',8,'R'),
('ru-yucuen','BracketLeft',1,11,1,'х','{',8,'R'),
('ru-yucuen','BracketRight',1,12,1,'ъ','}',8,'R'),
('ru-yucuen','Backslash',1,13,1.5,E'\\','|',8,'R'),

-- RU ЙЦУКЕН row 2
('ru-yucuen','KeyA',2,1,1,'ф','Ф',1,'L'),
('ru-yucuen','KeyS',2,2,1,'ы','Ы',2,'L'),
('ru-yucuen','KeyD',2,3,1,'в','В',3,'L'),
('ru-yucuen','KeyF',2,4,1,'а','А',4,'L'),
('ru-yucuen','KeyG',2,5,1,'п','П',4,'L'),
('ru-yucuen','KeyH',2,6,1,'р','Р',5,'R'),
('ru-yucuen','KeyJ',2,7,1,'о','О',5,'R'),
('ru-yucuen','KeyK',2,8,1,'л','Л',6,'R'),
('ru-yucuen','KeyL',2,9,1,'д','Д',7,'R'),
('ru-yucuen','Semicolon',2,11,1,'ж',':',7,'R'),
('ru-yucuen','Quote',2,12,1,'э','"',8,'R'),

-- RU ЙЦУКЕН row 3
('ru-yucuen','KeyZ',3,1,1,'я','Я',1,'L'),
('ru-yucuen','KeyX',3,2,1,'ч','Ч',2,'L'),
('ru-yucuen','KeyC',3,3,1,'с','С',3,'L'),
('ru-yucuen','KeyV',3,4,1,'м','М',4,'L'),
('ru-yucuen','KeyB',3,5,1,'и','И',4,'L'),
('ru-yucuen','KeyN',3,6,1,'т','Т',5,'R'),
('ru-yucuen','KeyM',3,7,1,'ь','Ь',5,'R'),
('ru-yucuen','Comma',3,8,1,'б',',',6,'R'),
('ru-yucuen','Period',3,9,1,'ю','.',7,'R'),
('ru-yucuen','Slash',3,10,1,'.','/',8,'R'),

-- RU space
('ru-yucuen','Space',4,3,6.25,' ',NULL,9,'T');

-- Insert physical keys
INSERT INTO public.keys (key_code, row_no, col_no, width_units, key_type)
SELECT DISTINCT ON (key_code)
    key_code,
    row_no,
    col_no,
    width_units,
    CASE WHEN key_code = 'Space' THEN 'space' ELSE 'printable' END
FROM tmp_layout_map
ORDER BY key_code, row_no, col_no
ON CONFLICT (key_code) DO UPDATE
SET row_no = EXCLUDED.row_no,
    col_no = EXCLUDED.col_no,
    width_units = EXCLUDED.width_units,
    key_type = EXCLUDED.key_type;

-- Insert unshifted variants
INSERT INTO public.key_variants (layout_id, key_code, char, shift, altgr, finger, hand)
SELECT layout_id, key_code, unshifted, FALSE, FALSE, finger, hand
FROM tmp_layout_map
WHERE unshifted IS NOT NULL AND unshifted <> ''
ON CONFLICT (layout_id, key_code, shift, altgr) DO UPDATE
SET char = EXCLUDED.char,
    finger = EXCLUDED.finger,
    hand = EXCLUDED.hand;

-- Insert shifted variants
INSERT INTO public.key_variants (layout_id, key_code, char, shift, altgr, finger, hand)
SELECT layout_id, key_code, shifted, TRUE, FALSE, finger, hand
FROM tmp_layout_map
WHERE shifted IS NOT NULL AND shifted <> ''
ON CONFLICT (layout_id, key_code, shift, altgr) DO UPDATE
SET char = EXCLUDED.char,
    finger = EXCLUDED.finger,
    hand = EXCLUDED.hand;

DROP TABLE IF EXISTS tmp_layout_map;

-- ============================================================================
-- SEED: levels
-- ============================================================================

INSERT INTO public.levels
    (level, title, xp_required, unlocks_modes, bonus_multiplier, color, icon, description)
VALUES
(1,  'Новичок',              0,     ARRAY['words','symbols']::TEXT[],                              1.00, '#94a3b8', '🌱', 'Базовые слова и простые символы'),
(2,  'Ученик',               100,   ARRAY['words','symbols']::TEXT[],                              1.00, '#60a5fa', '✏️', 'Первые уверенные результаты'),
(3,  'Практикант',           250,   ARRAY['words','sentences','symbols']::TEXT[],                  1.00, '#22d3ee', '📘', 'Открываются предложения'),
(4,  'Оператор',             500,   ARRAY['words','sentences','symbols']::TEXT[],                  1.05, '#34d399', '⌨️', 'Стабильная скорость'),
(5,  'Скоростной',           900,   ARRAY['words','sentences','code','symbols']::TEXT[],           1.05, '#a3e635', '🚀', 'Открывается режим кода'),
(6,  'Виртуоз',              1500,  ARRAY['words','sentences','code','symbols']::TEXT[],           1.10, '#facc15', '✨', 'Высокий темп печати'),
(7,  'Мастер',               2500,  ARRAY['words','sentences','code','symbols']::TEXT[],           1.10, '#fb923c', '🔥', 'Уверенная слепая печать'),
(8,  'Эксперт',              4000,  ARRAY['words','sentences','code','symbols']::TEXT[],           1.15, '#f472b6', '🎯', 'Сложные символы и точность'),
(9,  'Профи',                6500,  ARRAY['words','sentences','code','symbols']::TEXT[],           1.15, '#c084fc', '🏆', 'Профессиональный уровень'),
(10, 'Легенда',              10000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.20, '#22c55e', '👑', 'Все режимы открыты'),
(11, 'Киберпалец',           15000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.25, '#06b6d4', '🤖', 'Быстрые и точные сессии'),
(12, 'Печатный маг',         22000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.30, '#8b5cf6', '🪄', 'Редкие бонусы и комбо'),
(13, 'Виртуоз кода',         32000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.35, '#eab308', '💻', 'Специализация на коде'),
(14, 'Мастер потока',        45000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.40, '#ef4444', '🌊', 'Максимальные серии'),
(15, 'Легенда клавиатуры',   60000, ARRAY['words','sentences','code','symbols']::TEXT[],           1.50, '#ffffff', '🌟', 'Высший уровень тренажёра')
ON CONFLICT (level) DO UPDATE
SET title = EXCLUDED.title,
    xp_required = EXCLUDED.xp_required,
    unlocks_modes = EXCLUDED.unlocks_modes,
    bonus_multiplier = EXCLUDED.bonus_multiplier,
    color = EXCLUDED.color,
    icon = EXCLUDED.icon,
    description = EXCLUDED.description;

-- ============================================================================
-- SEED: achievements
-- ============================================================================

INSERT INTO public.achievements
    (code, title, description, category, condition_type, condition_value,
     xp_reward, score_reward, badge_icon, badge_color, is_hidden, unlock_level, metadata)
VALUES
('first_steps', 'Первые шаги', 'Завершите первую тренировку', 'volume', 'metric_threshold',
 '{"metric":"total_sessions","operator":">=","value":1}'::jsonb,
 10, 100, '🎯', '#22c55e', FALSE, NULL, '{}'::jsonb),

('hundred_chars', 'Сто символов', 'Напечатайте 100 верных символов суммарно', 'volume', 'metric_threshold',
 '{"metric":"correct_chars","operator":">=","value":100}'::jsonb,
 25, 250, '📝', '#60a5fa', FALSE, NULL, '{}'::jsonb),

('steel_finger', 'Стальной палец', '1000 правильных нажатий подряд без ошибок', 'combo', 'metric_threshold',
 '{"metric":"longest_combo","operator":">=","value":1000}'::jsonb,
 250, 2500, '🦾', '#a78bfa', FALSE, NULL, '{}'::jsonb),

('printing_genius', 'Печатный гений', 'Достичь скорости 300 слов в минуту', 'speed', 'metric_threshold',
 '{"metric":"best_wpm","operator":">=","value":300}'::jsonb,
 500, 5000, '🧠', '#facc15', FALSE, NULL, '{}'::jsonb),

('speed_demon', 'Демон скорости', 'Достичь 200 символов в минуту в сессии от 100 символов', 'speed', 'session_condition',
 '{"field":"cpm","operator":">=","value":200,"min_chars":100}'::jsonb,
 150, 1500, '⚡', '#fb923c', FALSE, NULL, '{}'::jsonb),

('laser_focus', 'Лазерный фокус', 'Точность 98% и более при 200+ символах', 'accuracy', 'session_condition',
 '{"field":"accuracy","operator":">=","value":98,"min_chars":200}'::jsonb,
 150, 1500, '🎯', '#22d3ee', FALSE, NULL, '{}'::jsonb),

('flawless_run', 'Безупречный пробег', 'Завершить сессию на 100+ символов без ошибок', 'accuracy', 'session_condition',
 '{"field":"incorrect_chars","operator":"=","value":0,"min_chars":100}'::jsonb,
 100, 1000, '💎', '#34d399', FALSE, NULL, '{}'::jsonb),

('combo_master', 'Мастер комбо', 'Достичь множителя x5: 50 правильных нажатий подряд', 'combo', 'metric_threshold',
 '{"metric":"longest_combo","operator":">=","value":50}'::jsonb,
 75, 750, '🔥', '#ef4444', FALSE, NULL, '{}'::jsonb),

('code_ninja', 'Кодовый ниндзя', 'Завершить 10 тренировок в режиме Код', 'mode', 'count_condition',
 '{"mode_code":"code","operator":">=","value":10}'::jsonb,
 200, 2000, '🥷', '#8b5cf6', FALSE, NULL, '{}'::jsonb),

('polyglot', 'Полиглот', 'Завершить тренировки на русском и английском языках', 'mode', 'multi_mode_condition',
 '{"languages":["ru","en"],"min_sessions_each":1}'::jsonb,
 100, 1000, '🌍', '#06b6d4', FALSE, NULL, '{}'::jsonb),

('marathoner', 'Марафонец', 'Накопить 10 часов общего времени тренировок', 'volume', 'metric_threshold',
 '{"metric":"total_time_seconds","operator":">=","value":36000}'::jsonb,
 300, 3000, '🏃', '#f59e0b', FALSE, NULL, '{}'::jsonb),

('weekly_streak', 'Серия недели', 'Тренироваться 7 дней подряд', 'special', 'metric_threshold',
 '{"metric":"streak_days","operator":">=","value":7}'::jsonb,
 175, 1750, '📅', '#ec4899', FALSE, NULL, '{}'::jsonb),

('symbol_slayer', 'Покоритель символов', 'Сессия в режиме Symbols: 150+ символов и точность 95%+', 'mode', 'session_condition',
 '{"mode_code":"symbols","field":"accuracy","operator":">=","value":95,"min_chars":150}'::jsonb,
 120, 1200, '⚔️', '#94a3b8', FALSE, NULL, '{}'::jsonb),

('level_10', 'Десятый уровень', 'Достичь 10 уровня', 'special', 'metric_threshold',
 '{"metric":"level","operator":">=","value":10}'::jsonb,
 250, 2500, '👑', '#facc15', FALSE, NULL, '{}'::jsonb)
ON CONFLICT (code) DO UPDATE
SET title = EXCLUDED.title,
    description = EXCLUDED.description,
    category = EXCLUDED.category,
    condition_type = EXCLUDED.condition_type,
    condition_value = EXCLUDED.condition_value,
    xp_reward = EXCLUDED.xp_reward,
    score_reward = EXCLUDED.score_reward,
    badge_icon = EXCLUDED.badge_icon,
    badge_color = EXCLUDED.badge_color,
    is_hidden = EXCLUDED.is_hidden,
    unlock_level = EXCLUDED.unlock_level,
    metadata = EXCLUDED.metadata;

-- ============================================================================
-- SEED: bonus definitions
-- ============================================================================

INSERT INTO public.bonus_definitions
    (code, name, description, effect_type, effect_value, duration_seconds,
     trigger_chance, cooldown_seconds, max_per_session, min_level,
     icon, color, is_enabled, metadata)
VALUES
('double_xp_10s', 'Двойной опыт', 'Удваивает получаемый XP на 10 секунд', 'xp_multiplier',
 '{"multiplier":2}'::jsonb, 10, 0.0800, 30, 1, 1,
 '⚡', '#facc15', TRUE, '{}'::jsonb),

('score_surge_15s', 'Всплеск очков', 'Увеличивает очки в 1.5 раза на 15 секунд', 'score_multiplier',
 '{"multiplier":1.5}'::jsonb, 15, 0.0700, 45, 1, 1,
 '🔥', '#fb923c', TRUE, '{}'::jsonb),

('speed_boost_20s', 'Ускорение', 'Даёт +50% к бонусной скорости для начисления XP/очков на 20 секунд', 'cpm_multiplier',
 '{"multiplier":1.5}'::jsonb, 20, 0.0500, 60, 1, 3,
 '🚀', '#60a5fa', TRUE, '{}'::jsonb),

('error_shield_10s', 'Щит ошибок', 'Ошибки не сбрасывают комбо и могут не учитываться, до 5 ошибок за 10 секунд', 'error_shield',
 '{"ignore_combo_reset":true,"max_errors":5,"suppress_error_count":true}'::jsonb, 10, 0.0600, 90, 1, 2,
 '🛡️', '#34d399', TRUE, '{}'::jsonb),

('combo_keep_15s', 'Сохранение комбо', 'Одна ошибка не прерывает серию комбо', 'combo_keep',
 '{"preserve_one_error":true}'::jsonb, 15, 0.0400, 120, 1, 5,
 '🧲', '#a78bfa', TRUE, '{}'::jsonb),

('xp_injection', 'Инъекция опыта', 'Мгновенно начисляет 100 XP', 'instant_xp',
 '{"amount":100}'::jsonb, 0, 0.0300, 180, 1, 1,
 '💉', '#22d3ee', TRUE, '{}'::jsonb)
ON CONFLICT (code) DO UPDATE
SET name = EXCLUDED.name,
    description = EXCLUDED.description,
    effect_type = EXCLUDED.effect_type,
    effect_value = EXCLUDED.effect_value,
    duration_seconds = EXCLUDED.duration_seconds,
    trigger_chance = EXCLUDED.trigger_chance,
    cooldown_seconds = EXCLUDED.cooldown_seconds,
    max_per_session = EXCLUDED.max_per_session,
    min_level = EXCLUDED.min_level,
    icon = EXCLUDED.icon,
    color = EXCLUDED.color,
    is_enabled = EXCLUDED.is_enabled,
    metadata = EXCLUDED.metadata;

-- ============================================================================
-- SEED: words
-- ============================================================================

INSERT INTO public.words (language_code, word, frequency, difficulty)
SELECT 'en', t.word, 100, 1
FROM unnest(ARRAY[
    'the','be','to','of','and','a','in','that','have','it',
    'for','not','on','with','he','as','you','do','at','this',
    'but','his','by','from','they','we','say','her','she','or',
    'an','will','my','one','all','would','there','their','what','so',
    'up','out','if','about','who','get','which','go','me','use'
]) AS t(word)
ON CONFLICT (language_code, word) DO UPDATE
SET frequency = EXCLUDED.frequency,
    difficulty = EXCLUDED.difficulty;

INSERT INTO public.words (language_code, word, frequency, difficulty)
SELECT 'ru', t.word, 100, 1
FROM unnest(ARRAY[
    'и','в','во','что','не','он','на','я','с','со',
    'как','а','то','все','она','так','его','но','да','ты',
    'к','у','же','вы','за','бы','по','только','её','мне',
    'было','уже','или','ни','быть','они','это','мы','вас','год'
]) AS t(word)
ON CONFLICT (language_code, word) DO UPDATE
SET frequency = EXCLUDED.frequency,
    difficulty = EXCLUDED.difficulty;

-- ============================================================================
-- SEED: sentences
-- ============================================================================

INSERT INTO public.sentences (language_code, text, difficulty)
SELECT 'en', t.text, 2
FROM unnest(ARRAY[
    'The quick brown fox jumps over the lazy dog.',
    'Practice makes perfect when you type every day.',
    'Silent words hide powerful meaning in simple letters.',
    'Keyboard shortcuts can save a lot of time.',
    'Focus on rhythm rather than raw speed first.',
    'Accuracy builds confidence and speed follows naturally.',
    'Developers read code more often than they write it.',
    'A good habit is worth more than a brilliant one.'
]) AS t(text)
ON CONFLICT (language_code, text) DO UPDATE
SET difficulty = EXCLUDED.difficulty;

INSERT INTO public.sentences (language_code, text, difficulty)
SELECT 'ru', t.text, 2
FROM unnest(ARRAY[
    'Быстрая коричневая лиса прыгает через ленивого пса.',
    'Практика каждый день делает печать уверенной.',
    'Сначала точность, потом скорость — главный принцип.',
    'Клавиатурные сокращения экономят много времени.',
    'Слепая печать освобождает взгляд для мышления.',
    'Ритм важнее рывков в начале тренировки.',
    'Развивайте мышцы памяти, а не только пальцы.',
    'Код читают чаще, чем пишут, поэтому важна ясность.'
]) AS t(text)
ON CONFLICT (language_code, text) DO UPDATE
SET difficulty = EXCLUDED.difficulty;

-- ============================================================================
-- SEED: code snippets
-- ============================================================================

INSERT INTO public.code_snippets (programming_language, code, difficulty)
VALUES
('python', E'def hello(name):\n    return f"Hello, {name}!"', 2),
('python', E'items = [x * 2 for x in range(10)]', 2),
('python', E'if score >= 100:\n    level += 1', 1),
('javascript', E'const sum = arr.reduce((a, b) => a + b, 0);', 3),
('javascript', E'document.addEventListener("click", () => {});', 2),
('javascript', E'async function fetchData(url) {\n  const res = await fetch(url);\n  return res.json();\n}', 4),
('html_css', E'<div class="card">\n  <h1>Title</h1>\n</div>', 2),
('html_css', E'.button { color: #fff; background: #000; }', 2),
('html_css', E'<input type="text" placeholder="Type here" />', 1)
ON CONFLICT (programming_language, code) DO UPDATE
SET difficulty = EXCLUDED.difficulty;

-- ============================================================================
-- SEED: symbol sets
-- ============================================================================

INSERT INTO public.symbol_sets (name, chars, difficulty, language_code)
VALUES
('Basic punctuation', '.,!?;:', 1, NULL),
('Digits and symbols', '0123456789!@#$%^&*()', 2, NULL),
('Code symbols', '{}[]();:=<>+-*/&|!?#$', 3, NULL),
('Russian uppercase', 'АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ', 2, 'ru'),
('English uppercase', 'ABCDEFGHIJKLMNOPQRSTUVWXYZ', 2, 'en'),
('Mixed hard', 'QZXJVK#$_&@!1098', 3, NULL)
ON CONFLICT (name) DO UPDATE
SET chars = EXCLUDED.chars,
    difficulty = EXCLUDED.difficulty,
    language_code = EXCLUDED.language_code;

-- ============================================================================
-- Functions: gamification helpers
-- ============================================================================

CREATE OR REPLACE FUNCTION public.fn_combo_multiplier(p_combo INTEGER)
RETURNS NUMERIC
LANGUAGE sql
IMMUTABLE
PARALLEL SAFE
AS $$
    SELECT CASE
        WHEN COALESCE(p_combo, 0) >= 50 THEN 5.0
        WHEN COALESCE(p_combo, 0) >= 25 THEN 3.0
        WHEN COALESCE(p_combo, 0) >= 10 THEN 2.0
        ELSE 1.0
    END::NUMERIC;
$$;

CREATE OR REPLACE FUNCTION public.fn_calculate_level(p_total_xp BIGINT)
RETURNS SMALLINT
LANGUAGE sql
STABLE
PARALLEL SAFE
AS $$
    SELECT COALESCE(
        (
            SELECT level
            FROM public.levels
            WHERE xp_required <= COALESCE(p_total_xp, 0)
            ORDER BY level DESC
            LIMIT 1
        ),
        1
    )::SMALLINT;
$$;

CREATE OR REPLACE FUNCTION public.fn_session_accuracy(p_correct INTEGER, p_incorrect INTEGER)
RETURNS NUMERIC
LANGUAGE sql
IMMUTABLE
PARALLEL SAFE
AS $$
    SELECT CASE
        WHEN COALESCE(p_correct, 0) + COALESCE(p_incorrect, 0) = 0 THEN 0.00
        ELSE ROUND(100.0 * p_correct / (p_correct + p_incorrect), 2)
    END::NUMERIC;
$$;

-- ============================================================================
-- Function: leaderboard
-- ============================================================================

CREATE OR REPLACE FUNCTION public.fn_leaderboard(
    p_metric TEXT DEFAULT 'xp',
    p_period TEXT DEFAULT 'all',
    p_limit INTEGER DEFAULT 100
)
RETURNS TABLE (
    user_id UUID,
    display_name TEXT,
    value NUMERIC,
    rank BIGINT
)
LANGUAGE plpgsql
STABLE
AS $$
DECLARE
    v_start TIMESTAMPTZ;
BEGIN
    p_metric := lower(p_metric);
    p_period := lower(p_period);

    IF p_metric NOT IN ('wpm', 'cpm', 'accuracy', 'xp') THEN
        RAISE EXCEPTION 'Unsupported metric: %', p_metric;
    END IF;

    IF p_period NOT IN ('day', 'week', 'month', 'all') THEN
        RAISE EXCEPTION 'Unsupported period: %', p_period;
    END IF;

    IF p_limit IS NULL OR p_limit < 1 THEN
        p_limit := 100;
    END IF;

    IF p_limit > 500 THEN
        p_limit := 500;
    END IF;

    IF p_period = 'day' THEN
        v_start := date_trunc('day', now());
    ELSIF p_period = 'week' THEN
        v_start := date_trunc('week', now());
    ELSIF p_period = 'month' THEN
        v_start := date_trunc('month', now());
    ELSE
        v_start := NULL;
    END IF;

    RETURN QUERY
    WITH base AS (
        SELECT
            s.user_id,
            COALESCE(u.display_name, split_part(u.email::text, '@', 1)) AS display_name,
            s.wpm,
            s.cpm,
            s.accuracy,
            s.xp_earned
        FROM public.sessions s
        JOIN public.users u ON u.id = s.user_id
        WHERE s.status = 'completed'
          AND s.user_id IS NOT NULL
          AND (v_start IS NULL OR s.ended_at >= v_start)
    ),
    agg AS (
        SELECT
            user_id,
            display_name,
            CASE p_metric
                WHEN 'xp' THEN SUM(xp_earned)::NUMERIC
                WHEN 'wpm' THEN MAX(wpm)::NUMERIC
                WHEN 'cpm' THEN MAX(cpm)::NUMERIC
                WHEN 'accuracy' THEN MAX(accuracy)::NUMERIC
            END AS value
        FROM base
        GROUP BY user_id, display_name
    )
    SELECT
        agg.user_id,
        agg.display_name,
        agg.value,
        RANK() OVER (ORDER BY agg.value DESC) AS rank
    FROM agg
    ORDER BY 4, 3 DESC
    LIMIT p_limit;
END;
$$;

-- ============================================================================
-- Views
-- ============================================================================

CREATE OR REPLACE VIEW public.v_daily_user_stats AS
SELECT
    s.user_id,
    (s.ended_at AT TIME ZONE 'UTC')::date AS stat_date,
    COUNT(*) AS sessions,
    COALESCE(SUM(s.correct_chars), 0) AS correct_chars,
    COALESCE(SUM(s.incorrect_chars), 0) AS incorrect_chars,
    ROUND(AVG(s.wpm), 2) AS avg_wpm,
    ROUND(AVG(s.cpm), 2) AS avg_cpm,
    ROUND(AVG(s.accuracy), 2) AS avg_accuracy,
    COALESCE(SUM(s.xp_earned), 0) AS xp_earned
FROM public.sessions s
WHERE s.status = 'completed'
  AND s.user_id IS NOT NULL
  AND s.ended_at IS NOT NULL
GROUP BY 1, 2;

CREATE OR REPLACE VIEW public.v_character_difficulty AS
SELECT
    cs.user_id,
    cs.char,
    cs.attempts,
    cs.correct,
    cs.incorrect,
    CASE
        WHEN cs.attempts > 0 THEN ROUND(100.0 * cs.incorrect / cs.attempts, 2)
        ELSE 0
    END AS error_rate,
    cs.avg_delay_ms,
    cs.last_seen_at
FROM public.character_stats cs
WHERE cs.attempts >= 5;

-- ============================================================================
-- Triggers: automatic user progress and settings
-- ============================================================================

CREATE OR REPLACE FUNCTION public.fn_user_after_insert()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
    INSERT INTO public.user_settings (user_id)
    VALUES (NEW.id)
    ON CONFLICT (user_id) DO NOTHING;

    INSERT INTO public.user_progress (user_id, level, total_xp)
    VALUES (NEW.id, public.fn_calculate_level(0), 0)
    ON CONFLICT (user_id) DO NOTHING;

    RETURN NEW;
END;
$$;

CREATE OR REPLACE TRIGGER trg_users_after_insert
AFTER INSERT ON public.users
FOR EACH ROW
EXECUTE FUNCTION public.fn_user_after_insert();

CREATE OR REPLACE FUNCTION public.fn_user_progress_before_update()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.level = public.fn_calculate_level(NEW.total_xp);
    NEW.updated_at = now();
    RETURN NEW;
END;
$$;

CREATE OR REPLACE TRIGGER trg_user_progress_before_update
BEFORE UPDATE ON public.user_progress
FOR EACH ROW
EXECUTE FUNCTION public.fn_user_progress_before_update();
```

---

# 9. Примеры бизнес-логики на Python

## 9.1 Расчёт комбо-множителя

```python
def combo_multiplier(combo: int) -> float:
    if combo >= 50:
        return 5.0
    if combo >= 25:
        return 3.0
    if combo >= 10:
        return 2.0
    return 1.0
```

---

## 9.2 Расчёт XP за правильный символ

```python
def calculate_xp_delta(
    combo: int,
    level_multiplier: float,
    active_bonus_xp_multiplier: float = 1.0,
) -> int:
    base_xp = 1
    multiplier = combo_multiplier(combo)
    xp = base_xp * multiplier * level_multiplier * active_bonus_xp_multiplier
    return max(1, round(xp))
```

---

## 9.3 Расчёт точности

```python
def calculate_accuracy(correct: int, incorrect: int) -> float:
    total = correct + incorrect
    if total == 0:
        return 0.0
    return round(correct / total * 100, 2)
```

---

## 9.4 Расчёт WPM / CPM

```python
def calculate_cpm(correct_chars: int, duration_seconds: int) -> float:
    if duration_seconds <= 0:
        return 0.0
    minutes = duration_seconds / 60.0
    return round(correct_chars / minutes, 2)


def calculate_wpm(correct_chars: int, duration_seconds: int) -> float:
    cpm = calculate_cpm(correct_chars, duration_seconds)
    return round(cpm / 5.0, 2)
```

---

## 9.5 Проверка достижения

Пример для `metric_threshold`:

```python
def check_metric_threshold(progress: dict, condition: dict) -> bool:
    metric = condition["metric"]
    operator = condition["operator"]
    value = condition["value"]

    actual = progress.get(metric, 0)

    if operator == ">=":
        return actual >= value
    if operator == ">":
        return actual > value
    if operator == "=":
        return actual == value
    if operator == "<=":
        return actual <= value
    if operator == "<":
        return actual < value

    raise ValueError(f"Unsupported operator: {operator}")
```

---

# 10. Нефункциональные требования

## 10.1 Производительность

- Первый рендер тренировки — до 1.5 секунд на обычном ноутбуке.
- Обработка `keydown` не должна вызывать лаги.
- Обновление DOM лучше делать точечно, не перерисовывать весь текст на каждый символ.
- Использовать `requestAnimationFrame` для анимаций.
- Бэкапить отправку событий пакетами, например каждые 500 мс или 50 событий.

## 10.2 Надёжность

- События ввода должны быть идемпотентными.
- При потере сети клиент хранит очередь событий локально.
- Backend должен корректно обрабатывать повторную отправку тех же `seq`.
- Сессия не должна теряться при перезагрузке страницы в течение короткого времени.

## 10.3 Безопасность

- Пароли только в Argon2id / bcrypt.
- JWT подписывать HS256 или RS256.
- Refresh token хранить hashed.
- Rate limiting:
  - login: 5–10 попыток в минуту;
  - register: 3 попытки в час по IP;
  - session events: разумный лимит на размер пакета.
- Проверять принадлежность сессии пользователю.
- Не доверять клиентским итоговым цифрам без пересчёта.

## 10.4 Масштабируемость

- Таблица `session_events` может расти быстро. Для продакшена рассмотреть:
  - партиционирование по месяцу;
  - архивирование старых событий;
  - хранение агрегатов отдельно.
- Лидерборд кэшировать.
- Словари держать в памяти / Redis.

---

# 11. Тестирование

## Front-end

- Unit-тесты для:
  - расчёта комбо;
  - сравнения символов;
  - таймера;
  - обработки ошибок.
- Интеграционные тесты:
  - старт сессии;
  - ввод текста;
  - завершение сессии.
- E2E через Playwright:
  - регистрация;
  - тренировка слов;
  - появление ошибки;
  - открытие достижения;
  - просмотр лидерборда.

## Back-end

- Pytest.
- Testcontainers для PostgreSQL.
- Покрыть:
  - auth;
  - создание сессии;
  - приём событий;
  - завершение сессии;
  - начисление XP;
  - активацию бонусов;
  - выдачу достижений;
  - лидерборд.

---

# 12. Этапы разработки

## Этап 1. Каркас

- Настроить repo.
- Docker compose: PostgreSQL, backend, frontend.
- Alembic baseline.
- Базовые страницы.
- Healthcheck API.

## Этап 2. Аутентификация

- Регистрация.
- Вход.
- JWT.
- Refresh token.
- Профиль пользователя.

## Этап 3. Тренажёр без геймификации

- Генерация упражнений.
- Экран печати.
- Обработка ввода.
- Подсветка символов.
- Виртуальная клавиатура.
- Сохранение сессии.

## Этап 4. Геймификация

- Очки.
- Комбо.
- XP.
- Уровни.
- Достижения.
- Бонусы.

## Этап 5. Статистика и лидерборд

- Общая статистика.
- Графики.
- Анализ сложных символов.
- Таблица лидеров.

## Этап 6. Полировка

- Анимации.
- Звуки.
- Адаптивность.
- Доступность.
- Тесты.
- Деплой.

---

# 13. Definition of Done для MVP

Приложение считается готовым, если:

1. Пользователь может зарегистрироваться и войти.
2. Пользователь может выбрать режим, язык и начать тренировку.
3. Текст отображается посимвольно.
4. Правильный и неправильный ввод различаются визуально.
5. Виртуальная клавиатура подсвечивает нужную клавишу и палец.
6. Комбо начисляется и сбрасывается по правилам.
7. XP и уровни сохраняются в базе.
8. Хотя бы 10 достижений работают.
9. Хотя бы 5 бонусов активируются во время тренировки.
10. Статистика сессии отображается в реальном времени.
11. Общая статистика сохраняется после завершения тренировки.
12. Таблица лидеров показывает топ по WPM, CPM, accuracy, XP.
13. Данные не теряются при перезагрузке страницы для авторизованного пользователя.
14. Backend не позволяет фальсифицировать итоговые результаты без проверки.
15. Основные user flows покрыты тестами.

---

# 14. Итоговая рекомендация по реализации

Для быстрого MVP лучше сделать так:

- Front-end: vanilla JS + модульная архитектура, без тяжёлого фреймворка.
- Back-end: FastAPI + SQLAlchemy + PostgreSQL.
- Игровая логика: на сервере, клиент только отображает и отправляет сырые события.
- Бонусы и достижения: конфигурируются через БД, чтобы можно было балансировать без релиза.
- Лидерборд: сначала SQL-функция `fn_leaderboard`, позже кэш в Redis.
- Словари: начать с небольшого стартового набора, затем дополнять через seed / admin panel.

Такой подход позволит быстро запустить рабочий тренажёр и постепенно усиливать геймификацию без переписывания архитектуры.