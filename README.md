# Laborator_BD
# Лабораторные работы по Базам данных
**Студент:** Лебединский Илья
**Группа:** 2272
**Тема:** Система управления задачами (Task Management System)
**Цель:** Спроектировать, реализовать и оптимизировать базу данных для управления проектами, задачами, пользователями и напоминаниями, нормализованную до 3NF.
## Лабораторная работа 1. Проектирование структуры БД
<img width="864" height="708" alt="ER-диаграмма" src= "er-diagram.png" />
### Нормальные формы
- **1NF (Первая нормальная форма):**
  Соблюдена. Все атрибуты атомарны, строки уникальны, каждая таблица имеет первичный ключ.
- **2NF (Вторая нормальная форма):**
  Соблюдена. Все неключевые атрибуты полностью зависят от всего первичного ключа.
- **3NF (Третья нормальная форма):**
  Соблюдена. Транзитивные зависимости между неключевыми атрибутами отсутствуют.
## Лабораторная работа 2. Инсталляция БД на сервере
### Создание таблиц
```sql
-- Таблица priority (Приоритеты задач)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".priority
(
    priority_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".priority_priority_id_seq'::regclass),
    name character varying(50) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT priority_pkey PRIMARY KEY (priority_id)
);
-- Таблица projects (Проекты)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".projects
(
    project_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".projects_project_id_seq'::regclass),
    user_id integer NOT NULL,
    project_name character varying(255) COLLATE pg_catalog."default" NOT NULL,
    description character varying(255) COLLATE pg_catalog."default",
    creation_date date DEFAULT CURRENT_DATE,
    CONSTRAINT projects_pkey PRIMARY KEY (project_id),
    CONSTRAINT fk_projects_user FOREIGN KEY (user_id)
        REFERENCES "Lebedinskiy_2272".users (user_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE CASCADE
);
-- Таблица reminders (Напоминания)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".reminders
(
    reminder_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".reminders_reminder_id_seq'::regclass),
    task_id integer NOT NULL,
    reminder_time timestamp without time zone NOT NULL,
    type character varying(50) COLLATE pg_catalog."default",
    status character varying(50) COLLATE pg_catalog."default",
    CONSTRAINT reminders_pkey PRIMARY KEY (reminder_id),
    CONSTRAINT fk_reminders_task FOREIGN KEY (task_id)
        REFERENCES "Lebedinskiy_2272".tasks (task_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE CASCADE
);
-- Таблица status (Статусы задач)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".status
(
    status_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".status_status_id_seq'::regclass),
    name character varying(50) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT status_pkey PRIMARY KEY (status_id)
);
-- Таблица tasks (Задачи)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".tasks
(
    task_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".tasks_task_id_seq'::regclass),
    project_id integer NOT NULL,
    user_id integer NOT NULL,
    title character varying(255) COLLATE pg_catalog."default" NOT NULL,
    description character varying(255) COLLATE pg_catalog."default",
    status_id integer NOT NULL,
    priority_id integer NOT NULL,
    due_date date,
    completed_date date,
    creation_date date DEFAULT CURRENT_DATE,
    CONSTRAINT tasks_pkey PRIMARY KEY (task_id),
    CONSTRAINT fk_tasks_priority FOREIGN KEY (priority_id)
        REFERENCES "Lebedinskiy_2272".priority (priority_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE RESTRICT,
    CONSTRAINT fk_tasks_project FOREIGN KEY (project_id)
        REFERENCES "Lebedinskiy_2272".projects (project_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE CASCADE,
    CONSTRAINT fk_tasks_status FOREIGN KEY (status_id)
        REFERENCES "Lebedinskiy_2272".status (status_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE RESTRICT,
    CONSTRAINT fk_tasks_user FOREIGN KEY (user_id)
        REFERENCES "Lebedinskiy_2272".users (user_id) MATCH SIMPLE
        ON UPDATE NO ACTION ON DELETE CASCADE
);
-- Таблица users (Пользователи)
CREATE TABLE IF NOT EXISTS "Lebedinskiy_2272".users
(
    user_id integer NOT NULL DEFAULT nextval('"Lebedinskiy_2272".users_user_id_seq'::regclass),
    name character varying(255) COLLATE pg_catalog."default" NOT NULL,
    email character varying(255) COLLATE pg_catalog."default" NOT NULL,
    registration_date date DEFAULT CURRENT_DATE,
    password_hash character varying(255) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT users_pkey PRIMARY KEY (user_id),
    CONSTRAINT users_email_key UNIQUE (email)
);
```
### Заполнение данными
```sql
-- Заполняем справочники
INSERT INTO "Lebedinskiy_2272".priority (name) VALUES
('Низкий'), ('Средний'), ('Высокий'), ('Критический');
INSERT INTO "Lebedinskiy_2272".status (name) VALUES
('Новая'), ('В работе'), ('Завершена'), ('Отложена');
-- Заполняем пользователей
INSERT INTO "Lebedinskiy_2272".users (name, email, password_hash) VALUES
('Илья', 'ilya@example.com', 'hash123'),
('Анна', 'anna@example.com', 'hash456'),
('Сергей', 'sergey@example.com', 'hash789'),
('Полина', 'polina@example.com', 'hash1011');
-- Заполняем проекты
INSERT INTO "Lebedinskiy_2272".projects (user_id, project_name, description) VALUES
(1, 'Личный проект', 'Мои повседневные дела'),
(2, 'Рабочий проект', 'Разработка приложения'),
(3, 'Учебный проект', 'Курсовая работа'),
(4, 'Хобби проект', 'Игровой сервер');
-- Заполняем задачи (пример)
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
  u.name AS пользователь,
  p.project_name AS проект,
  t.title AS задача,
  s.name AS статус,
  pr.name AS приоритет,
  t.due_date AS срок
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
    u.name AS пользователь,
    p.project_name AS проект,
    t.title AS задача,
    t.description AS описание,
    s.name AS статус,
    pr.name AS приоритет,
    t.due_date AS срок,
    t.completed_date AS завершено,
    r.reminder_time AS время_напоминания,
    r.type AS тип_напоминания
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
    пользователь text,
    проект text,
    задача text,
    статус text,
    приоритет text,
    срок date
)
LANGUAGE SQL
AS $$
    SELECT
        v.пользователь,
        v.проект,
        v.задача,
        v.статус,
        v.приоритет,
        v.срок
    FROM "Lebedinskiy_2272".vw_full_task_info v
    ORDER BY v.приоритет DESC, v.срок ASC;
$$;
```
**Назначение функции**
Возвращает список задач, отсортированных по убыванию приоритета и сроку выполнения.
**Использование функции**
```sql
SELECT * FROM "Lebedinskiy_2272".get_tasks_sorted_by_priority();
```
## Лабораторная работа 4. Анализ производительности
### Генерация тестовых данных (20 000 записей)
```sql
INSERT INTO "Lebedinskiy_2272".tasks
(project_id, user_id, title, description, status_id, priority_id, due_date, creation_date)
SELECT
  (FLOOR(random()*50)+1)::int,
  (FLOOR(random()*100)+1)::int,
  'Автозадача #' || gs.i,
  'Тестовое описание ' || gs.i,
  (FLOOR(random()*4)+1)::int,
  (FLOOR(random()*4)+1)::int,
  CURRENT_DATE + (FLOOR(random()*180))::int,
  CURRENT_DATE - (FLOOR(random()*365))::int
FROM generate_series(1, 20000) gs(i);
```
### Анализ плана выполнения
```sql
EXPLAIN ANALYZE
SELECT
    pr.name AS приоритет,
    COUNT(*) AS кол_во_задач,
    AVG(t.due_date - t.creation_date) AS средний_срок_дней
FROM "Lebedinskiy_2272".tasks t
JOIN "Lebedinskiy_2272".priority pr ON t.priority_id = pr.priority_id
WHERE t.creation_date >= CURRENT_DATE - INTERVAL '90 days'
GROUP BY pr.name
ORDER BY кол_во_задач DESC;
```
### Создание индексов для оптимизации
```sql
CREATE INDEX idx_tasks_creation_date ON "Lebedinskiy_2272".tasks (creation_date);
CREATE INDEX idx_tasks_priority_id ON "Lebedinskiy_2272".tasks (priority_id);
```
(После создания индексов повторите EXPLAIN ANALYZE и сравните время выполнения + план запроса)
## Лабораторная работа 5. Триггеры и аудит
### Таблица журнала аудита
```sql
CREATE TABLE "Lebedinskiy_2272".audit (
  audit_id SERIAL PRIMARY KEY,
  table_name TEXT NOT NULL,
  operation TEXT NOT NULL CHECK (operation IN ('INSERT','UPDATE','DELETE','DELETE (cascade)')),
  record_id INT,
  old_values JSONB,
  new_values JSONB,
  changed_by TEXT NOT NULL DEFAULT CURRENT_USER,
  changed_at TIMESTAMP NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_audit_table ON "Lebedinskiy_2272".audit (table_name);
CREATE INDEX idx_audit_operation ON "Lebedinskiy_2272".audit (operation);
CREATE INDEX idx_audit_time ON "Lebedinskiy_2272".audit (changed_at);
```
### Функция аудита
```sql
CREATE OR REPLACE FUNCTION "Lebedinskiy_2272".audit_log_universal()
RETURNS TRIGGER AS $$
DECLARE
  pk_column_name TEXT;
  record_json JSONB;
  pk_value INT;
BEGIN
  SELECT a.attname INTO pk_column_name
  FROM pg_index i JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
  WHERE i.indrelid = TG_RELID AND i.indisprimary LIMIT 1;
  IF TG_OP = 'INSERT' THEN
    record_json := to_jsonb(NEW);
    pk_value := (record_json ->> pk_column_name)::INT;
  ELSE
    record_json := to_jsonb(OLD);
    pk_value := (record_json ->> pk_column_name)::INT;
  END IF;
  INSERT INTO "Lebedinskiy_2272".audit (
    table_name, operation, record_id, old_values, new_values, changed_by
  ) VALUES (
    TG_TABLE_NAME,
    TG_OP,
    pk_value,
    CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN record_json END,
    CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) END,
    CURRENT_USER
  );
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```
### Триггеры аудита
```sql
CREATE TRIGGER trg_audit_tasks
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".tasks
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();
CREATE TRIGGER trg_audit_projects
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".projects
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();
CREATE TRIGGER trg_audit_users
AFTER INSERT OR UPDATE OR DELETE ON "Lebedinskiy_2272".users
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".audit_log_universal();
```
### Функция каскадного удаления
```sql
CREATE OR REPLACE FUNCTION "Lebedinskiy_2272".cascade_delete_project()
RETURNS TRIGGER AS $$
DECLARE
  r RECORD;
BEGIN
  FOR r IN SELECT task_id FROM "Lebedinskiy_2272".tasks WHERE project_id = OLD.project_id LOOP
    INSERT INTO "Lebedinskiy_2272".audit (table_name, operation, record_id, changed_by)
    VALUES ('tasks', 'DELETE (cascade)', r.task_id, CURRENT_USER);
    INSERT INTO "Lebedinskiy_2272".audit (table_name, operation, record_id, changed_by)
    SELECT 'reminders', 'DELETE (cascade)', reminder_id, CURRENT_USER
    FROM "Lebedinskiy_2272".reminders WHERE task_id = r.task_id;
    DELETE FROM "Lebedinskiy_2272".reminders WHERE task_id = r.task_id;
    DELETE FROM "Lebedinskiy_2272".tasks WHERE task_id = r.task_id;
  END LOOP;
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```
### Триггер каскадного удаления
```sql
CREATE TRIGGER trg_cascade_delete_project
BEFORE DELETE ON "Lebedinskiy_2272".projects
FOR EACH ROW EXECUTE FUNCTION "Lebedinskiy_2272".cascade_delete_project();
```
### Проверка
```sql
-- Тест
INSERT INTO "Lebedinskiy_2272".projects (user_id, project_name) VALUES (1, 'Тестовый проект');
INSERT INTO "Lebedinskiy_2272".tasks (project_id, user_id, title, status_id, priority_id, due_date)
VALUES (currval('"Lebedinskiy_2272".projects_project_id_seq'), 1, 'Тест задача', 1, 1, CURRENT_DATE + 1);
INSERT INTO "Lebedinskiy_2272".reminders (task_id, reminder_time, type, status)
VALUES (currval('"Lebedinskiy_2272".tasks_task_id_seq'), CURRENT_TIMESTAMP + '1 day', 'Push', 'Активно');
UPDATE "Lebedinskiy_2272".tasks SET title = 'Обновлено' WHERE task_id = currval('"Lebedinskiy_2272".tasks_task_id_seq');
DELETE FROM "Lebedinskiy_2272".projects WHERE project_name = 'Тестовый проект';
SELECT * FROM "Lebedinskiy_2272".audit ORDER BY changed_at DESC LIMIT 10;

```<img width="808" height="675" alt="er-diagram" src="https://github.com/user-attachments/assets/39f2a769-a75e-400a-8247-5dc0e2c1ed4e" />
