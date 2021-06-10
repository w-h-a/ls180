## It's Schemas All The Way Down!

From the [public Launch School material](https://launchschool.com/books/sql/read/introduction):

> In simple terms, the relational model defines a set of relations (which [are represented by] tables) and describes the relationships, or connections, between them [...] to determine how the data stored in them can interact.

I realize the 'relation model' term comes from Codd. But, assuming I understand where Codd and the Launch School material are coming from, I believe the preferred term nowadays (at least in some circles) is 'relational schema' (which is not synonymous with 'relation schema', as we'll see). I think there are good reasons for this. Typically we think of models as being compared to some concrete system. But it's a bit strange to think of relational schemas as being compared to concrete systems, as they are just empty frameworks for scaffolding relational databases. Moreover, although 'model' is sometimes used to _prescribe_ as well as _describe_, I think 'schema' better connotes the idea that the purpose of a relational schema is first and foremost to _prescribe_ how we should structure and store our data of concrete systems. So, hereafter I drop the term 'relational model' in the hopes of making sense of relational schemas, database schemas, and relation schemas.

Often in the domain of relational databases, Entity-Relationship-diagrams (ER diagrams) are used as descriptions of relational schemas. But ordinary English works as well. Here's an example:

> A tech company consists of employees, products, and business-units. Employees have a salary and a skill-level, products are either successful or not, and business-units have revenue and either a tight budget or not. An employee can work on many products, and a product can be worked on by many employees. In addition, a business-unit can be funded by many products. However, a product funds at most one business-unit.

Our tech company description at least partially specifies a relational schema. But to create a full-blown relational schema of a tech company, we need to ensure that our relational schema consists of _three_ essential features.

_First_, our relational schema must contain a set of _classes_. Let's denote these by '**I**'. The set **I** is typically and usefully partitioned into **E**, the set of entity classes, and **R**, the set of relationship classes. What are these? Here is a rough way of thinking about classes, generally:

> A class of a property is the extension of the property. A class contains all and only individuals that have the property.

Consider first the _entity_ classes of our tech company. In this example, the employee class, the product class, and the business-unit class are the elements of **E**. The _employee_ class is an abstract object that is the extension of the concrete property of _being an employee at our tech company_. You may accordingly think of it as the set of employees at our tech company. We may say similar things about the product class (and the corresponding property of _being a product at our tech company_) and the business-unit class (and the corresponding property of _being a business-unit at our tech company_).

Let's not forget about the _relationship_ classes. In our example, the develops class and the funds class are the elements of **R**. The _develops_ class is an abstract object that is the extension of the concrete relation of _being an employee that develops a product at our tech company_. You may accordingly think of it as the set of all such employee-product pairs at our tech company. We may say similar things about the funds class (and the corresponding relation of _being a product that funds a business-unit at our tech company_).

_Second_, our schema is a _relational_ schema because it contains a database schema! In particular, the database schema contains a relation schema for each class. Each relation schema defines the set of attributes for the class. The relation schema itself is a (finitary) relation:

> S({set of identifiers}, {set of data types}, {set of constraints})

What's going on here? Generally, _S_ takes a particular set of identifiers, a particular set of data types (e.g., `varchar(50)`, `boolean`, `decimal(8, 2))`, and a particular set of constraints (e.g., `unique`, `not null`, `primary key`, `foreign key`) as input and yields a (usually proper) subset of the Cartesian product of the three sets. So, for example, the relation schema for _product_ is a particular subset of all possible ordered triples of:

> ({id, name, market_success}, {integer, boolean, varchar(20), ...}, {unique, not null, primary key, foreign key, ...})

_That_ relation schema or subset of all possible ordered triples is:

> P({id, name, market_success}, {integer, boolean, varchar(20), ...}, {unique, not null, primary key, foreign key, ...})

Each such ordered triple in the above set is an attribute of _product_. We may say similar things about our other classes, including our relationship classes. For example, the relation schema for _funds_ is the subset of all possible ordered triples:

> F({id, product_id, business_unit_id}, {integer, boolean, varchar(20), ...}, {unique, not null, primary key, foreign key, ...})

The set of such ordered triples would be the attributes of _funds_.

The _third_ ingredient of our _relational_ schema is that it contains a _cardinality_ function. To get a better idea of this, we need to be clear about functions. A `card` function ain't the loose sort of function in your code (unless your functional programming). A `card` function is a pure total function; it's a mathematical function. For example, a 2-place relation _Q_ is a unary total function whenever, for every argument _x_ of set _X_, there is a unique return value _y_ of set _Y_ such that _x_ is _Q_-related to _y_. More generally,

> An _n'_-place relation _Q_ is a total function with _n_-arity whenever, for all arguments _x1 of X1, ..., xn of Xn_, there is a unique return value _y_ of _Y_ such that _x1, ..., xn, y_ exemplify _Q_.

In our case, a `card` is a binary total function such that the first argument is any _R_ of **R**, and the second argument is one of _R_'s entity classes _E_. Given any two such arguments, there is a unique return value of the set _{ONE, MANY}_. The value returned depends on how many instances of _R_ within which _an_ instance of _E_ _can_ participate. Here's an example `card` (a `card` with a particular signature):

```text
// an employee can participate in many instances of develops
card(develops, employee) = MANY;

// a product can participate in many instances of develops
card(develops, product) = MANY;

// a product can participate in at most one instance of funds
card(funds, product) = ONE;

// a business-unit can participate in many instances of funds
card(funds, business-unit) = MANY;
```

So, pulling all of this together, our relational schema is identical to the abstract object that contains a set of classes **I**, a database schema (or set of relation schemas), and a `card`.

To put this into practice, let's walkthrough implementing a relational schema for the tech corporation in psql. Fire up psql and write:

```sql
create database tech_comp;
\c tech_comp
```
