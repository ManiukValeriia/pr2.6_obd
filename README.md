# pr2.6_obd
# ПРАКТИЧНА РОБОТА № 6
# Поняття транзакцій й робота з ними в MSSQL

**Короткий опис:**

У роботі розглянуто індекси в Microsoft SQL Server, їх призначення та вплив на продуктивність запитів. Вивчено способи створення, оптимізації, реорганізації 
та видалення індексів. Отримано практичні навички роботи з індексами та аналізу їх ефективності для покращення роботи бази даних.

**Завдання 2**

Транзакція — це логічна послідовність однієї або кількох SQL-операцій, які виконуються як єдине ціле. Вона має властивості ACID:
1. Атомарність (Atomicity): або всі зміни виконуються, або жодна.
2. Узгодженість (Consistency): транзакція переводить БД з одного узгодженого стану в інший.
3. Ізольованість (Isolation): паралельні транзакції не повинні заважати одна одній.
4. Довговічність (Durability): після фіксації результат зберігається навіть у разі збою

**Завдання 3**

BEGIN TRAN;

INSERT INTO Animal (
    Nickname, Gender, Age, Purpose,
    PigletsCount, PigletsBirthDate,
    FatherNickname, MotherNickname
)
VALUES (
    'Bella', 'female', 2, 'breeding',
    4, '2025-03-20',
    'Rocky', 'Lilly'
);

COMMIT;

**Завдання 4**

BEGIN TRAN;

UPDATE Animal SET Purpose = 'slaughter' WHERE Nickname = 'Bella';

IF (SELECT COUNT(*) FROM Animal WHERE Gender = 'nonbinary') = 0
    ROLLBACK;
ELSE
COMMIT;

**Завдання 5**

BEGIN TRAN;

UPDATE Animal SET Age = Age + 1 WHERE Nickname = 'UnknownName';

IF @@ERROR <> 0
    ROLLBACK;
ELSE
    COMMIT;

**Завдання 6**

BEGIN TRAN;

SAVE TRANSACTION point1;

UPDATE Animal SET Age = 99 WHERE Nickname = 'Bella';

ROLLBACK TRANSACTION point1;
COMMIT;
 
**Завдання 7**

BEGIN TRAN;

BEGIN TRY
    UPDATE Animal SET Gender = 'unknown' WHERE Nickname = 'Bella'; -- порушує CHECK
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT 'Помилка під час оновлення: ' + ERROR_MESSAGE();
END CATCH;

**Завдання 8**

CREATE TABLE AuditLog (
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    Action NVARCHAR(255),
    Timestamp DATETIME
);

BEGIN TRAN;

UPDATE Animal SET Age = Age + 1 WHERE Nickname = 'Bella';

INSERT INTO AuditLog (Action, Timestamp)
VALUES ('Updated age for Bella', GETDATE());

COMMIT;

**Завдання 9**

BEGIN TRAN;

BEGIN TRY
    INSERT INTO Animal (
        Nickname, Gender, Age, Purpose,
        PigletsCount, PigletsBirthDate,
        FatherNickname, MotherNickname
    )
    VALUES (
        'Luna', 'female', 2, 'breeding',
        5, '2025-03-01',
        'Max', 'Daisy'
    );

    DECLARE @AnimalID INT = SCOPE_IDENTITY();

    INSERT INTO WeightMeasurement (AnimalID, Date, Weight)
    VALUES (@AnimalID, GETDATE(), 51.2);

    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT 'Атомарність порушена, транзакція скасована: ' + ERROR_MESSAGE();
END CATCH;

**Завдання 10**

BEGIN TRAN;

BEGIN TRY
    INSERT INTO Animal (
        Nickname, Gender, Age, Purpose,
        PigletsCount, PigletsBirthDate,
        FatherNickname, MotherNickname
    )
    VALUES (
        'Oscar', 'male', 1, 'sale',
        NULL, NULL,
        'Bruno', 'Mia'
    );

    UPDATE AnimalStats SET TotalAnimals = TotalAnimals + 1 WHERE StatID = 1;

    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT 'Порушення узгодженості, транзакція скасована: ' + ERROR_MESSAGE();
END CATCH;

**Завдання 11**

BEGIN TRAN;

BEGIN TRY
    -- 1. Додавання тварини
    INSERT INTO Animal (
        Nickname, Gender, Age, Purpose,
        PigletsCount, PigletsBirthDate,
        FatherNickname, MotherNickname
    )
    VALUES (
        'Molly', 'female', 3, 'breeding',
        3, '2025-04-01',
        'Rex', 'Nala'
    );

    DECLARE @NewAnimalID INT = SCOPE_IDENTITY();

    -- 2. Вакцинація
    INSERT INTO Vaccination (AnimalID, VaccineName, VaccinationDate)
    VALUES (@NewAnimalID, 'AntiFlu', GETDATE());

    -- 3. Оновлення мети
    UPDATE Animal SET Purpose = 'sale' WHERE AnimalID = @NewAnimalID;

    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT 'Помилка в транзакції (3 дії): ' + ERROR_MESSAGE();
END CATCH;

**Завдання 12**

Транзакція autocommit — це режим роботи бази даних, у якому кожна окрема SQL-операція автоматично виконується як окрема транзакція. Після її виконання 
зміни одразу фіксуються (тобто відбувається автоматичний COMMIT), і їх уже не можна скасувати без додаткових дій.

UPDATE authors SET au_fname = 'John' WHERE au_id = '172-32-1176';
Якщо режим autocommit увімкнено (а це за замовчуванням у багатьох системах, як MySQL), то цей оператор виконається одразу як транзакція:
- Система автоматично створює транзакцію.
- Виконується команда UPDATE.
- Одразу автоматично виконується COMMIT.
Це означає, що зміни зберігаються в базі назавжди, і скасувати їх уже не можна (наприклад, через ROLLBACK).
Використовується явна транзакція, якщо користувач хоче більше контролювати процес: 
BEGIN TRAN;
UPDATE authors SET au_fname = 'John' WHERE au_id = '172-32-1176';
-- тут ще можна виконати інші дії
COMMIT;

У цьому випадку:
- Зміни не фіксуються, поки ви явно не виконаєте COMMIT.
- Можна відкотити зміни через ROLLBACK, якщо передумали або сталася помилка.
Отже, у режимі autocommit кожна команда, наприклад UPDATE, одразу зберігається в базі, і її вже не можна скасувати. Це зручно, але небезпечно, якщо сталася помилка.
Використовуючи явну транзакцію (через BEGIN TRAN), то зміни зберігаються тільки після  підтвердження користувача (COMMIT). До цього моменту користувач може передумати
і скасувати все (ROLLBACK). Це безпечніше, особливо коли кілька операцій залежать одна від одної.
