# course-2016-fal-proj
Project repository for the course project in the Fall 2016 iteration of the Data Mechanics course at Boston University.

In this project, you will implement platform components that can obtain a some data sets from web services of your choice, and platform components that combine these data sets into at least two additional derived data sets. These components will interct with the backend repository by inserting and retrieving data sets as necessary. They will also satisfy a standard interface by supporting specified capabilities (such as generation of dependency information and provenance records).

**This project description will be updated as we continue work on the infrastructure.**

## MongoDB infrastructure

### Setting up

We have committed setup scripts for a MongoDB database that will set up the database and collection management functions that ensure users sharing the project data repository can read everyone's collections but can only write to their own collections. Once you have installed your MongoDB instance, you can prepare it by first starting `mongod` _without authentication_:
```
mongod --dbpath "<your_db_path>"
```
If you're setting up after previously running `setup.js`, you may want to reset (i.e., delete) the repository as follows.
```
mongo reset.js
```
Next, make sure your user directories (e.g., `alice_bob` if Alice and Bob are working together on a team) are present in the same location as the `setup.js` script, open a separate terminal window, and run the script:
```
mongo setup.js
```
Your MongoDB instance should now be ready. Stop `mongod` and restart it, enabling authentication with the `--auth` option:
```
mongod --auth --dbpath "<your_db_path>"
```

### Working on data sets with authentication

With authentication enabled, you can start `mongo` on the repository (called `repo` by default) with your user credentials:
```
mongo repo -u alice_bob -p alice_bob --authenticationDatabase "repo"
```
However, you should be unable to create new collections using `db.createCollection()` in the default `repo` database created for this project:
```
> db.createCollection("EXAMPLE");
{
  "ok" : 0,
  "errmsg" : "not authorized on repo to execute command { create: \"EXAMPLE\" }",
  "code" : 13
}
```
Instead, load the server-side functions so that you can use the customized `createTemp()` or `createPerm()` functions, which will create collections that can be read by everyone but written only by you:
```
> db.loadServerScripts();
> var EXAMPLE = createPerm("EXAMPLE");
```
Notice that this function also prefixes the user name to the name of the collection (unless the prefix is already present in the name supplied to the function).
```
> EXAMPLE
alice_bob.EXAMPLE
> db.alice_bob.EXAMPLE.insert({value:123})
WriteResult({ "nInserted" : 1 })
> db.alice_bob.EXAMPLE.find()
{ "_id" : ObjectId("56b7adef3503ebd45080bd87"), "value" : 123 }
```
For temporary collections that are only necessary during intermediate steps of of a computation, use `createTemp()`; for permanent collections that represent data that is imported or derived, use `createPerm()`.

If you do not want to run `db.loadServerScripts()` every time you open a new terminal, you can use a `.mongorc.js` file in your home directory to store any commands or calls you want issued whenever you run `mongo`.

## Other required libraries and tools

You will need the latest versions of the PROV and DML Python libraries. If you have `pip` installed, the following should install the latest versions automatically:
```
pip install prov --upgrade --no-cache-dir
pip install dml --upgrade --no-cache-dir
```
If you are having trouble with `lxml`, you could try retrieving it [here](http://www.lfd.uci.edu/~gohlke/pythonlibs/).

## Formatting the `auth.json` file

The `auth.json` file should remain empty and should not be submitted. When you are running your algorithms, you should use the file to store your credentials for any third-party data resources, APIs, services, or repositories that you use. An example of the contents you might store in your `auth.json` file is as follows:
```
{
    "services": {
        "cityofbostondataportal": {
            "service": "https://data.cityofboston.gov/",
            "username": "alice_bob@example.org",
            "token": "XxXXXXxXxXxXxxXXXXxxXxXxX",
            "key": "xxXxXXXXXXxxXXXxXXXXXXxxXxxxxXXxXxxX"
        },
        "mbtadeveloperportal": {
            "service": "http://realtime.mbta.com/",
            "username": "alice_bob",
            "token": "XxXX-XXxxXXxXxXXxXxX_x",
            "key": "XxXX-XXxxXXxXxXXxXxx_x"
        }
    }
}
```