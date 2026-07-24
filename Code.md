
-- Таблица "Станции"

CREATE TABLE IF NOT EXISTS Станции (
    id_станции SERIAL PRIMARY KEY,
    название VARCHAR(100) NOT NULL,
    адрес VARCHAR(200),
    дата_ввода DATE,
    статус VARCHAR(20) DEFAULT 'активна' CHECK (статус IN ('активна', 'на реконструкции', 'закрыта'))
);

COMMENT ON TABLE Станции IS 'Передающие станции филиала РТРС';
COMMENT ON COLUMN Станции.id_станции IS 'Уникальный идентификатор станции';
COMMENT ON COLUMN Станции.название IS 'Наименование станции';
COMMENT ON COLUMN Станции.адрес IS 'Юридический адрес станции';
COMMENT ON COLUMN Станции.дата_ввода IS 'Дата ввода станции в эксплуатацию';
COMMENT ON COLUMN Станции.статус IS 'Текущий статус станции';


-- Таблица "Оборудование"

CREATE TABLE IF NOT EXISTS Оборудование (
    id_оборудования SERIAL PRIMARY KEY,
    id_станции INTEGER NOT NULL REFERENCES Станции(id_станции) ON DELETE RESTRICT,
    название VARCHAR(100) NOT NULL,
    модель VARCHAR(50),
    серийный_номер VARCHAR(50) UNIQUE,
    дата_установки DATE,
    гарантия_до DATE,
    статус VARCHAR(20) DEFAULT 'в работе' CHECK (статус IN ('в работе', 'на ремонте', 'списано', 'резерв'))
);

CREATE INDEX IF NOT EXISTS idx_оборудование_станции ON Оборудование(id_станции);
CREATE INDEX IF NOT EXISTS idx_оборудование_статус ON Оборудование(статус);

COMMENT ON TABLE Оборудование IS 'Единицы оборудования на станциях';
COMMENT ON COLUMN Оборудование.id_оборудования IS 'Уникальный идентификатор оборудования';
COMMENT ON COLUMN Оборудование.id_станции IS 'Ссылка на станцию (внешний ключ)';
COMMENT ON COLUMN Оборудование.название IS 'Наименование оборудования';
COMMENT ON COLUMN Оборудование.модель IS 'Модель оборудования';
COMMENT ON COLUMN Оборудование.серийный_номер IS 'Серийный номер (уникальный)';
COMMENT ON COLUMN Оборудование.дата_установки IS 'Дата установки оборудования';
COMMENT ON COLUMN Оборудование.гарантия_до IS 'Дата окончания гарантии';
COMMENT ON COLUMN Оборудование.статус IS 'Текущий статус оборудования';


-- Таблица "Типы_работ"

CREATE TABLE IF NOT EXISTS Типы_работ (
    id_типа_работ SERIAL PRIMARY KEY,
    название_типа VARCHAR(50) NOT NULL UNIQUE,
    описание TEXT,
    периодичность_дней INTEGER DEFAULT 365,
    трудоемкость_час NUMERIC(5,2) DEFAULT 1.0
);

COMMENT ON TABLE Типы_работ IS 'Справочник типов ремонтных работ';
COMMENT ON COLUMN Типы_работ.id_типа_работ IS 'Уникальный идентификатор типа работ';
COMMENT ON COLUMN Типы_работ.название_типа IS 'Название типа работ (уникальное)';
COMMENT ON COLUMN Типы_работ.описание IS 'Описание работ';
COMMENT ON COLUMN Типы_работ.периодичность_дней IS 'Рекомендуемая периодичность в днях';
COMMENT ON COLUMN Типы_работ.трудоемкость_час IS 'Плановая трудоемкость в часах';


-- Таблица "Инженеры"

CREATE TABLE IF NOT EXISTS Инженеры (
    id_инженера SERIAL PRIMARY KEY,
    фио VARCHAR(100) NOT NULL,
    должность VARCHAR(50),
    телефон VARCHAR(20),
    email VARCHAR(50) UNIQUE NOT NULL,
    пароль_hash VARCHAR(255),
    дата_приема DATE,
    активен BOOLEAN DEFAULT TRUE
);

CREATE INDEX IF NOT EXISTS idx_инженеры_email ON Инженеры(email);

COMMENT ON TABLE Инженеры IS 'Сотрудники, ответственные за обслуживание оборудования';
COMMENT ON COLUMN Инженеры.id_инженера IS 'Уникальный идентификатор инженера';
COMMENT ON COLUMN Инженеры.фио IS 'ФИО сотрудника';
COMMENT ON COLUMN Инженеры.должность IS 'Должность';
COMMENT ON COLUMN Инженеры.телефон IS 'Контактный телефон';
COMMENT ON COLUMN Инженеры.email IS 'Email (уникальный, используется как логин)';
COMMENT ON COLUMN Инженеры.пароль_hash IS 'Хеш пароля для аутентификации';
COMMENT ON COLUMN Инженеры.дата_приема IS 'Дата приема на работу';
COMMENT ON COLUMN Инженеры.активен IS 'Признак активности сотрудника';


-- Таблица "График_ППР"

CREATE TABLE IF NOT EXISTS График_ППР (
    id_записи SERIAL PRIMARY KEY,
    id_оборудования INTEGER NOT NULL REFERENCES Оборудование(id_оборудования) ON DELETE CASCADE,
    id_типа_работ INTEGER NOT NULL REFERENCES Типы_работ(id_типа_работ),
    id_ответственного INTEGER NOT NULL REFERENCES Инженеры(id_инженера),
    плановая_дата DATE NOT NULL,
    статус VARCHAR(20) DEFAULT 'запланировано' CHECK (статус IN ('запланировано', 'в работе', 'выполнено', 'отменено', 'просрочено')),
    приоритет INTEGER DEFAULT 1 CHECK (приоритет BETWEEN 1 AND 5),
    создано TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    обновлено TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_ппр_оборудование ON График_ППР(id_оборудования);
CREATE INDEX IF NOT EXISTS idx_ппр_ответственный ON График_ППР(id_ответственного);
CREATE INDEX IF NOT EXISTS idx_ппр_плановая_дата ON График_ППР(плановая_дата);
CREATE INDEX IF NOT EXISTS idx_ппр_статус ON График_ППР(статус);

COMMENT ON TABLE График_ППР IS 'План-график планово-предупредительных ремонтов';
COMMENT ON COLUMN График_ППР.id_записи IS 'Уникальный идентификатор записи';
COMMENT ON COLUMN График_ППР.id_оборудования IS 'Ссылка на оборудование (внешний ключ)';
COMMENT ON COLUMN График_ППР.id_типа_работ IS 'Ссылка на тип работ (внешний ключ)';
COMMENT ON COLUMN График_ППР.id_ответственного IS 'Ссылка на ответственного инженера (внешний ключ)';
COMMENT ON COLUMN График_ППР.плановая_дата IS 'Плановая дата проведения работ';
COMMENT ON COLUMN График_ППР.статус IS 'Статус записи';
COMMENT ON COLUMN График_ППР.приоритет IS 'Приоритет работ (1-низкий, 5-высокий)';
COMMENT ON COLUMN График_ППР.создано IS 'Дата и время создания записи';
COMMENT ON COLUMN График_ППР.обновлено IS 'Дата и время последнего обновления';


-- Таблица "Выполненные_работы"

CREATE TABLE IF NOT EXISTS Выполненные_работы (
    id_выполнения SERIAL PRIMARY KEY,
    id_записи_ППР INTEGER NOT NULL REFERENCES График_ППР(id_записи) ON DELETE CASCADE,
    фактическая_дата DATE NOT NULL,
    описание_работ TEXT,
    затраты_время NUMERIC(5,2),
    использованные_мат_лы TEXT,
    примечания TEXT,
    создано TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_выполн_запись_ппр ON Выполненные_работы(id_записи_ППР);

COMMENT ON TABLE Выполненные_работы IS 'Отчеты о фактически выполненных работах';
COMMENT ON COLUMN Выполненные_работы.id_выполнения IS 'Уникальный идентификатор записи';
COMMENT ON COLUMN Выполненные_работы.id_записи_ППР IS 'Ссылка на запись в графике ППР (внешний ключ)';
COMMENT ON COLUMN Выполненные_работы.фактическая_дата IS 'Фактическая дата выполнения';
COMMENT ON COLUMN Выполненные_работы.описание_работ IS 'Описание выполненных работ';
COMMENT ON COLUMN Выполненные_работы.затраты_время IS 'Фактические затраты времени в часах';
COMMENT ON COLUMN Выполненные_работы.использованные_мат_лы IS 'Использованные материалы и запчасти';
COMMENT ON COLUMN Выполненные_работы.примечания IS 'Дополнительные примечания';


-- Таблица "Техническая_документация"

CREATE TABLE IF NOT EXISTS Техническая_документация (
    id_документа SERIAL PRIMARY KEY,
    id_оборудования INTEGER REFERENCES Оборудование(id_оборудования) ON DELETE CASCADE,
    название_файла VARCHAR(255) NOT NULL,
    тип_документа VARCHAR(50),
    файл_путь VARCHAR(500),
    дата_загрузки DATE DEFAULT CURRENT_DATE,
    загрузил INTEGER REFERENCES Инженеры(id_инженера),
    описание TEXT,
    версия VARCHAR(10) DEFAULT '1.0'
);

CREATE INDEX IF NOT EXISTS idx_документ_оборудование ON Техническая_документация(id_оборудования);

COMMENT ON TABLE Техническая_документация IS 'Техническая документация на оборудование';
COMMENT ON COLUMN Техническая_документация.id_документа IS 'Уникальный идентификатор документа';
COMMENT ON COLUMN Техническая_документация.id_оборудования IS 'Ссылка на оборудование (внешний ключ)';
COMMENT ON COLUMN Техническая_документация.название_файла IS 'Имя файла';
COMMENT ON COLUMN Техническая_документация.тип_документа IS 'Тип документа';
COMMENT ON COLUMN Техническая_документация.файл_путь IS 'Путь к файлу на сервере';
COMMENT ON COLUMN Техническая_документация.дата_загрузки IS 'Дата загрузки документа';
COMMENT ON COLUMN Техническая_документация.загрузил IS 'Инженер, загрузивший документ (внешний ключ)';
COMMENT ON COLUMN Техническая_документация.описание IS 'Описание документа';
COMMENT ON COLUMN Техническая_документация.версия IS 'Версия документа';


-- Таблица "Журнал_аудита"

CREATE TABLE IF NOT EXISTS Журнал_аудита (
    id_записи SERIAL PRIMARY KEY,
    пользователь_идентификатор INTEGER REFERENCES Инженеры(id_инженера),
    действие VARCHAR(50) NOT NULL,
    таблица VARCHAR(50),
    запись_ид INTEGER,
    старые_данные JSONB,
    новые_данные JSONB,
    ip_адрес INET,
    время TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_аудит_пользователь ON Журнал_аудита(пользователь_идентификатор);
CREATE INDEX IF NOT EXISTS idx_аудит_время ON Журнал_аудита(время);

COMMENT ON TABLE Журнал_аудита IS 'Журнал аудита действий пользователей';
COMMENT ON COLUMN Журнал_аудита.id_записи IS 'Уникальный идентификатор записи';
COMMENT ON COLUMN Журнал_аудита.пользователь_идентификатор IS 'Идентификатор пользователя (внешний ключ)';
COMMENT ON COLUMN Журнал_аудита.действие IS 'Тип действия';
COMMENT ON COLUMN Журнал_аудита.таблица IS 'Имя таблицы';
COMMENT ON COLUMN Журнал_аудита.запись_ид IS 'Идентификатор измененной записи';
COMMENT ON COLUMN Журнал_аудита.старые_данные IS 'Старые данные в формате JSON';
COMMENT ON COLUMN Журнал_аудита.новые_данные IS 'Новые данные в формате JSON';
COMMENT ON COLUMN Журнал_аудита.ip_адрес IS 'IP-адрес пользователя';
COMMENT ON COLUMN Журнал_аудита.время IS 'Время выполнения действия';


-- Заполнение справочников с тестовыми данными


-- Станции

INSERT INTO Станции (название, адрес, дата_ввода, статус) VALUES
('РТС Саранск', 'г. Саранск, ул. Гончарова, д. 39', '1960-01-01', 'активна'),
('РТС Рузаевка', 'г. Рузаевка, ул. Промышленная, д. 12', '1975-06-15', 'активна'),
('РТС Ковылкино', 'г. Ковылкино, ул. Советская, д. 45', '1980-03-20', 'активна'),
('РТС Ардатов', 'г. Ардатов, ул. Ленина, д. 78', '1985-11-01', 'активна'),
('РТС Краснослободск', 'г. Краснослободск, ул. Мордовская, д. 23', '1990-07-15', 'на реконструкции')
ON CONFLICT DO NOTHING;


-- Оборудование

INSERT INTO Оборудование (id_станции, название, модель, серийный_номер, дата_установки, гарантия_до, статус) VALUES
(1, 'Передатчик DVB-T2 1 кВт', 'T2-SM-1000', 'SN-2024-001', '2024-03-15', '2026-03-15', 'в работе'),
(1, 'Передатчик DVB-T2 2 кВт', 'T2-SM-2000', 'SN-2024-002', '2024-05-20', '2026-05-20', 'в работе'),
(1, 'Антенна UHF панельная', 'UHF-8-bay', 'ANT-2023-005', '2023-11-01', '2025-11-01', 'в работе'),
(2, 'Передатчик FM 500 Вт', 'FM-500', 'SN-2023-012', '2023-05-20', '2025-05-20', 'в работе'),
(2, 'Передатчик FM 1 кВт', 'FM-1000', 'SN-2023-013', '2023-08-10', '2025-08-10', 'в работе'),
(2, 'Антенна VHF', 'VHF-4-bay', 'ANT-2022-008', '2022-12-01', '2024-12-01', 'на ремонте'),
(3, 'Передатчик DVB-T2 500 Вт', 'T2-SM-500', 'SN-2024-003', '2024-01-15', '2026-01-15', 'в работе'),
(3, 'Система охлаждения', 'COOL-2000', 'SN-2023-020', '2023-09-01', '2025-09-01', 'в работе'),
(4, 'Передатчик FM 200 Вт', 'FM-200', 'SN-2022-025', '2022-06-01', '2024-06-01', 'в работе'),
(5, 'Передатчик DVB-T2 1 кВт', 'T2-SM-1000', 'SN-2021-030', '2021-11-15', '2023-11-15', 'резерв')
ON CONFLICT (серийный_номер) DO NOTHING;


-- Типы работ

INSERT INTO Типы_работ (название_типа, описание, периодичность_дней, трудоемкость_час) VALUES
('Профилактический осмотр', 'Визуальный осмотр, проверка параметров, очистка', 30, 2.0),
('Плановый ремонт', 'Ремонт с заменой изношенных деталей', 365, 8.0),
('Замена комплектующих', 'Замена вышедших из строя компонентов', 180, 4.0),
('Капитальный ремонт', 'Полный ремонт с восстановлением ресурса', 1095, 24.0),
('Настройка и регулировка', 'Настройка параметров работы оборудования', 90, 3.0),
('Диагностика', 'Полная диагностика с использованием измерительного оборудования', 60, 2.0)
ON CONFLICT (название_типа) DO NOTHING;


-- Инженеры

INSERT INTO Инженеры (фио, должность, телефон, email, пароль_hash, дата_приема, активен) VALUES
('Иванов Иван Иванович', 'Главный инженер', '+7-927-111-2233', 'ivanov@rtrs.ru', 'hash_ivanov', '2010-03-15', TRUE),
('Петров Петр Петрович', 'Инженер ПТО', '+7-927-222-3344', 'petrov@rtrs.ru', 'hash_petrov', '2015-07-01', TRUE),
('Сидоров Сергей Сергеевич', 'Инженер по эксплуатации', '+7-927-333-4455', 'sidorov@rtrs.ru', 'hash_sidorov', '2018-02-20', TRUE),
('Козлова Анна Михайловна', 'Инженер по документации', '+7-927-444-5566', 'kozlov@rtrs.ru', 'hash_kozlov', '2020-09-10', TRUE),
('Морозов Дмитрий Алексеевич', 'Инженер-электронщик', '+7-927-555-6677', 'morozov@rtrs.ru', 'hash_morozov', '2016-11-05', TRUE),
('Волков Алексей Сергеевич', 'Младший инженер', '+7-927-666-7788', 'volkov@rtrs.ru', 'hash_volkov', '2022-06-01', TRUE),
('Громова Елена Викторовна', 'Инженер по документообороту', '+7-927-777-8899', 'gromova@rtrs.ru', 'hash_gromova', '2019-04-15', TRUE),
('Белов Антон Владимирович', 'Стажер', '+7-927-888-9900', 'belov@rtrs.ru', 'hash_belov', '2026-01-10', TRUE)
ON CONFLICT (email) DO NOTHING;


-- График ППР

INSERT INTO График_ППР (id_оборудования, id_типа_работ, id_ответственного, плановая_дата, статус, приоритет) VALUES
(1, 1, 2, '2026-08-15', 'запланировано', 3),
(1, 2, 1, '2026-12-10', 'запланировано', 4),
(2, 1, 3, '2026-09-01', 'запланировано', 3),
(3, 5, 3, '2026-07-20', 'запланировано', 2),
(4, 1, 2, '2026-08-05', 'запланировано', 3),
(4, 3, 5, '2026-11-15', 'запланировано', 4),
(5, 1, 2, '2026-07-10', 'выполнено', 3),
(5, 2, 1, '2026-10-20', 'запланировано', 4),
(6, 1, 3, '2026-06-01', 'просрочено', 5),
(7, 1, 2, '2026-08-25', 'запланировано', 2),
(8, 4, 4, '2026-09-15', 'запланировано', 5),
(9, 1, 3, '2026-07-01', 'отменено', 1),
(10, 1, 2, '2026-08-10', 'запланировано', 3)
ON CONFLICT DO NOTHING;


-- Выполненные работы

INSERT INTO Выполненные_работы (id_записи_ППР, фактическая_дата, описание_работ, затраты_время, использованные_мат_лы, примечания) VALUES
(7, '2026-07-10', 'Профилактический осмотр передатчика FM 1 кВт. Проверка параметров, чистка контактов.', 2.5, 'Спирт, салфетки безворсовые', 'Все параметры в норме'),
(9, '2026-07-05', 'Профилактический осмотр передатчика FM 200 Вт. Замена предохранителя.', 1.0, 'Предохранитель 10А', 'Работа выполнена в срок')
ON CONFLICT DO NOTHING;


-- Техническая документация

INSERT INTO Техническая_документация (id_оборудования, название_файла, тип_документа, файл_путь, загрузил, описание, версия) VALUES
(1, 'T2-SM-1000_Паспорт.pdf', 'паспорт', '/docs/equipment/T2-SM-1000_pasp.pdf', 2, 'Технический паспорт передатчика', '2.0'),
(1, 'T2-SM-1000_Схема.pdf', 'схема', '/docs/equipment/T2-SM-1000_sch.pdf', 2, 'Принципиальная электрическая схема', '1.1'),
(4, 'FM-500_Инструкция.pdf', 'инструкция', '/docs/equipment/FM-500_inst.pdf', 3, 'Руководство по эксплуатации', '1.0')
ON CONFLICT DO NOTHING;


-- Создание представлений

CREATE OR REPLACE VIEW V_Инженер_ППР AS
SELECT 
    Инженеры.фио AS инженер,
    Оборудование.название AS оборудование,
    Станции.название AS станция,
    Типы_работ.название_типа AS тип_работ,
    График_ППР.плановая_дата,
    График_ППР.статус,
    График_ППР.приоритет
FROM График_ППР
JOIN Оборудование ON График_ППР.id_оборудования = Оборудование.id_оборудования
JOIN Станции ON Оборудование.id_станции = Станции.id_станции
JOIN Типы_работ ON График_ППР.id_типа_работ = Типы_работ.id_типа_работ
JOIN Инженеры ON График_ППР.id_ответственного = Инженеры.id_инженера;

COMMENT ON VIEW V_Инженер_ППР IS 'Представление для инженера: его график работ';

CREATE OR REPLACE VIEW V_Отчет_Руководителю AS
SELECT 
    Станции.название AS станция,
    COUNT(Оборудование.id_оборудования) AS всего_оборудования,
    COUNT(CASE WHEN Оборудование.статус = 'на ремонте' THEN 1 END) AS на_ремонте,
    COUNT(CASE WHEN Оборудование.статус = 'резерв' THEN 1 END) AS в_резерве,
    COUNT(DISTINCT График_ППР.id_оборудования) AS оборудование_с_ППР,
    COUNT(CASE WHEN График_ППР.плановая_дата < CURRENT_DATE AND График_ППР.статус = 'запланировано' THEN 1 END) AS просроченных_ППР,
    COUNT(CASE WHEN График_ППР.статус = 'выполнено' THEN 1 END) AS выполненных_ППР
FROM Станции
LEFT JOIN Оборудование ON Станции.id_станции = Оборудование.id_станции
LEFT JOIN График_ППР ON Оборудование.id_оборудования = График_ППР.id_оборудования
WHERE Станции.статус = 'активна'
GROUP BY Станции.id_станции;

COMMENT ON VIEW V_Отчет_Руководителю IS 'Сводный отчет по оборудованию для руководителя';

CREATE OR REPLACE VIEW V_Ближайшие_ППР AS
SELECT 
    Станции.название AS станция,
    Оборудование.название AS оборудование,
    Оборудование.модель,
    Типы_работ.название_типа AS тип_работ,
    График_ППР.плановая_дата,
    Инженеры.фио AS ответственный,
    График_ППР.приоритет,
    CASE 
        WHEN График_ППР.плановая_дата < CURRENT_DATE THEN 'ПРОСРОЧЕНО'
        WHEN График_ППР.плановая_дата <= CURRENT_DATE + INTERVAL '7 days' THEN 'Скоро (7 дней)'
        WHEN График_ППР.плановая_дата <= CURRENT_DATE + INTERVAL '30 days' THEN 'В этом месяце'
        ELSE 'Позже'
    END AS срок
FROM График_ППР
JOIN Оборудование ON График_ППР.id_оборудования = Оборудование.id_оборудования
JOIN Станции ON Оборудование.id_станции = Станции.id_станции
JOIN Типы_работ ON График_ППР.id_типа_работ = Типы_работ.id_типа_работ
JOIN Инженеры ON График_ППР.id_ответственного = Инженеры.id_инженера
WHERE График_ППР.статус IN ('запланировано', 'в работе')
ORDER BY График_ППР.плановая_дата ASC
LIMIT 50;

COMMENT ON VIEW V_Ближайшие_ППР IS 'Представление для оператора: ближайшие ППР';

CREATE OR REPLACE VIEW V_История_Ремонтов AS
SELECT 
    Оборудование.название AS оборудование,
    Оборудование.серийный_номер,
    Станции.название AS станция,
    Типы_работ.название_типа AS тип_работ,
    График_ППР.плановая_дата AS плановая_дата,
    Выполненные_работы.фактическая_дата,
    Выполненные_работы.описание_работ,
    Выполненные_работы.затраты_время,
    Инженеры.фио AS выполнил
FROM Выполненные_работы
JOIN График_ППР ON Выполненные_работы.id_записи_ППР = График_ППР.id_записи
JOIN Оборудование ON График_ППР.id_оборудования = Оборудование.id_оборудования
JOIN Станции ON Оборудование.id_станции = Станции.id_станции
JOIN Типы_работ ON График_ППР.id_типа_работ = Типы_работ.id_типа_работ
JOIN Инженеры ON Выполненные_работы.id_выполнения = Выполненные_работы.id_записи_ППР
ORDER BY Выполненные_работы.фактическая_дата DESC;

COMMENT ON VIEW V_История_Ремонтов IS 'История выполненных ремонтов по оборудованию';


-- Триггеры

CREATE OR REPLACE FUNCTION update_updated_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.обновлено = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_ппр_обновлено ON График_ППР;
CREATE TRIGGER trg_ппр_обновлено
    BEFORE UPDATE ON График_ППР
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_column();

CREATE OR REPLACE FUNCTION update_ppr_status()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.статус = 'запланировано' AND NEW.плановая_дата < CURRENT_DATE THEN
        NEW.статус = 'просрочено';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_ппр_статус ON График_ППР;
CREATE TRIGGER trg_ппр_статус
    BEFORE INSERT OR UPDATE OF плановая_дата, статус ON График_ППР
    FOR EACH ROW
    EXECUTE FUNCTION update_ppr_status();

CREATE OR REPLACE FUNCTION update_equipment_status_on_repair()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.статус = 'выполнено' AND OLD.статус != 'выполнено' THEN
        UPDATE Оборудование 
        SET статус = 'в работе' 
        WHERE id_оборудования = NEW.id_оборудования 
          AND статус = 'на ремонте';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trg_оборудование_статус_после_ппр ON График_ППР;
CREATE TRIGGER trg_оборудование_статус_после_ппр
    AFTER UPDATE OF статус ON График_ППР
    FOR EACH ROW
    WHEN (NEW.статус = 'выполнено' AND OLD.статус != 'выполнено')
    EXECUTE FUNCTION update_equipment_status_on_repair();


-- Ролевая модель

DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'admin_role') THEN
        CREATE ROLE admin_role;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'chief_role') THEN
        CREATE ROLE chief_role;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'engineer_role') THEN
        CREATE ROLE engineer_role;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'operator_role') THEN
        CREATE ROLE operator_role;
    END IF;
    IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'guest_role') THEN
        CREATE ROLE guest_role;
    END IF;
END $$;

COMMENT ON ROLE admin_role IS 'Администратор системы - полный доступ';
COMMENT ON ROLE chief_role IS 'Руководитель - просмотр и утверждение';
COMMENT ON ROLE engineer_role IS 'Инженер-эксплуатационник - работа со своим оборудованием';
COMMENT ON ROLE operator_role IS 'Оператор - просмотр и создание заявок';
COMMENT ON ROLE guest_role IS 'Гость - только чтение отчетов';


-- Назначение привилегий

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin_role;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO admin_role;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO admin_role;
GRANT CREATE ON SCHEMA public TO admin_role;

GRANT SELECT, UPDATE ON Станции, Оборудование, Типы_работ, Инженеры, График_ППР, Выполненные_работы, Техническая_документация TO chief_role;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO chief_role;

GRANT SELECT ON Станции, Оборудование, Типы_работ, Инженеры, Техническая_документация TO engineer_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON График_ППР, Выполненные_работы TO engineer_role;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO engineer_role;

GRANT SELECT ON Станции, Оборудование, График_ППР, Выполненные_работы, V_Ближайшие_ППР TO operator_role;

GRANT SELECT ON V_Отчет_Руководителю, V_История_Ремонтов TO guest_role;


-- ROW-LEVEL SECURITY

ALTER TABLE График_ППР ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS engineer_ppr_select_policy ON График_ППР;
CREATE POLICY engineer_ppr_select_policy ON График_ППР
    FOR SELECT
    TO engineer_role
    USING (
        id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user)
        OR current_setting('app.current_role', TRUE) IN ('admin_role', 'chief_role')
    );

DROP POLICY IF EXISTS engineer_ppr_modify_policy ON График_ППР;
CREATE POLICY engineer_ppr_modify_policy ON График_ППР
    FOR ALL
    TO engineer_role
    USING (id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user))
    WITH CHECK (id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user));

DROP POLICY IF EXISTS chief_ppr_policy ON График_ППР;
CREATE POLICY chief_ppr_policy ON График_ППР
    FOR ALL
    TO chief_role
    USING (TRUE)
    WITH CHECK (TRUE);

ALTER TABLE Выполненные_работы ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS engineer_work_select_policy ON Выполненные_работы;
CREATE POLICY engineer_work_select_policy ON Выполненные_работы
    FOR SELECT
    TO engineer_role
    USING (
        id_записи_ППР IN (
            SELECT id_записи FROM График_ППР 
            WHERE id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user)
        )
    );

DROP POLICY IF EXISTS engineer_work_modify_policy ON Выполненные_работы;
CREATE POLICY engineer_work_modify_policy ON Выполненные_работы
    FOR ALL
    TO engineer_role
    USING (
        id_записи_ППР IN (
            SELECT id_записи FROM График_ППР 
            WHERE id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user)
        )
    )
    WITH CHECK (
        id_записи_ППР IN (
            SELECT id_записи FROM График_ППР 
            WHERE id_ответственного = (SELECT id_инженера FROM Инженеры WHERE email = current_user)
        )
    );


-- Функции

CREATE OR REPLACE FUNCTION get_current_engineer_info()
RETURNS TABLE(
    id_инженера INTEGER,
    фио VARCHAR,
    email VARCHAR,
    должность VARCHAR,
    роль VARCHAR
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        Инженеры.id_инженера,
        Инженеры.фио,
        Инженеры.email,
        Инженеры.должность,
        (SELECT rolname FROM pg_authid WHERE oid = (SELECT roloid FROM pg_auth_members WHERE member = (SELECT oid FROM pg_authid WHERE rolname = current_user)) LIMIT 1) AS роль
    FROM Инженеры
    WHERE Инженеры.email = current_user;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE FUNCTION создать_заявку_ремонт(
    p_id_оборудования INTEGER,
    p_описание TEXT,
    p_приоритет INTEGER DEFAULT 3
)
RETURNS INTEGER AS $$
DECLARE
    v_id_записи INTEGER;
    v_инженер_ид INTEGER;
BEGIN
    IF NOT EXISTS (SELECT 1 FROM Оборудование WHERE id_оборудования = p_id_оборудования) THEN
        RAISE EXCEPTION 'Оборудование с ID % не найдено', p_id_оборудования;
    END IF;
    
    SELECT id_инженера INTO v_инженер_ид FROM Инженеры WHERE должность = 'Главный инженер' LIMIT 1;
    
    INSERT INTO График_ППР (id_оборудования, id_типа_работ, id_ответственного, плановая_дата, статус, приоритет)
    VALUES (
        p_id_оборудования,
        (SELECT id_типа_работ FROM Типы_работ WHERE название_типа = 'Диагностика' LIMIT 1),
        v_инженер_ид,
        CURRENT_DATE + INTERVAL '3 days',
        'запланировано',
        p_приоритет
    )
    RETURNING id_записи INTO v_id_записи;
    
    INSERT INTO Журнал_аудита (пользователь_идентификатор, действие, таблица, запись_ид, новые_данные)
    VALUES (
        (SELECT id_инженера FROM Инженеры WHERE email = current_user),
        'INSERT',
        'График_ППР',
        v_id_записи,
        jsonb_build_object('описание', p_описание, 'приоритет', p_приоритет)
    );
    
    RETURN v_id_записи;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

COMMENT ON FUNCTION создать_заявку_ремонт IS 'Функция для создания заявки на внеплановый ремонт';


-- Индексы

CREATE INDEX IF NOT EXISTS idx_ппр_дата_статус ON График_ППР(плановая_дата, статус);
CREATE INDEX IF NOT EXISTS idx_оборудование_серийный ON Оборудование(серийный_номер);
CREATE INDEX IF NOT EXISTS idx_документ_тип ON Техническая_документация(тип_документа);


-- Проверочные запросы

SELECT 'Станции' AS таблица, COUNT(*) AS записей FROM Станции
UNION ALL
SELECT 'Оборудование', COUNT(*) FROM Оборудование
UNION ALL
SELECT 'Типы_работ', COUNT(*) FROM Типы_работ
UNION ALL
SELECT 'Инженеры', COUNT(*) FROM Инженеры
UNION ALL
SELECT 'График_ППР', COUNT(*) FROM График_ППР
UNION ALL
SELECT 'Выполненные_работы', COUNT(*) FROM Выполненные_работы
UNION ALL
SELECT 'Техническая_документация', COUNT(*) FROM Техническая_документация
ORDER BY таблица;

SELECT * FROM V_Отчет_Руководителю;
SELECT * FROM V_Ближайшие_ППР;
