@tags blox

# Data Composition in Blox

Suppose you have two tables in your database:

- companies
- employees

Every employee has a foreign key to the company he works for.

Now suppose you want to build a result set that composites the data like this:

[:.resources/data-composition-structure.jpg]

A simple implementation might look like this:

[:.resources/data-composition-loop.png]

You loop over the companies and for each company you select the required employees.
While this will obviously work, it is not a very performant option.

Suppose we have 10 companies in our initial result set, this is the result of a single select.
We then loop 10 times and do an additional database select statement, this means we suddenly do **11 database calls**.
When the composition gets larger (e.g. multiple things to compose) or the iterations get bigger, this amount of database calls can explode.

While a single database call will generally be more performant than any application-level logic you can think off, once you start doing multiple calls, performance tends to drop steeply.

There are two alternatives to this:

- depending on the usecase, you might be able to "flatten" your resultset and perform it into a single query. This query can be constructed visually with CRUD or (in more complex usecases), custom SQL can be used for this. 
- the same resultset as in the above, but with fewer database calls

Especially in web applications the first solution is often a good one because you are feeding a very specific screen, meaning a tailored output is generally acceptable. For generic API's however you may want to retain the composition so let's zoom in on the second approach.

The general setup is like this:

[:.resources/data-composition-fast-loop.png]

We perform 2 selects, regardless of how many companies there are:

- the first one is to retrieve the companies
- the second one selects all the employees for all the companies

So the first select is unaltered compared to the original, the second select looks something like this:

[:.resources/select-all-employees.png]

We simply pass along all the ids of the relevant companies.

In the loop we map all the employees for that given company:

[:.resources/data-composition-filter.png]