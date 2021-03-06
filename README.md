# Lockit SQL adapter

[![Build Status](https://travis-ci.org/zeMirco/lockit-sql-adapter.svg?branch=master)](https://travis-ci.org/zeMirco/lockit-sql-adapter) [![NPM version](https://badge.fury.io/js/lockit-sql-adapter.svg)](http://badge.fury.io/js/lockit-sql-adapter)

SQL adapter for [Lockit](https://github.com/zeMirco/lockit).

## Installation

`npm install lockit-sql-adapter`

```js
var adapter = require('lockit-sql-adapter');
```

The adapter is built on top of [sequelize](http://sequelizejs.com/).
The following databases are supported:

 - MySQL
 - MariaDB (not yet tested but should work)
 - SQLite
 - PostgreSQL

You have to install the connector for your database of choice manually.

```
npm install pg       # for postgres
npm install mysql    # for mysql
npm install sqlite3  # for sqlite
npm install mariasql # for mariasql
```

## Configuration

The following settings are required.

```js
exports.db = 'postgres://127.0.0.1:5432/users';  // for postgres
// exports.db = 'mysql://127.0.0.1:9821/users';  // for mysql
// exports.db = 'sqlite://:memory:';             // for sqlite
exports.dbCollection = 'my_user_table';             // table name
```

## Features

### 1. Create user

`adapter.save(name, email, pass, callback)`

 - `name`: String - i.e. 'john'
 - `email`: String - i.e. 'john@email.com'
 - `pass`: String - i.e. 'password123'
 - `callback`: Function - `callback(err, user)` where `user` is the new user now in our database.

The `user` object has the following properties

 - `_id`: unique id
 - `name`: username chosen during sign up
 - `email`: email that was provided at the beginning
 - `salt`: salt generated by `crypto.randomBytes()`
 - `derived_key`: password hash generated by pbkdf2
 - `signupTimestamp`: Date object to remember when the user signed up
 - `signupToken`: unique token sent to user's email for email verification
 - `signupTokenExpires`: Date object usually 24h ahead of `signupTimestamp`
 - `failedLoginAttempts`: save failed login attempts during login process, default is `0`

```js
adapter.save('john', 'john@email.com', 'secret', function(err, user) {
  if (err) console.log(err);
  console.log(user);
  // {
  //   _id: 1,
  //   name: 'john',
  //   email: 'john@email.com',
  //   derived_key: 'c4c7a83f7b3936437798316d4c7b8c7b731a55dc',
  //   salt: 'ff449a4980a58a80c4ed80bddd34b8c9',
  //   signupToken: '13eefbe7-6bc8-43f5-b27f-0bf0ca98b8db',
  //   signupTimestamp: Fri Apr 11 2014 21:37:47 GMT+0200 (CEST),
  //   signupTokenExpires: Sat Apr 12 2014 21:37:47 GMT+0200 (CEST),
  //   failedLoginAttempts: 0,
  //   emailVerificationTimestamp: null,
  //   emailVerified: null,
  //   pwdResetToken: null,
  //   pwdResetTokenExpires: null,
  //   accountLocked: null,
  //   accountLockedUntil: null,
  //   previousLoginTime: null,
  //   previousLoginIp: null,
  //   currentLoginTime: null,
  //   currentLoginIp: null
  // }
});
```

### 2. Find user

`adapter.find(match, query, callback)`

 - `match`: String - one of the following: 'name', 'email' or 'signupToken'
 - `query`: String - corresponds to `match`, i.e. 'john@email.com'
 - `callback`:  Function - `callback(err, user)`

```js
adapter.find('name', 'john', function(err, user) {
  if (err) console.log(err);
  console.log(user);
  // {
  //   _id: 1,
  //   name: 'john',
  //   email: 'john@email.com',
  //   derived_key: '75b43d8393715cbf476ee55b12f888246d7f7015',
  //   salt: 'f39f9a5104e5ae61347dced750b63b16',
  //   signupToken: '6c93c6f8-06b6-4c6d-be58-1e89e8590d0f',
  //   signupTimestamp: Fri Apr 11 2014 21:39:28 GMT+0200 (CEST),
  //   signupTokenExpires: Sat Apr 12 2014 21:39:28 GMT+0200 (CEST),
  //   failedLoginAttempts: 0,
  //   emailVerificationTimestamp: null,
  //   emailVerified: null,
  //   pwdResetToken: null,
  //   pwdResetTokenExpires: null,
  //   accountLocked: null,
  //   accountLockedUntil: null,
  //   previousLoginTime: null,
  //   previousLoginIp: null,
  //   currentLoginTime: null,
  //   currentLoginIp: null
  // }
});
```

### 3. Update user

`adapter.update(user, callback)`

 - `user`: Object - must have `_id` key
 - `callback`: Function - `callback(err, user)` - `user` is the updated user object

```js
// get a user from db first
adapter.find('name', 'john', function(err, user) {
  if (err) console.log(err);

  // add some new properties to our existing user
  user.firstOldKey = 'and some value';
  user.secondOldKey = true;

  // save updated user to db
  adapter.update(user, function(err, user) {
    if (err) console.log(err);
    // ...
  });
});
```

### 4. Delete user

`adapter.remove(name, callback)`

 - `name`: String
 - `callback`: Function - `callback(err, res)` - `res` is `true` if everything went fine

```js
adapter.remove('john', function(err, res) {
  if (err) console.log(err);
  console.log(res);
  // true
});
```

## Test

`grunt`

## License

MIT
