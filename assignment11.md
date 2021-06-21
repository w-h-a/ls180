Here is my walk through for assignment 11.

PART I

The entity classes include: _device_ and _part_. The relationship classes include: _contains_. The _contains_ relationship is many-to-one. Here's the relevant cardinality function:

`card(contains, device) = MANY`

`card(contains, part) = ONE`

Finally, here are the relevant relation schemas:

```sql
create table device(
  device_id int generated always as identity primary key,
  device_name text not null,
  created_at timestamp default current_timestamp
);

create table part(
  part_id int generated always as identity primary key,
  part_number int unique not null
);

create table contains(
  device_id int references device(device_id) on delete cascade,
  part_id int unique references part(part_id) on delete cascade,
  primary key (device_id, part_id)
);
```

PART II

To construct our relations:

```sql
insert into device(device_name)
values
  ('Accelerometer'),
  ('Gyroscope');

insert into part(part_number)
values
  (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), (11);

insert into contains(device_id, part_id)
values
  (1, 1), (1, 2), (1, 3),
  (2, 4), (2, 5), (2, 6), (2, 7), (2, 8);
```

PART III

To return a virtual relation between `device_name` and `part_number`:

```sql
select
  device_name "device name",
  part_number "part identifier"
from device
inner join contains using(device_id)
inner join part using(part_id)
order by "device name"
fetch first 10 rows only;

select
  device_name "device name",
  part_number "part identifier"
from device
inner join contains on contains.device_id = device.device_id
inner join part on part.part_id = contains.part_id
order by "device name"
fetch first 10 rows only;
```

PART IV

To return elements of the part relation where the value of `part_number` starts with 3:

```sql
select *
from part
where
  cast(part_number as text) ilike '3%';
```

PART V

To return a virtual relation between `device_name` and the count of parts:

```sql
select
  device_name "device name",
  count(part_id) "part count"
from device
left join contains using(device_id)
left join part using(part_id)
group by device_id
order by "part count" desc
fetch first 10 rows only;
```

PART VI

```sql
select
  device_name "device name",
  count(part_id) "part count"
from device
left join contains using(device_id)
left join part using(part_id)
group by device_id
order by "device name" desc
fetch first 10 rows only;
```

PART VII

To return a virtual inner relation between `part_number` and `device_id`:

```sql
select
  part_number,
  device_id
from contains
inner join part using(part_id);
```

To return a virtual outer null relation between `part_number` and `device_id`:

```sql
select
  part_number,
  device_id
from contains
right join part using(part_id)
where device_id is null;
```

PART VIII

To update our relations:

```sql
insert into device (device_name)
values ('Magnetometer');

insert into part (part_number)
values (42);

insert into contains (device_id, part_id)
values (3, 12);
```

To return the names of the oldest devices:

```sql
select string_agg(device_name, ', ') "oldest devices"
from device
where
  (now() - created_at) = some(
    select max(now() - created_at)
    from device
  )
group by (now() - created_at);
```

I was overthinking this a bit.


PART IX

To update our contains relation:

```sql
update contains
set device_id = 1
where
  part_id = some(
    select part_id
    from contains
    where device_id = 2
    order by part_id desc
    fetch first 2 rows only
  );
```

```sql
update contains
set device_id = 2
from part
where
  part.part_id = contains.part_id and
  part_number = some(
    select part_number
    from part
    order by part_number
    fetch first row only
  );
```

PART X

To delete:

```sql
delete from part
where
  part_id = some(
    select part_id
    from contains
    where
      device_id = 1
  );

delete from device
where device_name = 'Accelerometer';

delete from part
where
  part_id != all(
    select part_id
    from contains
  );
```
