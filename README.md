This is an opinionated way to build APIs and API-based web applications with a relational database as the primary data store.

### Goals

- Provide a production-quality template you can follow to build a web application
  - Health checks, error handling, logging, performance, security, etc.
- Secure by default, with support for advanced security features
  - Centralized authentication
  - Centralized coordination of active defense
  - Runtime checks and reporting of library/platform vulnerabilities
  - Fully leverage the Java security sandbox
  - Pre-configure static source code analysis tools
- Easy to set up and develop with
  - Embedded HSQLDB database
  - Embedded fake authentication and authorization
  - Easy to Dockerize and deploy/manage
- Keep as much of the infrastructure as possible in a library ([vertx-base](https://github.com/susom/vertx-base)) or Maven parent pom ([vertx-parent](https://github.com/susom/vertx-parent)) to simplify maintenance and security updates

## Getting Started

#### What You Need

[Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
and [Maven 3](https://maven.apache.org/).

Some of the libraries referenced by this project are currently private to
IRT, so you will need to have Maven configured to use our internal repository
(see https://medwiki.stanford.edu/display/kbase/Maven).

#### Run Locally

Recommended steps for development:

- Clone the repository
- Drag the directory to IntelliJ
- Run Main.java in the debugger

The above steps should allow you to modify the HTML on the fly, and
even some of the server code (the IntelliJ debugger reloads code on
the fly for at least some cases).

To adjust your configuration, copy the sample properties file rather
than modifying it, so you won't accidentally commit your local changes.
The default configuration uses an embedded HSQLDB database. You will
need to adjust your local properties if you use a different database.

```
cp sample.properties local.properties
```

#### Maven Instructions

Build everything, create the database schema, and start the server.

```
mvn -DskipTests clean package
java -jar target/vertx-*-SNAPSHOT.jar create-database
java -jar target/vertx-*-SNAPSHOT.jar run
```

If you want to wipe out the embedded database, just delete the data files.

```
rm -rf ./hsql
```

You can combine the above to quickly reset and restart everything.

```
rm -rf .hsql ; mvn -DskipTests clean package ; java -jar target/vertx-*-SNAPSHOT.jar create-database run
```

Before you commit your changes, run the static analysis checks. This will
make sure code is formatted correctly and doesn't contain certain kinds of
errors and security vulnerabilities. These are divided into two sets, only
because certain of the tools are mutually incompatible.

```
mvn -DskipTests -Dcheck1 clean verify
mvn -DskipTests -Dcheck2 clean verify
```

#### Run Docker Locally

To do this you should first install Docker for Mac (or Docker for Windows as the case may be).

Build the Maven artifacts and use it to create a docker image.

```
mvn -DskipTests -Ppostgres,release clean package
unzip target/deploy.zip -d target/deploy
docker build --pull -t app target/deploy
```

Spin up the PostgreSQL database.

```
docker volume create --name postgres-data
docker run -d --name postgres \
       -v postgres-data:/var/lib/postgresql/data \
       -e "POSTGRES_PASSWORD=secret" postgres
```

Spin up the application, linking to the database container.

```
docker volume create --name app-conf
docker volume create --name app-logs
docker run -it -v app-conf:/app/conf \
       --name app app vi conf/app.properties
  # Add this property to make it use fake authentication
  insecure.fake.security=yes
docker rm app
docker run -d -p 8080:8080/tcp -v app-conf:/app/conf -v app-logs:/app/logs \
       --link postgres:postgres --name app app
```

If you want to watch the logs:

```
docker exec -it app tail -f logs/app.log
```

Now open the application in your browser.

[http://localhost:8080/secure-app](http://localhost:8080/secure-app)
