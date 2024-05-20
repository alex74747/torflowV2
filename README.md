# THIS IS NOT A SERIOUS FORK (FOR NOW)
# To be done 
Fix npm packages security (upgrade package version)
# TorFlow

> Data flow in the Tor network

![Image of TorFlow](https://github.com/alex74747/torflowV2/raw/master/public/sample.png)

## Building

Requires [Docker](https://docker.io), [node](http://nodejs.org/), [bower](http://bower.io/), and [gulp](http://http://gulpjs.com/).

Install server-side modules:

```bash
npm install
```

Install client-side modules:

```bash
cd public
bower install
gulp install
```

Up the database container:

```bash
cd containers/
docker-compose up -d
```

Edit config.js to point to your MySQL database if you're running without the local containerized database.

## Ingesting Data

Ingest data into MySQL via the bin/ingest node script. There is a set of sample data in the ./data/sample folder. To import this into your database:

```bash
node bin/ingest data/sample
```

## Running

Start the server:

```bash
npm start
```

The application will be available in your browser at http://localhost:3000/

## Building the Docker Containers

### Prepare the Build Directories

```bash
gulp build
```

### Application Server Container

The "torflow" app container will run the application, and connect to an external MySQL database. The "config.js" configuration file build into the container will specify the connection parameters.

Build the app container:

```bash
cd /deploy/app
docker build -t docker.uncharted.software/torflow .
```

Run the app container:

```bash
docker run -ti --rm --name torflow -v /logs/:/var/log/supervisor/ -p 3000:3000 docker.uncharted.software/torflow
```

Or up it in the test docker-compose stack:
```bash
cd containers/
docker-compose -f docker-compose-test.yaml up
```

If your container config.js points at a MySQL server that can't be resolved, you can add a hosts entry at run-time using the Docker parameter `--add-host`.

### Ingest Container

The "torflow-ingest" container will run the ingest program described above, ingesting whatever data is mounted at the command-line below.  It will use the "config.js" configuration file built into the container.

Build the ingest container:

```bash
cd /deploy/ingest
docker build -t docker.uncharted.software/torflow-ingest .
```

Run the ingest container:

```bash
docker run -ti --rm --name torflow-ingest -v /torflow/data/sample/:/torflow/data docker.uncharted.software/torflow-ingest
```

This assumes you are importing the sample data in the /torflow/data/sample folder. If your container config.js points at a MySQL server that can't be resolved, you can add a hosts entry at run-time using the Docker parameter `--add-host`.

### Demo Container

The demo container is pre-configured to run against the demo MySQL database, and will automatically ingest the the sample data from the /torflow/data/sample folder.  The "config.js" for the demo app, and the "mysql.properties" for the MySQL server, are already configured to match each other. If you change one, you need to update the other.

Run the MySQL container:

```bash
docker run -ti --rm --name torflow-mysql -p 3306:3306 --env-file mysql.properties mysql:5.7
```

Build the demo container:

```bash
cd /deploy/demo
docker build -t docker.uncharted.software/torflow-demo .
```

Run the demo container:

```bash
docker run -ti --rm --name torflow --link torflow-mysql:MYSQL -v /logs/:/var/log/supervisor/ -p 3000:3000 docker.uncharted.software/torflow-demo
```
