Service contains mainly 5 blocks – URI block,SQL block,Format block,Error Handler block and evaluator block

Once the URI is specified in the browser, service.py is opened by soft link REST located in cgibin directory.
From this point service.py works in full independence.

URI block:
- Gets the URL from browser by os.environ()
- Splits the given URL with the help of urlparse.urlparse() into different components like service root URL, query
  and resource path. 
- MyREST fixes the size of service path to 3 which comprises of devi.com/cgibin/REST.
- Prepares resource key list which comprises of all table names along with primary key if specified.
  MyREST takes only 1 primary key for now.
- Returns resource key list and parsed URL to the main() function.

SQL block:
- Intermediate driver function called delegateResponsibilityToSQL() handles in resource key list and query dictionary
  to getSQLQuery() function.
- This function prepares the select clause, from clause where clause and order by clause.
- Select clause is prepared by getSelect() which takes in query dictionary to handle $select situation.
- From clause is prepared by getFrom() which takes in resource key list to construct the join of set of tables.
- Where clause is prepared by getWhere() which takes  in resource key list and query dictionary. Resource key list 
  is used to prepare the join condition between the primary key and foreign keys. Query dictionary is used to 
  address $filter situations.
- Order by clause is prepared by getOrder() which takes in query dictionary and returns order based on $order condition.
- With the help of all above clauses SQL query is formed and result is returned by fetching records from the 
  database cursor.
  
Format Block:
- This block takes result and column names which were returned by SQL block and querying INFORMATION_SCHEMA 
  database respectively.
- It also takes query dictionary to analyze the $format query parameter if specified. Default format is JSON.

Error Handler block:
- Error handling happens in SQL, empty result and improper format specification situations.
- In SQL, DATA ERROR(numeric value out of range or division by zero),INTEGRITY ERROR(relational integrity is not 
  adhered to),OPERATIONAL ERROR(unexpected disconnect or data source not found),
  PROGRAMMING ERROR(table not found or already exists or a wrong number of parameters is given) is handled.
- If a primary key does not exist for a table or if the table name is not specified properly, querying the
  INFORMATION_SCHEMA results in None. Error handling is done here to detect invalid table name.
- If improper format is specified in $format apart from json,xml or html FORMAT error is reported.
- Error Handler block takes in expression specified in $filter and replaces %20,%27,%28,%29 from the URL with 
  ' ','\'','(' and ')' respectively.
- It also replaces all logical and arithmetic operations which OData specifies.
- Major set of canonical functions which are supported are replaced according to MySQL standards.
- MyREST does not support nested functional calls.Implementation for such complex queries is left for
  future scope.

HTTP codes used in MyREST:
- All operations which remain successful including error handling situations are returned with HTTP 200(OK)
- All operations which return empty records are returned with HTTP 404(NOT FOUND)