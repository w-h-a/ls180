Here is my walk through for assignment 14.

PART I

A customer uses services, and a service is used by customers. The entity classes are: _customer_ and _service_. The relationship classes are: _uses_. The cardinality constraints are given by the following function:

```text
card(uses, customer) = MANY
card(uses, service) = MANY
```

The necessity (modality) constraints are not-necessarily (possibly-not) and not-necessarily. Here are our implemented relation schemas (one for each item class):

```sql
create table customer(
  customer_id int generated always as identity primary key,
  customer_name text not null,
  payment_token char(8) not null unique check(payment_token ~ '^[A-Z]{8}$')
);

create table service(
  service_id int generated always as identity primary key,
  service_description text not null,
  price numeric(10, 2) not null check(price >= 0.00)
);

create table uses(
  customer_id int references customer(customer_id) on delete cascade,
  service_id int references service(service_id),
  primary key (customer_id, service_id)
);
```

There are many conventions in database practice. In some traditions the join relation would be called "customer_service". But others think that is not so clear. Since in other traditions that join relation actually corresponds to a relationship class (in a Chen ER-diagram), it makes more sense to title the relation by the name of the relationship class's name. Let a thousand flowers bloom, I say. Also, you get more semantic information that way. Okay, now to construct our relations:

```sql
insert into customer(customer_name, payment_token)
values
  ('Pat Johnson', 'XHGOAHEQ'),
  ('Nancy Monreal', 'JKWQPJKL'),
  ('Lynn Blake', 'KLZXWEEE'),
  ('Chen Ke-Hua', 'KWETYCVX'),
  ('Scott Lakso', 'UUEAPQPS'),
  ('Cyndee Pokorny', 'XKJEYAZA');

insert into service(service_description, price)
values
  ('Unix Hosting', 5.95),
  ('DNS', 4.95),
  ('Whois Registration', 1.95),
  ('High Bandwidth', 15.00),
  ('Business Support', 250.00),
  ('Dedicated Hosting', 50.00),
  ('Bulk Email', 250.00),
  ('One-on-one Training', 999.00); -- where they get you

insert into uses(customer_id, service_id)
values
  (1, 1), (1, 2), (1, 3),
  (3, 1), (3, 2), (3, 3), (3, 4), (3, 5),
  (4, 1), (4, 4),
  (5, 1), (5, 2), (5, 6),
  (6, 1), (6, 6), (6, 7);
```

PART II

To return all and only customer data on customers currently subscribed:

```sql
select *
from customer
where
  exists(
    select 1
    from uses
    where
      uses.customer_id = customer.customer_id
  );
```


PART III

To return all and only customer data on customers with null subscription:

```sql
select *
from customer
where
  not exists(
    select 1
    from uses
    where
      uses.customer_id = customer.customer_id
  );
```

To return a virtual full null relation:

```sql
select
  customer_name,
  payment_token,
  service_description,
  price
from customer
full join uses using(customer_id)
full join service using(service_id)
where
  customer_id is null or
  service_id is null;
```

PART IV

```sql
select service_description
from uses
right join service using(service_id)
where
 customer_id is null;
```

PART V

To return a relation between `customer_name` and the list of services the customer is using:

```sql
select
  customer_name,
  string_agg(service_description, ', ') as services
from customer
left join uses using(customer_id)
left join service using(service_id)
group by customer_name;
```

PART VI

To return a relation between `service_description` and the count of corresponding customers who use at least 3 services:

```sql
select
  service_description,
  count(customer_id)
from uses
inner join service using(service_id)
group by service_description
having
  count(customer_id) > 2
order by count
fetch first 2 rows only;
```

PART VII

```sql
select
  sum(price) as gross
from uses
inner join service using(service_id);  
```

PART VIII

```sql
insert into customer(customer_name, payment_token)
values
  ('John Doe', 'EYODHLCN');

insert into uses(customer_id, service_id)
values
  (7, 1), (7, 2), (7, 3);
```

PART XI

```sql
select
  sum(price) as expected
from uses
inner join service using(service_id)
where
  price > 100.00;

select
  sum(price) as counterfactual
from customer
cross join service
where
  price > 100.00;
```

PART X

```sql
delete from uses
where service_id = 7;

delete from service
where
  service_description = 'Bulk Email';

delete from customer
where
  customer_name = 'Chen Ke-Hua';
```
