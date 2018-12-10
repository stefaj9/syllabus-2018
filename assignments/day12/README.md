# Day 12

## Database Migration

In this step we will add database migration to our project, keep in mind that since
this project is academic we are destroying and creating an entirely new database each
time we deploy the game. Normally the database would be kept between and that is when
database migrations are useful.

Database migrations should be bi-directional, that is we should be able to migrate
the database from a lower => higher version and back from a higher => lower version.

We will be using the `db-migrate` npm package to create and run our migration scripts.
The package should normally be installed as a dev dependency but because of how we are
recreating the database using docker-compose, we will install it ad a production 
dependency for simplicity.

Create file `database.json`:
```json
{
  "pg": {
    "driver": "pg",
    "host": "my_database_container",
    "database": "game_database",
    "user": "my_user",
    "password": "my_password",
    "schema": "postgres",
    "port": 5432,
    "max": 10,
    "idleTimeoutMillis": 30000
  }
}
```

Now run:
```bash
node node_modules/db-migrate/bin/db-migrate --env pg create GameResultTable
```
To create an empty migration script.

It should look like this:\
`20181210110458-GameResultTable.js`:
```javascript
'use strict';

var dbm;
var type;
var seed;

/**
  * We receive the dbmigrate dependency from dbmigrate initially.
  * This enables us to not have to rely on NODE_PATH.
  */
exports.setup = function(options, seedLink) {
  dbm = options.dbmigrate;
  type = dbm.dataType;
  seed = seedLink;
};

exports.up = function(db) {
  return null;
};

exports.down = function(db) {
  return null;
};

exports._meta = {
  "version": 1
};
```

The only functions we care about are `up` and `down`, since this is our first migration
script up would be moving from an empty database to a database with a GameResult table
and down would be moving back (removing the GameResult table).

Since this is an academic exercise we will be intentionally leaving out the `InsertDate`
column that is being used in our API, so that we can create another migration script
that modifies our GameResult table by adding it.

Here is the code for the `GameResultTable` migration:
```javascript
'use strict';

var dbm;
var type;
var seed;

/**
  * We receive the dbmigrate dependency from dbmigrate initially.
  * This enables us to not have to rely on NODE_PATH.
  */
exports.setup = function(options, seedLink) {
  dbm = options.dbmigrate;
  type = dbm.dataType;
  seed = seedLink;
};

exports.up = function(db) {
  return db.createTable('GameResult', {
    ID: { type: 'int', primaryKey: true, autoIncrement: true },
    Won: { type: 'boolean', notNull: true },
    Score: { type: 'int', notNull: true },
    Total: { type: 'int', notNull: true }
  });  
};

exports.down = function(db) {
  return db.dropDatabase('GameResult');
};

exports._meta = {
  "version": 1
};
```

Now remove the initialization part from `database.js` since we will be using `db-migrate`
to create our table.

Add npm script:\
`"migratedb:pg": "db-migrate --verbose --env pg --config ./database.json --migrations-dir ./migrations up",`

Now make sure to copy the migration files to the docker container and change CMD:
```dockerfile
# Give postgres time to setup before we try to migrate.
CMD sleep 5 && npm run migratedb:pg && node app.js
```

Now install the postgres driver npm package for db-migrate, `db-migrate-pg`:\
`npm install db-migrate-pg --save`

Our `pg` package converts our query names (database, table, columns) to lowercase, this
did not cause any problems when we were creating and querying the database with `pg`, but
now that we are using `pg` and `db-migrate` we will run into issues since Postgres is
case sensitive and `db-migrate` will create the table with upper-case characters in both
the table name and column names, but `pg` will convert our queries to use all lower-case.
To avoid this you should add quotes around the names in your queries:\
`INSERT INTO "GameResult" ("Won", "Score", "Total", "InsertDate") VALUES($1, $2, $3, CURRENT_TIMESTAMP);`

Now run API using `docker-compose` locally, you should see postgres complain:\
`ERROR:  column "InsertDate" of relation "GameResult" does not exist`

Now create another migration script that adds the `InsertDate` column to the `GameResult`
table.

Your game should now be playable through the API without getting any errors.

There is a bug in the API, when the user starts the game with 21, the game does not get
stored in the database, you will need to fix it.

You should also be able to call `/stats` on your API (you might have to change it in `server.js`
from `post` to `get`), and receive accurate statistics.

To verify:
- Start the API locally
- Configure the capacity test to play 10.000 games
- Run the capacity test against your local API
- Call the `/stats` method
- You should get roughly: \
`{"totalNumberOfGames":"10000","totalNumberOfWins":"4162","totalNumberOf21":"919"}`

Playing 10.000 games against a local instance took about 60 seconds, so if it takes
longer than 5 minutes there is something wrong.

## Connect UI

Has not been finalized yet, check back later.
