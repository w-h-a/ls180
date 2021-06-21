Here's my _second_ walk through of assignment 10.

PART I

```sql
create table star(
  star_id int generated always as identity primary key,
  star_name varchar(25) unique not null,
  distance_from_earth int not null check(distance_from_earth > 0),
  spectral_type char(1) check(spectral_type in ('O', 'B', 'A', 'F', 'G', 'K', 'M')),
  companions int not null check(companions > -1)
);

create table planet(
  planet_id int generated always as identity primary key,
  designation char(1) unique,
  mass int
);
```

PART II

```sql
create table orbits(
  planet_id int references planet(planet_id) on delete cascade,
  star_id int references star(star_id) on delete cascade,
  primary key (planet_id, star_id)
);
```

PART III

```sql
alter table star
alter column star_name type varchar(50);
```

PART IV

```sql
alter table star
alter column distance_from_earth type numeric;
```

PART V

```sql
alter table star
alter column spectral_type
  set not null;
```

PART VI

```sql
alter table star
drop constraint star_spectral_type_check;

create type spectral_type_enum as enum('O', 'B', 'A', 'F', 'G', 'K', 'M');

alter table star
alter column spectral_type type spectral_type_enum
  using spectral_type::spectral_type_enum;
```

PART VII

```sql
alter table planet
alter column mass type numeric,
alter column mass
  set not null,
alter column designation
  set not null,
add check(mass > 0);
```

PART VIII

```sql
alter table planet
add column semi_major_axis numeric not null;
```

PART IX

```sql
create table moon(
  moon_id int generated always as identity primary key,
  designation int not null check(designation > 0),
  semi_major_axis numeric check(semi_major_axis > 0),
  mass numeric check(mass > 0)
);

alter table orbits
drop constraint orbits_pkey;

alter table orbits
add column moon_id int references moon(moon_id) on delete cascade,
add primary key (moon_id, planet_id, star_id);
```
