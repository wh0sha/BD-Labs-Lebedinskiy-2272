# Laborator_BD
# Лабораторные работы по Базам данных
**Студент:** Лебединский Илья  
**Группа:** 2272  
**Тема:** Система управления задачами (Task Management System)  
**Цель:** Спроектировать, реализовать и оптимизировать базу данных для управления проектами, задачами, пользователями и напоминаниями, нормализованную до 3NF.

## Лабораторная работа 1. Проектирование структуры БД
![ER-диаграмма](https://github.com/yourusername/Laborator_BD/raw/main/er-diagram.png)

### Нормальные формы
- **1NF (Первая нормальная форма):**  
&nbsp;&nbsp;Соблюдена. Все атрибуты атомарны, строки уникальны, каждая таблица имеет первичный ключ.
- **2NF (Вторая нормальная форма):**  
&nbsp;&nbsp;Соблюдена. Все неключевые атрибуты полностью зависят от всего первичного ключа.
- **3NF (Третья нормальная форма):**  
&nbsp;&nbsp;Соблюдена. Транзитивные зависимости между неключевыми атрибутами отсутствуют.

## Лабораторная работа 2. Инсталляция БД на сервере
### Создание таблиц (в правильном порядке)
```sql
-- Создание схемы (если не существует)
CREATE SCHEMA IF NOT EXISTS "Lebedinskiy_2272";

-- Таблица users (Пользователи)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    registration_date DATE DEFAULT CURRENT_DATE,
    password_hash VARCHAR(255) NOT NULL
);

-- Таблица priority (Приоритеты задач)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".priority (
    priority_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

-- Таблица status (Статусы задач)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".status (
    status_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

-- Таблица projects (Проекты)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".projects (
    project_id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".users(user_id) ON DELETE CASCADE,
    project_name VARCHAR(255) NOT NULL,
    description VARCHAR(255),
    creation_date DATE DEFAULT CURRENT_DATE
);

-- Таблица tasks (Задачи)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".tasks (
    task_id SERIAL PRIMARY KEY,
    project_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".projects(project_id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".users(user_id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description VARCHAR(255),
    status_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".status(status_id) ON DELETE RESTRICT,
    priority_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".priority(priority_id) ON DELETE RESTRICT,
    due_date DATE,
    completed_date DATE,
    creation_date DATE DEFAULT CURRENT_DATE
);

-- Таблица reminders (Напоминания)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".reminders (
    reminder_id SERIAL PRIMARY KEY,
    task_id INTEGER NOT NULL REFERENCES "Lebedinskiy_2272".tasks(task_id) ON DELETE CASCADE,
    reminder_time TIMESTAMP NOT NULL,
    type VARCHAR(50),
    status VARCHAR(50)
);
```

### Заполнение данными
```sql
-- Заполняем справочники
INSERT INTO "Lebedinskiy_2272".priority (name) VALUES
('Низкий'), ('Средний'), ('Высокий'), ('Критический')
ON CONFLICT (name) DO NOTHING;

INSERT INTO "Lebedinskiy_2272".status (name) VALUES
('Новая'), ('В работе'), ('Завершена'), ('Отложена')
ON CONFLICT (name) DO NOTHING;

-- Заполняем пользователей
INSERT INTO "Lebedinskiy_2272".users (name, email, password_hash) VALUES
('Илья', 'ilya@example.com', 'hash123'),
('Анна', 'anna@example.com', 'hash456'),
('Сергей', 'sergey@example.com', 'hash789'),
('Полина', 'polina@example.com', 'hash1011')
ON CONFLICT (email) DO NOTHING;

-- Заполняем проекты
INSERT INTO "Lebedinskiy_2272".projects (user_id, project_name, description) VALUES
(1, 'Личный проект', 'Мои повседневные дела'),
(2, 'Рабочий проект', 'Разработка приложения'),
(3, 'Учебный проект', 'Курсовая работа'),
(4, 'Хобби проект', 'Игровой сервер');

-- Заполняем задачи
INSERT INTO "Lebedinskiy_2272".tasks (project_id, user_id, title, description, status_id, priority_id, due_date) VALUES
(1, 1, 'Купить продукты', 'Молоко, хлеб, яйца', 1, 2, '2026-01-20'),
(2, 2, 'Сдать отчет', 'Подготовить презентацию', 2, 3, '2026-01-25'),
(3, 3, 'Написать код', 'Реализовать авторизацию', 1, 4, '2026-01-22'),
(4, 4, 'Настроить сервер', 'Установить моды', 3, 1, '2026-01-18');

-- Заполняем напоминания
INSERT INTO "Lebedinskiy_2272".reminders (task_id, reminder_time, type, status) VALUES
(1, '2026-01-19 18:00:00', 'Push', 'Активно'),
(2, '2026-01-24 09:00:00', 'Email', 'Активно'),
(3, '2026-01-21 14:00:00', 'SMS', 'Активно'),
(4, '2026-01-17 10:00:00', 'In-app', 'Отправлено');
```

### Содержательный SELECT запрос с JOIN (3+ таблицы)
```sql
SELECT
  u.name AS "пользователь",
  p.project_name AS "проект",
  t.title AS "задача",
  s.name AS "статус",
  pr.name AS "приоритет",
  t.due_date AS "срок"
FROM "Lebedinskiy_2272".tasks t
JOIN "Lebedinskiy_2272".users u ON t.user_id = u.user_id
JOIN "Lebedinskiy_2272".projects p ON t.project_id = p.project_id
JOIN "Lebedinskiy_2272".status s ON t.status_id = s.status_id
JOIN "Lebedinskiy_2272".priority pr ON t.priority_id = pr.priority_id
WHERE t.due_date >= CURRENT_DATE
ORDER BY t.due_date;
```

## Лабораторная работа 3. Представления и процедуры
### Создание представления
```sql
CREATE OR REPLACE VIEW "Lebedinskiy_2272".vw_full_task_info AS
SELECT
    u.name AS "пользователь",
    p.project_name AS "проект",
    t.title AS "задача",
    t.description AS "описание",
    s.name AS "статус",
    pr.name AS "приоритет",
    t.due_date AS "срок",
    t.completed_date AS "завершено",
    r.reminder_time AS "время_напоминания",
    r.type AS "тип_напоминания"
FROM "Lebedinskiy_2272".tasks t
JOIN "Lebedinskiy_2272".users u ON t.user_id = u.user_id
JOIN "Lebedinskiy_2272".projects p ON t.project_id = p.project_id
JOIN "Lebedinskiy_2272".status s ON t.status_id = s.status_id
JOIN "Lebedinskiy_2272".priority pr ON t.priority_id = pr.priority_id
LEFT JOIN "Lebedinskiy_2272".reminders r ON r.task_id = t.task_id;
```

**Назначение представления**  
Представление объединяет данные из 6 таблиц для удобного просмотра полной информации по задачам, включая напоминания.

**Использование представления**
```sql
SELECT * FROM "Lebedinskiy_2272".vw_full_task_info LIMIT 10;
```

### Создание функции (сортировка задач по приоритету)
```sql
CREATE OR REPLACE FUNCTION "Lebedinskiy_2272".get_tasks_sorted_by_priority()
RETURNS TABLE (
    "пользователь" TEXT,
    "проект" TEXT,
    "задача" TEXT,
    "статус" TEXT,
    "приоритет" TEXT,
    "срок" DATE
)
LANGUAGE SQL
AS $$
    SELECT
        "пользователь",
        "проект",
        "задача",
        "статус",
        "приоритет",
        "срок"
    FROM "Lebedinskiy_2272".vw_full_task_info
    ORDER BY 
        CASE "приоритет"
            WHEN 'Критический' THEN 1
            WHEN 'Высокий' THEN 2
            WHEN 'Средний' THEN 3
            WHEN 'Низкий' THEN 4
            ELSE 5
        END,
        "срок" ASC;
$$;
```

**Назначение функции**  
Возвращает список задач, отсортированных по убыванию приоритета и сроку выполнения.

**Использование функции**
```sql
SELECT * FROM "Lebedinskiy_2272".get_tasks_sorted_by_priority();
```

## Лабораторная работа 4. Анализ производительности
### Добавление дополнительных пользователей и проектов для тестирования
```sql
-- Добавляем больше пользователей
INSERT INTO "Lebedinskiy_2272".users (name, email, password_hash)
SELECT 
    'Пользователь ' || i,
    'user' || i || '@example.com',
    md5(random()::text)
FROM generate_series(5, 100) AS i
ON CONFLICT (email) DO NOTHING;

-- Добавляем больше проектов
INSERT INTO "Lebedinskiy_2272".projects (user_id, project_name, description)
SELECT 
    (random() * 96 + 1)::INT + 1,  -- Выбираем случайного пользователя из 1-100
    'Проект ' || i,
    'Тестовый проект #' || i
FROM generate_series(5, 50) AS i;
```

### Генерация тестовых данных (20 000 записей)
```sql
INSERT INTO "Lebedinskiy_2272".tasks (
    project_id, user_id, title, description, 
    status_id, priority_id, due_date, creation_date
)
SELECT
    (SELECT project_id FROM "Lebedinskiy_2272".projects ORDER BY random() LIMIT 1),
    (SELECT user_id FROM "Lebedinskiy_2272".users ORDER BY random() LIMIT 1),
    'Автозадача #' || gs.i,
    'Тестовое описание для задачи #' || gs.i,
    (SELECT status_id FROM "Lebedinskiy_2272".status ORDER BY random() LIMIT 1),
    (SELECT priority_id FROM "Lebedinskiy_2272".priority ORDER BY random() LIMIT 1),
    CURRENT_DATE + (random() * 180)::INT,
    CURRENT_DATE - (random() * 365)::INT
FROM generate_series(1, 20000) AS gs(i);
```

### Анализ плана выполнения
```sql
EXPLAIN ANALYZE
SELECT
    pr.name AS "приоритет",
    COUNT(*) AS "кол_во_задач",
    AVG(t.due_date - t.creation_date) AS "средний_срок_дней"
FROM "Lebedinskiy_2272".tasks t
JOIN "Lebedinskiy_2272".priority pr ON t.priority_id = pr.priority_id
WHERE t.creation_date >= CURRENT_DATE - INTERVAL '90 days'
GROUP BY pr.name
ORDER BY "кол_во_задач" DESC;
```

### Создание индексов для оптимизации
```sql
CREATE INDEX CONCURRENTLY idx_tasks_creation_date ON "Lebedinskiy_2272".tasks (creation_date);
CREATE INDEX CONCURRENTLY idx_tasks_priority_id ON "Lebedinskiy_2272".tasks (priority_id);
CREATE INDEX CONCURRENTLY idx_tasks_due_date ON "Lebedinskiy_2272".tasks (due_date);
CREATE INDEX CONCURRENTLY idx_reminders_time ON "Lebedinskiy_2272".reminders (reminder_time);
```

## Лабораторная работа 5. Триггеры и аудит
### Таблица журнала аудита
```sql
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".audit (
    audit_id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL CHECK (operation IN ('INSERT','UPDATE','DELETE','DELETE (cascade)')),
    record_id TEXT,  -- Изменено с INT на TEXT для поддержки UUID и других типов
    old_values JSONB,
    new_values JSONB,
    changed_by TEXT NOT NULL DEFAULT CURRENT_USER,
    changed_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX CONCURRENTLY idx_audit_table ON "Lebedinskiy_2272".audit (table_name);
CREATE INDEX CONCURRENTLY idx_audit_operation ON "Lebedinskiy_2272".audit (operation);
CREATE INDEX CONCURRENTLY idx_audit_time ON "Lebedinskiy_2272".audit (changed_at);
```

### Универсальная функция аудита
```sql
CREATE OR REPLACE FUNCTION "Lebedinskiy_2272".audit_log_universal()
RETURNS TRIGGER AS $$
DECLARE
    pk_column_name TEXT;
    pk_value TEXT;
BEGIN
    -- Получаем имя первичного ключа таблицы
    SELECT a.attname INTO pk_column_name
    FROM pg_index i 
    JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
    WHERE i.indrelid = TG_RELID AND i.indisprimary 
    LIMIT 1;

    IF pk_column_name IS NULL THEN
        RAISE EXCEPTION 'Primary key not found for table %', TG_TABLE_NAME;
    END IF;

    -- Получаем значение первичного ключа
    IF TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN
        EXECUTE format('SELECT $1.%I::TEXT', pk_column_name) USING NEW INTO pk_value;
    ELSE
        EXECUTE format('SELECT $1.%I::TEXT', pk_column_name) USING OLD INTO pk_value;
    END IF;

    -- Вставляем запись в аудит
    INSERT INTO "Lebedinskiy_2272".audit (
        table_name, operation, record_id, old_values, new_values, changed_by
    ) VALUES (
        TG_TABLE_NAME,
        TG_OP,
        pk_value,
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN to_jsonb(OLD) END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) END,
        CURRENT_USER
    );
    
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

### Триггеры аудита для всех основных таблиц
```sql
-- Триггеры для основных таблиц
CREATE TRIGGER trg_audit_users
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".users
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();

CREATE TRIGGER trg_audit_projects
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".projects
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();

CREATE TRIGGER trg_audit_tasks
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".tasks
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();

CREATE TRIGGER trg_audit_reminders
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".reminders
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();
```

### Проверка аудита
```sql
-- Тестовые операции
INSERT INTO "Lebedinskiy_2272".projects (user_id, project_name) 
VALUES (1, 'Тестовый проект');

INSERT INTO "Lebedinskiy_2272".tasks (project_id, user_id, title, status_id, priority_id, due_date)
VALUES (
    currval('"Lebedinskiy_2272".projects_project_id_seq'), 
    1, 
    'Тестовая задача', 
    1, 
    1, 
    CURRENT_DATE + 1
);

INSERT INTO "Lebedinskiy_2272".reminders (task_id, reminder_time, type, status)
VALUES (
    currval('"Lebedinskiy_2272".tasks_task_id_seq'), 
    CURRENT_TIMESTAMP + INTERVAL '1 day', 
    'Push', 
    'Активно'
);

UPDATE "Lebedinskiy_2272".tasks 
SET title = 'Обновленная задача' 
WHERE task_id = currval('"Lebedinskiy_2272".tasks_task_id_seq');

DELETE FROM "Lebedinskiy_2272".projects 
WHERE project_name = 'Тестовый проект';

-- Просмотр записей аудита
SELECT * FROM "Lebedinskiy_2272".audit 
ORDER BY changed_at DESC 
LIMIT 10;
```
