# SQL

### [Тренажёр](https://sql-academy.org/ru/trainer)

1. Вывести имена всех людей, которые есть в базе данных авиакомпаний.

<details>
  <summary>ERD</summary>
  <br>

  <img width="640" height="451" alt="image" src="https://github.com/user-attachments/assets/90e3e238-8583-4394-af13-a11df491122b" />
</details>

```
SELECT name FROM Passenger
```

2. Вывести названия всеx авиакомпаний.

```
SELECT name FROM Company
```

3. Вывести все рейсы, совершенные из Москвы.

```
SELECT * FROM Trip
WHERE town_from LIKE 'Moscow'
```

4. Вывести имена людей, которые заканчиваются на "man".

```
SELECT name FROM Passenger
WHERE name LIKE '%man'
```

5. Вывести количество рейсов, совершенных на TU-134.

```
SELECT COUNT(*) AS count FROM Trip
WHERE plane LIKE 'TU-134'
```

6. Какие компании совершали перелеты на Boeing.

```
SELECT DISTINCT name FROM Company
JOIN Trip ON Company.id=Trip.company
WHERE plane LIKE 'Boeing%'
```

> DISTINCT используется для получения уникальных строк в наборе результатов.

7. Вывести все названия самолётов, на которых можно улететь в Москву (Moscow).

```
SELECT DISTINCT plane FROM Trip
WHERE town_to LIKE 'Moscow'
```

8. В какие города можно улететь из Парижа (Paris) и сколько времени это займёт? Формат для вывода времени: HH:MM:SS

```
SELECT town_to, TIMEDIFF(time_in, time_out) AS flight_time FROM Trip
WHERE town_from LIKE 'Paris'
```

9. Какие компании организуют перелеты из Владивостока (Vladivostok)?

```
SELECT name FROM Company
JOIN Trip ON Company.id=Trip.company
WHERE town_from LIKE 'Vladivostok'
```

10. Вывести вылеты, совершенные с 10 ч. по 14 ч. 1 января 1900 г.

```
SELECT * FROM Trip
WHERE time_out BETWEEN '1900-01-01 10:00:00' AND '1900-01-01 14:00:00'
```

11. Выведите пассажиров с самым длинным ФИО. Пробелы, дефисы и точки считаются частью имени.

```
SELECT name FROM Passenger
WHERE LENGTH(name) = (SELECT LENGTH(MAX(name)) FROM Passenger)
```

> WHERE LENGTH(name) = <результат подзапроса>: Фильтрует строки, оставляя только те, у которых длина имени (LENGTH(name)) равна числу, полученному во внутреннем запросе.

12. Выведите идентификаторы всех рейсов и количество пассажиров на них. Обратите внимание, что на каких-то рейсах пассажиров может не быть. В этом случае выведите число "0".

```
SELECT Trip.id as id, COALESCE(COUNT(Pass_in_trip.passenger), 0) AS count FROM Trip
LEFT JOIN Pass_in_trip ON Trip.id=Pass_in_trip.Trip
GROUP BY Trip.id
```

> GROUP BY используется в SQL для объединения строк, имеющих одинаковые значения в одном или нескольких столбцах, в одну сводную строку.

> LEFT JOIN используется для объединение с таблицей Pass_in_trip (столбец, в котором могут отсутствовать значения).

> COALESCE функция, которая возвращает первое не-NULL значение из списка переданных ей аргументов.
> Если все аргументы равны NULL, функция возвращает NULL. Ее часто используют для замены пропущенных значений на значения по умолчанию или для выбора наиболее релевантного значения из нескольких столбцов.  

13. Вывести имена людей, у которых есть полный тёзка среди пассажиров.

```
SELECT name FROM Passenger
GROUP BY name
HAVING COUNT(*)>1
```

> Cначала группирует записи по имени, затем удаляет все дубликаты (оставляя по одной записи для каждого уникального имени), и только после этого фильтрует полученные группы, оставляя только те, где количество записей (совпадающих имен) больше 1.

14. В какие города летал Bruce Willis.

```
SELECT town_to FROM Trip
JOIN Pass_in_trip ON Trip.id=Pass_in_trip.Trip
JOIN Passenger ON Pass_in_trip.passenger=Passenger.id
WHERE name LIKE 'Bruce Willis'
```

15. Выведите идентификатор пассажира Стив Мартин (Steve Martin) и дату и время его прилёта в Лондон (London).

```
SELECT Passenger.id, Trip.time_in FROM Passenger
JOIN Pass_in_trip ON Passenger.id=Pass_in_trip.passenger
JOIN Trip ON Pass_in_trip.trip=Trip.id
WHERE name LIKE 'Steve Martin' AND town_to LIKE 'London'
```

16. Вывести отсортированный по количеству перелетов (по убыванию) и имени (по возрастанию) список пассажиров, совершивших хотя бы 1 полет.

```
SELECT name, COUNT(Pass_in_trip.id) AS count FROM Passenger
JOIN Pass_in_trip ON Passenger.id=Pass_in_trip.passenger
GROUP BY name
HAVING count > 0
ORDER BY count DESC, name ASC
```

17. Определить, сколько потратил в 2005 году каждый из членов семьи. В результирующей выборке не выводите тех членов семьи, которые ничего не потратили.

<details>
  <summary>ERD</summary>
  <br>

  <img width="618" height="445" alt="image" src="https://github.com/user-attachments/assets/1ea95782-3251-41bc-8cda-a19b6405f23f" />
</details>

```
SELECT member_name, status, SUM(amount*unit_price) AS costs FROM FamilyMembers
JOIN Payments ON FamilyMembers.member_id=Payments.family_member
WHERE YEAR(date) = 2005
GROUP BY member_name, status
```

> Сортировку группировки по году нельзя сделать через HAVING, потому что HAVING используется для фильтрации групп после их агрегирования (SUM(amount*unit_price) > 100000), а не для фильтрации отдельных строк перед группировкой.

18. Выведите имя самого старшего человека. Если таких несколько, то выведите их всех.

```
SELECT member_name FROM FamilyMembers
ORDER BY birthday ASC
LIMIT 1
```

```
SELECT member_name FROM FamilyMembers
WHERE birthday = (SELECT MIN(birthday) FROM FamilyMembers)
```

19. Определить, кто из членов семьи покупал картошку (potato).

```
SELECT DISTINCT status FROM FamilyMembers
JOIN Payments ON FamilyMembers.member_id=Payments.family_member
JOIN Goods ON Payments.good=Goods.good_id
WHERE good_name LIKE 'potato'
```

20. Сколько и кто из семьи потратил на развлечения (entertainment). Вывести статус в семье, имя, сумму.

```
SELECT status, member_name, SUM(unit_price*amount) AS costs FROM FamilyMembers
JOIN Payments ON FamilyMembers.member_id=Payments.family_member
JOIN Goods ON Payments.good=Goods.good_id
JOIN GoodTypes ON Goods.type=GoodTypes.good_type_id
WHERE good_type_name LIKE 'entertainment'
GROUP BY status, member_name
```

21. Определить товары, которые покупали более 1 раза.

```
SELECT good_name FROM Goods
JOIN Payments ON Goods.good_id=Payments.good
GROUP BY good_name
HAVING COUNT(*)>1
```

22. Найти имена всех матерей (mother).

```
SELECT member_name FROM FamilyMembers
WHERE status LIKE 'mother'
```

23. 
