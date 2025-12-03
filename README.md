# SQL

### [Тренажёр](https://sql-academy.org/ru/trainer)

1. Вывести имена всех людей, которые есть в базе данных авиакомпаний.

<details>
  <summary>ERD</summary>
  <br>

  <img width="640" height="451" alt="image" src="https://github.com/user-attachments/assets/90e3e238-8583-4394-af13-a11df491122b" />
</details>

```
SELECT name
FROM Passenger
```

2. Вывести названия всеx авиакомпаний.

```
SELECT name
FROM Company
```

3. Вывести все рейсы, совершенные из Москвы.

```
SELECT *
FROM Trip
WHERE town_from LIKE 'Moscow'
```

4. Вывести имена людей, которые заканчиваются на "man".

```
SELECT name
FROM Passenger
WHERE name LIKE '%man'
```

5. Вывести количество рейсов, совершенных на TU-134.

```
SELECT COUNT(*) AS count
FROM Trip
WHERE plane LIKE 'TU-134'
```

6. Какие компании совершали перелеты на Boeing.

```
SELECT DISTINCT name
FROM Company
JOIN Trip ON Company.id=Trip.company
WHERE plane LIKE 'Boeing%'
```

> DISTINCT используется для получения уникальных строк в наборе результатов.

7. Вывести все названия самолётов, на которых можно улететь в Москву (Moscow).

```
SELECT DISTINCT plane
FROM Trip
WHERE town_to LIKE 'Moscow'
```

8. В какие города можно улететь из Парижа (Paris) и сколько времени это займёт? Формат для вывода времени: HH:MM:SS

```
SELECT town_to,
	TIMEDIFF(time_in, time_out) AS flight_time
FROM Trip
WHERE town_from LIKE 'Paris'
```

9. Какие компании организуют перелеты из Владивостока (Vladivostok)?

```
SELECT name
FROM Company
JOIN Trip ON Company.id=Trip.company
WHERE town_from LIKE 'Vladivostok'
```

10. Вывести вылеты, совершенные с 10 ч. по 14 ч. 1 января 1900 г.

```
SELECT *
FROM Trip
WHERE time_out BETWEEN '1900-01-01 10:00:00' AND '1900-01-01 14:00:00'
```

11. Выведите пассажиров с самым длинным ФИО. Пробелы, дефисы и точки считаются частью имени.

```
SELECT name
FROM Passenger
WHERE LENGTH(name) = (
		SELECT LENGTH(MAX(name))
		FROM Passenger
	)
```

> WHERE LENGTH(name) = <результат подзапроса>: Фильтрует строки, оставляя только те, у которых длина имени (LENGTH(name)) равна числу, полученному во внутреннем запросе.

12. Выведите идентификаторы всех рейсов и количество пассажиров на них. Обратите внимание, что на каких-то рейсах пассажиров может не быть. В этом случае выведите число "0".

```
SELECT Trip.id AS id,
	COALESCE(COUNT(Pass_in_trip.passenger), 0) AS COUNT
FROM Trip
	LEFT JOIN Pass_in_trip ON Trip.id = Pass_in_trip.Trip
GROUP BY Trip.id
```

> GROUP BY используется в SQL для объединения строк, имеющих одинаковые значения в одном или нескольких столбцах, в одну сводную строку.

> LEFT JOIN используется для объединение с таблицей Pass_in_trip (столбец, в котором могут отсутствовать значения).

> COALESCE функция, которая возвращает первое не-NULL значение из списка переданных ей аргументов.
> Если все аргументы равны NULL, функция возвращает NULL. Ее часто используют для замены пропущенных значений на значения по умолчанию или для выбора наиболее релевантного значения из нескольких столбцов.  

13. Вывести имена людей, у которых есть полный тёзка среди пассажиров.

```
SELECT name
FROM Passenger
GROUP BY name
HAVING COUNT(*)>1
```

> Cначала группирует записи по имени, затем удаляет все дубликаты (оставляя по одной записи для каждого уникального имени), и только после этого фильтрует полученные группы, оставляя только те, где количество записей (совпадающих имен) больше 1.

14. В какие города летал Bruce Willis.

```
SELECT town_to
FROM Trip
	JOIN Pass_in_trip ON Trip.id = Pass_in_trip.Trip
	JOIN Passenger ON Pass_in_trip.passenger = Passenger.id
WHERE name LIKE 'Bruce Willis'
```

15. Выведите идентификатор пассажира Стив Мартин (Steve Martin) и дату и время его прилёта в Лондон (London).

```
SELECT Passenger.id,
	Trip.time_in
FROM Passenger
	JOIN Pass_in_trip ON Passenger.id = Pass_in_trip.passenger
	JOIN Trip ON Pass_in_trip.trip = Trip.id
WHERE name LIKE 'Steve Martin'
	AND town_to LIKE 'London'
```

16. Вывести отсортированный по количеству перелетов (по убыванию) и имени (по возрастанию) список пассажиров, совершивших хотя бы 1 полет.

```
SELECT name,
	COUNT(Pass_in_trip.id) AS COUNT
FROM Passenger
	JOIN Pass_in_trip ON Passenger.id = Pass_in_trip.passenger
GROUP BY name
HAVING COUNT > 0
ORDER BY COUNT DESC,
	name ASC
```

17. Определить, сколько потратил в 2005 году каждый из членов семьи. В результирующей выборке не выводите тех членов семьи, которые ничего не потратили.

<details>
  <summary>ERD</summary>
  <br>

  <img width="618" height="445" alt="image" src="https://github.com/user-attachments/assets/1ea95782-3251-41bc-8cda-a19b6405f23f" />
</details>

```
SELECT member_name,
	STATUS,
	SUM(amount * unit_price) AS costs
FROM FamilyMembers
	JOIN Payments ON FamilyMembers.member_id = Payments.family_member
WHERE YEAR(date) = 2005
GROUP BY member_name,
	STATUS
```

> Сортировку группировки по году нельзя сделать через HAVING, потому что HAVING используется для фильтрации групп после их агрегирования (SUM(amount*unit_price) > 100000), а не для фильтрации отдельных строк перед группировкой.

18. Выведите имя самого старшего человека. Если таких несколько, то выведите их всех.

```
SELECT member_name
FROM FamilyMembers
ORDER BY birthday ASC
LIMIT 1
```

```
SELECT member_name
FROM FamilyMembers
WHERE birthday = (
		SELECT MIN(birthday)
		FROM FamilyMembers
	)
```

19. Определить, кто из членов семьи покупал картошку (potato).

```
SELECT DISTINCT status
FROM FamilyMembers
  JOIN Payments ON FamilyMembers.member_id=Payments.family_member
  JOIN Goods ON Payments.good=Goods.good_id
WHERE good_name LIKE 'potato'
```

20. Сколько и кто из семьи потратил на развлечения (entertainment). Вывести статус в семье, имя, сумму.

```
SELECT STATUS,
	member_name,
	SUM(unit_price * amount) AS costs
FROM FamilyMembers
	JOIN Payments ON FamilyMembers.member_id = Payments.family_member
	JOIN Goods ON Payments.good = Goods.good_id
	JOIN GoodTypes ON Goods.type = GoodTypes.good_type_id
WHERE good_type_name LIKE 'entertainment'
GROUP BY STATUS,
	member_name
```

21. Определить товары, которые покупали более 1 раза.

```
SELECT good_name
FROM Goods
  JOIN Payments ON Goods.good_id=Payments.good
GROUP BY good_name
HAVING COUNT(*)>1
```

22. Найти имена всех матерей (mother).

```
SELECT member_name
FROM FamilyMembers
WHERE status LIKE 'mother'
```

23. Найдите самый дорогой деликатес (delicacies) и выведите его цену.

```
SELECT good_name,
	unit_price
FROM Goods
	JOIN Payments ON Goods.good_id = Payments.good
WHERE unit_price =(
		SELECT MAX(unit_price)
		FROM Payments
			JOIN Goods ON Payments.good = Goods.good_id
			JOIN GoodTypes ON Goods.type = GoodTypes.good_type_id
		WHERE good_type_name LIKE 'delicacies'
	)
```

24. Определить, кто и сколько потратил в июне 2005.

```
SELECT member_name,
	SUM(unit_price * amount) AS costs
FROM FamilyMembers
	JOIN Payments ON FamilyMembers.member_id = Payments.family_member
WHERE Payments.date BETWEEN '2005-06-01' AND '2005-06-30'
GROUP BY member_name
```

25. Определить, какие товары не покупались в 2005 году.

```
SELECT good_name
FROM Goods
WHERE good_id NOT IN(
		SELECT good
		FROM Payments
		WHERE YEAR(date) = 2005
	)
```

26. Определить группы товаров, которые не приобретались в 2005 году.

```
SELECT DISTINCT good_type_name
FROM GoodTypes
WHERE good_type_id NOT IN (
		SELECT good_type_id
		FROM GoodTypes
			JOIN Goods ON GoodTypes.good_type_id = Goods.type
			JOIN Payments ON Goods.good_id = Payments.good
		WHERE date BETWEEN '2005-01-01' AND '2005-12-31 '
	)
```

27. Cколько было потрачено на каждую из групп товаров в 2005 году. Выведите название группы и потраченную на неё сумму.

```
SELECT good_type_name,
	SUM(unit_price * amount) AS costs
FROM GoodTypes
	JOIN Goods ON GoodTypes.good_type_id = Goods.type
	JOIN Payments ON Goods.good_id = Payments.good
WHERE date BETWEEN '2005-01-01' AND '2005-12-31'
GROUP BY good_type_name
```

28. Сколько рейсов совершили авиакомпании из Ростова (Rostov) в Москву (Moscow) ?

```
SELECT COUNT(*)
FROM Trip
WHERE town_from LIKE 'Rostov'
	AND town_to LIKE 'Moscow'
```

29. Выведите имена пассажиров, улетевших в Москву (Moscow) на самолете TU-134. В ответе не должно быть дубликатов.

```
SELECT name
FROM Passenger
JOIN Pass_in_trip ON Passenger.id=Pass_in_trip.passenger
JOIN Trip ON Pass_in_trip.trip=Trip.id
WHERE town_to='Moscow' AND plane='TU-134'
GROUP BY name
```

```
SELECT DISTINCT name
FROM Passenger
	JOIN Pass_in_trip ON Passenger.id = Pass_in_trip.passenger
	JOIN Trip ON Pass_in_trip.trip = Trip.id
WHERE plane like 'TU-134'
	AND town_to like 'Moscow'
```

30. Вывести количество занятых мест по каждому рейсу из таблицы Pass_in_trip, отсортировав результат по убыванию количества занятых мест.

```
SELECT trip, COUNT(passenger) AS count
FROM Pass_in_trip
GROUP BY trip
ORDER BY count DESC
```

31. Вывести всех членов семьи с фамилией Quincey.

```
SELECT *
FROM FamilyMembers
WHERE member_name LIKE '%Quincey%'
```

32. Вывести средний возраст людей (в годах), хранящихся в базе данных. Результат округлите до целого в меньшую сторону.

```
SELECT FLOOR(AVG(YEAR(CURRENT_DATE) - YEAR(birthday))) AS age
FROM FamilyMembers
```

33. Найдите среднюю цену икры на основе данных, хранящихся в таблице Payments. В базе данных хранятся данные о покупках красной (red caviar) и черной икры (black caviar). В ответе должна быть одна строка со средней ценой всей купленной когда-либо икры.

```
SELECT AVG(unit_price) AS cost
FROM Payments
	JOIN Goods ON Payments.good = Goods.good_id
WHERE good_name LIKE '%caviar'
```

34. Сколько всего 10-ых классов?

<details>
	<summary>ERD</summary>
	<br>
	<img width="641" height="407" alt="image" src="https://github.com/user-attachments/assets/eb50b76c-3074-4e01-858d-17a510c2b633" />
</details>

```
SELECT COUNT(name) AS COUNT
FROM Class
WHERE name LIKE '10%'
```

35. Сколько различных кабинетов школы использовались 2 сентября 2019 года для проведения занятий?

```
SELECT COUNT(DISTINCT classroom) AS count
FROM Schedule
WHERE DATE(date) = '2019-09-02'
```

36. Выведите информацию об обучающихся, живущих на улице Пушкина (ul. Pushkina)?

```
SELECT *
FROM Student
WHERE address LIKE '%Pushkina%'
```

37. Сколько лет самому молодому обучающемуся ?

```
SELECT TIMESTAMPDIFF(YEAR, birthday, current_date) AS year 
FROM Student
ORDER BY birthday DESC 
LIMIT 1
```

```
SELECT MIN(TIMESTAMPDIFF(YEAR, birthday, CURRENT_DATE)) AS year 
FROM Student
```

38. Сколько учениц с именем Анна (Anna) учится в школе?

```
SELECT COUNT(*) AS COUNT
FROM Student
WHERE first_name = 'Anna'
```

39. Сколько обучающихся в 10 B классе ?

```
SELECT COUNT(student) AS COUNT
FROM Student_in_class
	JOIN Class ON Student_in_class.class = Class.id
WHERE name = '10 B'
```

```
SELECT COUNT(student) AS COUNT
FROM Student_in_class
WHERE class = (
		SELECT id
		FROM Class
		WHERE name = '10 B'
	)
```

40. Выведите название предметов, которые преподает Ромашкин П.П. (Romashkin P.P.). Обратите внимание, что в базе данных есть несколько учителей с такой фамилией.

```
SELECT name AS subjects
FROM Subject
	JOIN Schedule ON Subject.id = Schedule.subject
	JOIN Teacher ON Schedule.teacher = Teacher.id
WHERE first_name LIKE 'P%'
	AND middle_name LIKE 'P%'
	AND last_name LIKE 'Romashkin'
```

41. Выясните, во сколько по расписанию начинается четвёртое занятие.

```
SELECT DISTINCT start_pair
FROM Timepair
	JOIN Schedule ON Timepair.id = Schedule.number_pair
WHERE number_pair = 4
```

42. Сколько времени обучающийся будет находиться в школе, учась со 2-го по 4-ый уч. предмет?

```
SELECT DISTINCT TIMEDIFF(
		(
			SELECT end_pair
			FROM Timepair
			WHERE id = 4
		),
		(
			SELECT start_pair
			FROM Timepair
			WHERE id = 2
		)
	) AS time
FROM Timepair
```

43. Выведите фамилии преподавателей, которые ведут физическую культуру (Physical Culture). Отсортируйте преподавателей по фамилии в алфавитном порядке.

```
SELECT last_name
FROM Teacher
	JOIN Schedule ON Teacher.id = Schedule.teacher
	JOIN Subject ON Schedule.subject = Subject.id
WHERE name LIKE 'Physical Culture'
ORDER BY last_name ASC
```

44. Найдите максимальный возраст (количество лет) среди обучающихся 10 классов на сегодняшний день. Для получения текущих даты и времени используйте функцию NOW().

```
SELECT MAX(TIMESTAMPDIFF(YEAR, birthday, CURRENT_DATE)) AS max_year
FROM Student
	JOIN Student_in_class ON Student.id = Student_in_class.student
	JOIN Class ON Student_in_class.class = Class.id
WHERE name LIKE '10%'
```

45. Какие кабинеты чаще всего использовались для проведения занятий? Выведите те, которые использовались максимальное количество раз.

```
SELECT classroom
FROM Schedule
	JOIN Class ON Schedule.class = Class.id
	JOIN Timepair ON Schedule.number_pair = Timepair.id
	JOIN Teacher ON Schedule.teacher = Teacher.id
	JOIN Subject ON Schedule.subject = Subject.id
GROUP BY classroom
ORDER BY COUNT(classroom) DESC
LIMIT 3
```

```
SELECT classroom
FROM Schedule
GROUP BY classroom
HAVING COUNT(*) = (
		SELECT MAX(count)
		FROM (
				SELECT COUNT(*) AS count
				FROM Schedule
				GROUP BY classroom
			) AS counts
	)
```

46. В каких классах введет занятия преподаватель "Krauze" ?

```
SELECT name
FROM Class
	JOIN Schedule ON Class.id = Schedule.class
	JOIN Teacher ON Schedule.teacher = Teacher.id
WHERE last_name LIKE 'Krauze'
GROUP BY name
```

47. Сколько занятий провел Krauze 30 августа 2019 г.?

```
SELECT COUNT(date) AS COUNT
FROM Schedule
	JOIN Teacher ON Schedule.teacher = Teacher.id
WHERE last_name LIKE 'Krauze'
	AND date = '2019-08-30'
```

48. Выведите заполненность классов в порядке убывания.

```
SELECT name,
	COUNT(student) AS COUNT
FROM Class
	JOIN Student_in_class ON Class.id = Student_in_class.class
GROUP BY name
ORDER BY COUNT(student) DESC
```

49. Какой процент обучающихся учится в "10 A" классе? Выведите ответ в диапазоне от 0 до 100 с округлением до четырёх знаков после запятой, например, 96.0201.

```
WITH a_10 AS (
	SELECT COUNT(class) AS count_10
	FROM Student_in_class
		JOIN Class ON Student_in_class.class = Class.id
	WHERE name = '10 A'
), all_10 AS (
	SELECT COUNT(class) AS count_all
	FROM Student_in_class
)
SELECT (count_10 / count_all) * 100 AS percent
FROM a_10,
	all_10
```

50. Какой процент обучающихся родился в 2000 году? Результат округлить до целого в меньшую сторону.

```
WITH a_2000 AS (
	SELECT COUNT(*) AS count_10
	FROM Student
	WHERE YEAR(birthday) = 2000
), all_10 AS (
	SELECT COUNT(*) AS count_all
	FROM Student
)
SELECT 
    FLOOR(count_10 * 100.0 / count_all) AS percent
FROM a_2000, all_10
```

51. Добавьте товар с именем "Cheese" и типом "food" в список товаров (Goods).

```
INSERT INTO Goods (good_id, good_name, type)
VALUES(
		(
			SELECT MAX(good_id) + 1
			FROM (SELECT * FROM Goods) AS temp
		),
		'Cheese',
		(
			SELECT good_type_id
			FROM GoodTypes
			WHERE good_type_name = 'food'
		)
	)
```

52. Добавьте в список типов товаров (GoodTypes) новый тип "auto".

```
INSERT INTO GoodTypes
VALUES(
		(
			SELECT MAX(good_type_id) + 1
			FROM (
					SELECT *
					FROM GoodTypes
				) AS temp
		),
		'auto'
	)
```

53. Измените имя "Andie Quincey" на новое "Andie Anthony".

```
UPDATE FamilyMembers
SET member_name = 'Andie Anthony'
WHERE member_name = 'Andie Quincey'
```

54. Удалить всех членов семьи с фамилией "Quincey".

```
DELETE FROM FamilyMembers
WHERE member_name LIKE '%Quincey'
```

55. Удалить компании, совершившие наименьшее количество рейсов.

```
WITH trips AS (
	SELECT company,
		COUNT(*) AS count_
	FROM Trip
	GROUP BY company
)
DELETE FROM Company
WHERE id IN (
		SELECT company
		FROM trips
		WHERE count_ =(
				SELECT MIN(count_)
				FROM trips
			)
	)
```

56. Удалить все перелеты, совершенные из Москвы (Moscow).

```
DELETE FROM Trip
WHERE town_from = 'Moscow'
```

57. Перенести расписание всех занятий на 30 мин. вперед.

```
UPDATE Timepair
SET start_pair = TIMESTAMPADD(MINUTE, 30, start_pair),
	end_pair = TIMESTAMPADD(MINUTE, 30, end_pair)
```
