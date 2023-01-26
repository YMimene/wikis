# Using-Query-Builder-and-Testing-Queries

## How to test QB queries:
There are two types of queries:

### Independant queries
- The queries that can be executed directly and these queries are not changed inside a function
- To test these queries, we can save the old query (written in raw SQL) in the test file and then run both queries (raw SQL and Query Builder) and check if it gives the same results
- In the case of (INSERTING, DELETING, UPDATING) we may need to do a rollback between the two queries to start with the same data before running the query
- Or we can write a full senario in the test file without need to save the old query

### methods queries
- These are the queries that get changed inside a function, in this case we should test the whole function and not only the query
- We save the old query and the related function in the test file and we convert the query and the function to QB
- After that, we test if the results given by the QB (query + function) are the same as the old (query + function)
- We may also need to rollback between the two executions

**Note 1:** Some method queries are very complex and it may be very difficult to test them by unit tests, so we need to test the results manually from Desk.  
**Note 2:** Some method queries may be very difficult to convert them to QB (we need to discuss what should we do in this case)

## Where to put the test file
There are 3 levels of tests:

#### Doctype tests
In this type, the python test file is already created inside the doctype folder

#### Module tests
These are the queries that are in the module file, we need to create a folder **tests** inside the module where we put the python test files.

#### App tests
These are the queries that are in the app level, we need to create a folder **tests** inside the app folder where we put the python test files.


## Resources:
- [Frappe query Builder](https://frappeframework.com/docs/v14/user/en/api/query-builder)
- [Frappe unit tests](https://frappeframework.com/docs/v14/user/en/guides/automated-testing/unit-testing)
