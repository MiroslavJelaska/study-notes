#Basic definitions
###Table
In terms of the relational model of databases, a table can be considered a convenient representation of a relation, but the two are not strictly equivalent. For instance, an SQL table can potentially contain duplicate rows, whereas a true relation cannot contain duplicate tuples. Similarly, representation as a table implies a particular ordering to the rows and columns, whereas a relation is explicitly unordered. However, the database system does not guarantee any ordering of the rows unless an ORDER BY clause is specified in the SELECT statement that queries the table.
###Column
In the context of a relational database table, a column is a set of data values of a particular simple type, one for each row of the table. The columns provide the structure according to which the rows are composed.

The term field is often used interchangeably with column, although some consider it more correct to use field (or field value) to refer specifically to the single item that exists at the intersection between one row and one column.

###Tuple
Basically the same thing as a row. Tuples are not ordered; instead, each attribute value is identified solely by the attribute name and never by its ordinal position within the tuple.

###Atomic value
A piece of data in a database table that cannot be broken down any further.

From a purely practical point of view, we shall define an atomic attribute as an attribute that, in a where clause, can always be referred to in full. You can split and chop an attribute as much as you want in the select list (where it is returned); but if you need to refer to parts of the attribute inside the where clause, the attribute lacks the level of atomicity you need. <sup>[TAS]</sup>

#Functional dependancy
###Functional dependancy
Given a relation R, a set of attributes X in R is said to functionally determine another set of attributes Y, also in R, (written X → Y) if, and only if, each X value is associated with precisely one Y value; R is then said to satisfy the functional dependency X → Y.Customarily X is called the determinant set and Y the dependent set. A functional dependency FD: X → Y is called trivial if Y is a subset of X. <sup>[WIKI](http://en.wikipedia.org/wiki/Functional_dependency)</sup>

[Armstrong's axioms.](http://en.wikipedia.org/wiki/Armstrong%27s_axioms)

###Superkey
A superkey is defined in the relational model of database organization as a set of attributes of a relation variable for which it holds that in all relations assigned to that variable, there are no two distinct tuples (rows) that have the same values for the attributes in this set.<sup>[1]</sup> Equivalently a superkey can also be defined as a set of attributes of a relation schema upon which all attributes of the schema are functionally dependent.

Note that the set of all attributes is a trivial superkey, because in relational algebra duplicate rows are not permitted.
**Minimal superkey**, a minimal set of attributes that can be used to identify a single tuple. **Candidate key** is minimal superkey.<sup>[2]</sup>

#Normal Forms

The normalization process is fundamentally based on the application of atomicity to the world you are modeling.<sup>[TAS]</sup>
Normalization is basically to design a database schema such that duplicate and redundant data is avoided. If some piece of data is duplicated several places in the database, there is the risk that it is updated in one place but not the other, leading to data corruption.<sup>[1]</sup>

List of Normal forms:
* First Normal Form (1NF)
* Second Normal Form (2NF)
* Third Normal Form (3NF)
* Boyce-Codd Normal Form (BCNF)
* Fourth Normal Form (4NF)
* Fifth Normal Form (5NF)
* Domain/Key Normal Form (DKNF)

You can have 3NF but not BCNF if your table contains several sets of columns that are unique (candidate keys, which are possible unique identifiers of a row) and share one column. Such situations are not very common. In the vast majority of cases, a database modeled in 3NF will also be in BCNF and 5NF.<sup>[TAS]</sup>
##1NF
###From [TAS] - Ensure Atomicity
2 major benefits:
* The ability to perform an efficient search
* Database-guaranteed data correctness
Defining data atoms isn’t always a simple exercise. We should not create design problems for information we don’t need to process.

The next step is to identify what uniquely characterizes a row—the primary key.At this stage, it is very likely that this key will be a compound one. As a general rule, you should, whenever possible, use a unique identifier that has meaning rather than some obscure sequential integer. I must stress that the primary key is what characterizes the data—which is not the case with some sequential identifier associated with each new row. You can even promote the sequential identifier to the envied status of primary key, as a technical substitute (or shorthand) for the true key, in exactly the same way that you’d use table aliases in a query in order to be able to write:

```where a.id = b.id```

instead of:

```where table_with_a_long_name.id = table_even_worse_than_the_other.id```

But a technical, numerical identifier doesn’t constitute a real primary key by the mere virtue of its existence and mustn’t be mistaken for the real thing. Once all the attributes are atomic and keys are identified, our data is in first normal form (1NF).

###From [BDD]
First normal form is the most important, and essentially says that we should not try to cram several pieces of data into a single field.
####Definition
 A table is not in first normal form if it is keeping multiple values for a piece of information.
####Fix
If a table is not in first normal form, remove the multivalued information from the table. Create a new table with that information and the primary key of the original table.

###From [BDS]
Qualifications:
* Each column must have a unique name.
* The order of the rows and columns doesn’t matter.
* Each column must have a single data type.
* No two rows can contain identical values.
* Each column must contain a single value.
* Columns cannot contain repeating groups.

This reminds me of the joke where some guy wants to buy two new residents for his aquarium by mail-order (this was before Internet shopping) but he doesn’t know whether the plural of octopus is octopi or octopuses. So he writes, ‘‘Dear Sirs, please send me an octopus. Oh and please send me another one.’’

##2NF
###From [TAS] - Check Dependence on the Whole Key
Motivation:
* Data redundancy
* Query performance

To remove dependencies on a part of the key, we must create tables. The keys of those new tables will each be a part of the key for our original table. Then we must move all the attributes that depend on those new keys to the new tables, and retain only some in the original table.

###From [BDD]
####Definition
A table is in second normal form if it is in first normal form AND we need ALL the fields in the key to determine the values of the non–key fields.
####Fix
If a table is not in second normal form, remove those non–key fields that are not dependent on the whole of the primary key. Create another table with these fields and the part of the primary key on which they do depend.

Splitting up of an unnormalized table is often referred to as *decomposition*.

###From [BDS]
1. It is in 1NF.
2. All of the non-key fields depend on all of the key fields.

##3NF
###From [TAS] - Check Attribute Independence
Very often, a data set in 2NF will already be in 3NF. 3NF is reached when we cannot infer the value of an attribute from any attribute other than those in the unique key. For example, the question must be asked: “Given the value of attribute A, can the value of attribute B be determined?”

Motivation
* A properly normalized model protects against the evolution of requirements.
* Normalization minimizes data duplication.

Note that there are cases in which designers deliberately choose not to model in third normal form; dimensional modeling.
###From [BDD]
####Definition
A table is in third normal form if it is in second normal form AND no non–key fields depend on a field(s) that is not the primary key.
####Fix
If a table is not in third normal form, remove the non–key fields that are dependent on a field(s) that is not the primary key. Create another table with this field(s) and the field on which it does depend.

###From [BDS]
1. It is in 2NF.
2. It contains no transitive dependencies.

A *transitive dependency* is when one non-key field’s value depends on another non-key field’s value.

##Boyce-Codd Normal Form (BCNF)
###From [BDD]
This is the last normal form that involves functional dependencies. For most tables, it is equivalent to third normal form, but it is a slightly stronger statement for some tables where there is more than one possible combination of fields that could be used as a primary key.
####Definition
A table is in Boyce–Codd normal form if every determinant could be a primary key.

Bill Kent:
```
A table is based on
the key,
the whole key,
and nothing but the key (so help me Codd)
```
###From [BDS]
1. It is in 3NF.
2. Every determinant is a candidate key.

A *determinant* is a field that at least partly determines the value in another field.Note that the definition of 3NF worries about fields that are dependent on another field that is not part of the primary key. Now we’re talking about fields that might be dependent on fields that are part of the primary key (or any candidate key).

#Resources
* TAS [The Art of SQL by Stephane Faroult, Peter Robson](https://www.goodreads.com/book/show/1032724.The_Art_of_SQL)
* BDD [Beginning Database Design: From Novice to Professional](http://www.goodreads.com/book/show/2084821.Beginning_Database_Design)
* BDS [Beginning Database Design Solutions](http://www.goodreads.com/book/show/6470360-beginning-database-design-solutions)
* 1 [SO: What is Normalisation (or Normalization)?](http://stackoverflow.com/questions/246701/what-is-normalisation-or-normalization)
* 2 [SO: What are the differences between a superkey and a candidate key?](http://stackoverflow.com/questions/4519825/what-are-the-differences-between-a-superkey-and-a-candidate-key)