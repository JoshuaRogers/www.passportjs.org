# Verify Password

Now that we are prompting the user for their username and password, the next
step is to verify the password.

To do that, we are going to use Passport and the `passport-local` strategy.
Install both as dependencies:

```sh
$ npm install passport
$ npm install passport-local
```

Next, we need to configure Passport.  Open `'routes/auth.js'` and `require` the
newly installed packages at line 2, below where `express` is `require`'d:

```js
var express = require('express');
var passport = require('passport');
var LocalStrategy = require('passport-local');
var crypto = require('crypto');
var db = require('../db');
```

The built-in crypto module and the app's database are also `require`'d.

Add the following code at line 7 to configure the `LocalStrategy`.

```js
passport.use(new LocalStrategy(function verify(username, password, cb) {
  db.get('SELECT * FROM users WHERE username = ?', [ username ], function(err, row) {
    if (err) { return cb(err); }
    if (!row) { return cb(null, false, { message: 'Incorrect username or password.' }); }
    
    crypto.pbkdf2(password, row.salt, 310000, 32, 'sha256', function(err, hashedPassword) {
      if (err) { return cb(err); }
      if (!crypto.timingSafeEqual(row.hashed_password, hashedPassword)) {
        return cb(null, false, { message: 'Incorrect username or password.' });
      }
      return cb(null, row);
    });
  });
}));
```

This configures the `LocalStrategy` to fetch the user record from the app's
database and verify the hashed password that is stored with the record.  If
that succeeds, the password is valid and the user is authenticated.

Next, we need to add a route that will process the form submission when the user
clicks "Sign in."

Continuing within `'routes/auth.js'`, add this route at line 28, below the
`'/login'` route:

```js
router.post('/login/password', passport.authenticate('local', {
  successRedirect: '/',
  failureRedirect: '/login'
}));
```

We've now got a route that will process the form on the login page!  Let's test
it out to see what happens.

Start the server:

```sh
$ npm start
```

Open [http://localhost:3000](http://localhost:3000), click "Sign in," and enter
the following credentials:

```
Username: alice
Password: letmein
```

Click "Sign in."

Uh oh... we are informed that there's an error related to sessions.  Next, we
will fix that by [establishing a session](../session/).
