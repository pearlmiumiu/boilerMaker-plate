##1. npm install --save -dev your DEV dependencies
-----npm install --save -dev webpack babel -core babel -loader babel-preset-react
##2. npm install --save your regular dependencies
-----npm install --save react react-dom react-router-dom
##3. decide on an 'entry' file and an 'output' fle for your webpack pipeline
-----your entry file might be sth simple like an index.js, app/main.jsx, client/app.jsx or brower/index.jsx
-----your output file will be created by webpack. you dont need to actually create it yet. just decide where you want it to live. this could be in the root of your app or a public folder
##4. in your package.json, set up an npm start command to build your client javascript and run your server
-----here is an example where we run webpack in --watch mode in the background and simultaneousely start a server with nodemond in server.js
'start': 'node server',
'start-dev': webpack -w & nodemon server.js
##5. npm install --save redux react-redux redux-thunk redux-logger
##6. css
npm install --save-dev style-loader css-loader
and then add to webpack.config file
##7. project file guide line
	=== my-project
	--- client/
	----index.js   ---> entry point for client javascript
	-- node_modules/
	-- public/
	-- server
	----index.js ---> entry point for server javascript
	---.gitignore
	--package.json
#back-end server
##8. npm install --save express
	=========create an app with express==========
	const express= require('express')
	const app=express()
##9. having server logs helps with debuggig even in prouction enviorments
	install and hook up a logger like morgan, express-logger
	npm install --save morgan
	const morgan =require('morgan')
	app.use(morgan('dev'))
##10. static middle ware
	 once  your browser gets your index.html, it often needs to request static assets from your server -these include javascript files, css files, and images. many developers organize this content by putting it into a public folder.
	 app.use(express.static(path.join(__dirname,'.static assets path')))
##11. parsing middleware to parse request's body. 
	Be sure to npm install --save body-parser.

	const bodyParser = require('body-parser');
	app.use(bodyParser.json());
	app.use(bodyParser.urlencoded({ extended: true }));
##12. api routes
###Your API is the main course of your server. It's often preferable to break up your different routes using the router object. By convention, API routes are prefixed with /api/ - this is purely done to namespace them away from your "front-end routes" (such as those created by react-router).
Assume we have a file structure like this:

	/my-project
	--/apiRoutes
	----kittens.js
	----index.js
	----puppies.js
	----users.js
	--server.js
	###From your main app pipeline, you might mount all of your API routes on /api like so:
    ##app.use('/api', require('./apiRoutes')); // matches all requests to /api
    Then, in apiRoutes/index.js, you might further delegate each router into its own namespace like so:

	// apiRoutes/index.js
	const router = require('express').Router();

	router.use('/users', require('./users')); // matches all requests to /api/users/
	router.use('/puppies', require('./puppies')); // matches all requests to  /api/puppies/
	router.use('/kittens', require('./kittens')); // matches all requests to  /api/kittens/

	module.exports = router;
	###Now, in each individual router, each route will automatically match on /api/routeName/, so you can write your routes in the following fashion:

	// apiRoutes/puppies.js
	const router = require('express').Router();

	// matches GET requests to /api/puppies/
	router.get('/', function (req, res, next) { /* etc */});
	// matches POST requests to /api/puppies/
	router.post('/', function (req, res, next) { /* etc */});
	// matches PUT requests to /api/puppies/:puppyId
	router.put('/:puppyId', function (req, res, next) { /* etc */});
	// matches DELTE requests to /api/puppies/:puppyId
	router.delete('/:puppyId', function (req, res, next) { /* etc */});

	module.exports = router;
##13. handle error middleware
###What if a user requests an API route that doesn't exist? For example, if we're serving up puppies, kittens and users, what if a user asks for /api/sloths?

Give 'em the 'ol 404!

	Using our apiRoutes/index.js example from before:

	// apiRoutes/index.js
	const router = require('express').Router();

	router.use('/users', require('./users')); // Users? Check.
	router.use('/puppies', require('./puppies')); // Puppies? Check. 
	router.use('/kittens', require('./kittens')); // Kittens? Check.

	// Sloths?!?! Get outta town!
	router.use(function (req, res, next) {
	  const err = new Error('Not found.');
	  err.status = 404;
	  next(err);
	});
##14. send index html
###Because we generally want to build single-page applications (or SPAs), our server should send its index.html for any requests that don't match one of our API routes.
	app.get('*', function (req, res) {
	  res.sendFile(path.join(__dirname, './path/to/index.html');
	});
##15. handle 500 errors
###Make sure this is at the very end of your server entry file!
	app.use(function (err, req, res, next) {
	  console.error(err);
	  console.error(err.stack);
	  res.status(err.status || 500).send(err.message || 'Internal server error.');
	});
##16. start the server, listen to the port
	const port = process.env.PORT || 3000; // this can be very useful if you deploy to Heroku!
	app.listen(port, function () {
	  console.log("Knock, knock");
	  console.log("Who's there?");
	  console.log(`Your server, listening on port ${port}`);
	});
#back-end sequelize 
##17. create database
	Either use the createDb command (which comes with some installations of postgres), or psql/Postico/etc to create your database.
##create sequelize instance
	const Sequelize = require('sequelize');

	const db = new Sequelize('postgres://localhost:5432/yourdbname', {
	  logging: false // unless you like the logs
	  // ...and there are many other options you may want to play with
	});

	module.exports = db;
##sync your database
###It's not strictly required for your database to sync before your server starts, but it is considered a best practice.

###Remember that if you pass the force: true option to sync, that will drop all of your tables before re-creating them. Be sure to never do this in production!
	// say our sequelize instance is create in 'db.js'
	const db = require('./db.js'); 
	// and our server that we already created and used as the previous entry point is 'server.js'
	const app = require('./server');
	const port = process.env.PORT || 3000;

	db.sync()  // sync our database
	  .then(function(){
	    app.listen(port) // then start listening with our express server once we have synced
	  })
###session middleware
	const session = require('express-session');

	app.use(session({
	  secret: 'a wildly insecure secret',
	  resave: false,
	  saveUninitialized: false
	}));
###protect session secrete
	app.use(session({
	  secret: process.env.SESSION_SECRET || 'a wildly insecure secret',
	  resave: false,
	  saveUninitialized: false
	}));
###session store
	we're storing our session information in memory, which means it will only live for the life of our server. This is fine for testing, but a production app could be making new deploys several times a day, so this could be disruptive.

	Instead, look up connect-session-sequelize as a more resilient option - session information will be stored in our postgres database instead, so we can re-deploy/re-start our server without worrying about interrupting any currently logged-in users.

	Create a session store using connect-session-sequelize and hook up our new store into our session middleware.	
	// we will need our sequelize instance from somewhere
	const db = require('./db');
	// we should do this in the same place we've set up express-session
	const session = require('express-session');

	// configure and create our database store
	const SequelizeStore = require('connect-session-sequelize')(session.Store);
	const dbStore = new SequelizeStore({ db: db });

	// sync so that our session table gets created
	dbStore.sync();

	// plug the store into our session middleware
	app.use(session({
	  secret: process.env.SESSION_SECRET || 'a wildly insecure secret',
	  store: dbStore,
	  resave: false,
	  saveUninitialized: false
	}));

##initialize passport
###We need to initialize passport so that it will consume our req.session object, and attach the user to the request object.

Make sure to put this AFTER our session middleware!
	const passport = require('passport');

	app.use(passport.initialize());
	app.use(passport.session());
##serialize/deserialize user
	###Remember that serialization is usually only done once per session (after we invoke req.login, so that passport knows how to remember the user in our session store. Generally, we use the user's id.

	###Deserialization runs with every subsequent request that contains a serialized user on the session - passport gets the key that we used to serialize the user, and uses this to re-obtain the user from our database.
	//example
	passport.serializeUser((user, done) => {
	  try {
	    done(null, user.id);
	  } catch (err) {
	    done(err);
	  }
	});

	passport.deserializeUser((id, done) => {
	  User.findById(id)
	    .then(user => done(null, user))
	    .catch(done);
	});

##Encrypt passwords
	###Storing plain-text passwords in our database is incredibly insecure. If our users are going to trust us, we need to encrypt their passwords.

	###Research the concept of salting and hashing passwords. You'll want to take advantage of either node's built-in crypto module or a library like bcryptjs.

	The following is an example of a very robust user model that contains methods and hooks for encrypting and authenticating passwords. It also contains a helpful sanitize method you can use to make sure you don't send any more information than needed down to the client.

	It assumes you have your sequelize instance stored in a file called db.js in the same directory as the model's js file. It also avails itself of the lodash npm package - you'll need to npm install --save lodash in order to use it.

	const crypto = require('crypto');
	const _ = require('lodash');
	const Sequelize = require('sequelize');

	const db = require('./db');

	const User = db.define('user', {
	  email: {
	    type: Sequelize.STRING,
	    unique: true,
	    allowNull: false
	  },
	  password: {
	    type: Sequelize.STRING
	  },
	  salt: {
	    type: Sequelize.STRING
	  }
	}, {
	  hooks: {
	    beforeCreate: setSaltAndPassword,
	    beforeUpdate: setSaltAndPassword
	  }
	});
	// instance methods
	User.prototype.correctPassword = function (candidatePassword) {
	  return this.Model.encryptPassword(candidatePassword, this.salt) === this.password;
	};
	User.prototype.sanitize = function () {
	  return _.omit(this.toJSON(), ['password', 'salt']);
	};
	// class methods
	User.generateSalt = function () {
	  return crypto.randomBytes(16).toString('base64');
	};
	User.encryptPassword = function (plainText, salt) {
	  const hash = crypto.createHash('sha1');
	  hash.update(plainText);
	  hash.update(salt);
	  return hash.digest('hex');
	};
	function setSaltAndPassword (user) {
	  // we need to salt and hash again when the user enters their password for the first time
	  // and do it again whenever they change it
	  if (user.changed('password')) {
	    user.salt = User.generateSalt()
	    user.password = User.encryptPassword(user.password, user.salt)
	  }
	}

	module.exports = User;
##login
	Here is an example that assumes a req.body that contains identifying key-value pairs (like an email field), and that users have some instance method to evaluate the password on req.body.

	router.post('/login', (req, res, next) => {
	  User.findOne({
	    where: {
	      email: req.body.email
	    }
	  })
	    .then(user => {
	      if (!user) res.status(401).send('User not found');
	      else if (!user.hasMatchingPassword(req.body.password) res.status(401).send('Incorrect password');
	      else {
	        req.login(user, err => {
	          if (err) next(err);
	          else res.json(user);
	        });
	      }
	    })
	    .catch(next);
	});
##sign up
	router.post('/signup', (req, res, next) => {
	  User.create(req.body)
	    .then(user => {
	      req.login(user, err => {
	        if (err) next(err);
	        else res.json(user);
	      });
	    })
	    .catch(next);
	});
##log out
	router.post('/logout', (req, res, next) =&gt; {
	  req.logout();
	  res.sendStatus(200);
	});
##get me
	We should also write a method that our app can use to fetch the logged-in user on our session. Our client will make this request every time the client application loads - this allows us to keep the user logged in on the client even after they refresh.
	router.get('/me', (req, res, next) =&gt; {
	  res.json(req.user);
	});
##protect credential

	example
	// localSecrets.js
	process.env.GOOGLE_CLIENT_ID = 'etc';
	process.env.GOOGLE_CLIENT_SECRET = 'etc';
	// in your app's entry point
	require('./localSecrets'); // mutate the process.env object with your variables
	require('./mainApp')       // run your app after you're sure the env variables are set.
	// main.js

	// this means that we need to make sure our local NODE_ENV variable is in fact set to 'development'
	// Node may have actually done this for you when you installed it! If not though, be sure to do that.
	if (process.env.NODE_ENV === 'development') {
	  require('./localSecrets'); // this will mutate the process.env object with your secrets.
	}

	require('./mainApp');
## add google id to user model
	db.define('user', {
	  // etc...
	  google_id: {
	    type: Sequelize.STRING
	  }
	  // etc...
	});
##redirect when user petitions to use oauth
###On our front-end, we'll have some kind of "Log In With Google" button. When the click that, it should make a GET request to our server (something like "/auth/google").

###Handle requests at this route by redirecting to the Provider (in this case, Google).
router.get('/auth/google', passport.authenticate('google', { scope: 'email' }));
##configure callback
	Passport takes care of the hard stuff again. This assumes that the callback you registered is "/auth/google/callback".

	router.get('/auth/google/callback', passport.authenticate('google', {
	  successRedirect: '/',
	  failureRedirect: '/login'
	}));
##configure strategy
	When the Provider (Google) passed the user back to us via our callback url, it gave us a temporary token. Now, we need to go present this token to Google, along with our Google client secret. Passport takes care of this part (via passport.authenticate). If the temporary token we present is good, the Provider will send us back the user's profile and a more permanent access token (which we can store on our user model). We need to take that access token and profile information and turn it into a user in our database. Passport can't do this part for us, so we need to write a function that will.

	We also need to give passport our callback URL and client ID so that it can perform the previous two tasks appropriately.

	To do all this, we need to write what passport calls a "Strategy", and instruct passport to use this strategy.
	const GoogleStrategy = require('passport-google-oauth').OAuth2Strategy;

	// collect our google configuration into an object
	const googleConfig = {
	  clientID: process.env.GOOGLE_CLIENT_ID,
	  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
	  callbackURL: '/auth/google/callback'
	};

	// configure the strategy with our config object, and write the function that passport will invoke after google sends
	// us the user's profile and access token
	const strategy = new GoogleStrategy(googleConfig, function (token, refreshToken, profile, done) {
	  const googleId = profile.id;
	  const name = profile.displayName;
	  const email = profile.emails[0].value;

	  User.findOne({where: { googleId: googleId  }})
	    .then(function (user) {
	      if (!user) {
	        return User.create({ name, email, googleId })
	          .then(function (user) {
	            done(null, user);
	          });
	      } else {
	        done(null, user);
	      }
	    })
	    .catch(done);
	});

	// register our strategy with passport
	passport.use(strategy);


