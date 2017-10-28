# Rabble Rouser

Rabble Rouser consists of many small components, and the code is spread across lots of [repositories](https://github.com/rabblerouser).
Rather than making you clone them all and install their dependencies individually before you can even get started, this
repository contains a [Cage](http://cage.faraday.io) project to help make things easier. Cage allows you to start by
running everything from pre-built Docker images, and then check out the source code for only the components you want to
make changes to.

It is our hope that this will make it fairly painless to get started with Rabble Rouser. If there is any way we can help
make it a better experience for you, please let us know by opening an issue on this repository, or talking to one of the
team members.

## Dependencies

1. [Docker](https://store.docker.com/search?type=edition&offering=community)
2. docker-compose: For most systems this is installed automatically with Docker.
3. [Cage](http://cage.faraday.io/setup) (Windows users [see here](https://github.com/faradayio/cage/blob/master/WINDOWS.md))

## Run the application

From this repository, just run this one command!

```sh
cage up --init
```

This will start up all the microservices and dependencies, and seed initial data. Note that you only need the `--init`
flag on the first run, unless you manually clean up the data directories.

## Try it out!

1. Open up your browser and head to http://localhost:8080
2. Register a new member - make sure you see a success message
3. Head to http://localhost:8080/dashboard and log in - you should see your newly registered member
    - Email: `superadmin@rabblerouser.team`
    - Password: `password1234`

## Making your first code change

Let's make a small change to the UI and see the result.

### Check out some source code and mount it

```sh
#1. Clone the source code for the core app, and tell Cage to 'mount' it
cage source mount core
#2. Install all the library dependencies of this app
cage run core npm install
#3. Restart the container in source mode, rather than just as a pre-built image
cage up
```

### Make a change

You can make any visible change you like here. Perhaps open up the file `./src/core/frontend/src/signup/components/DetailsForm.js`
and change one of the field labels to something different. Once you save the file, the frontend will automatically
rebuild itself, the page will refresh, and you should see your changes!

Note: Any other services you mount will be cloned into `./src/<app-name>/`.

## Putting things back

If you're no longer working on a service, you might want to unmount the source code and set it back to pre-built Docker
image mode. It's probably also a good idea to pull the latest image:

```sh
cage source unmount core
cage pull core
```

## Running tests
The best way to run the tests of a particular project is to shell into the running container and run the tests from
inside. For example:

```sh
cage source mount core
cage shell core
# You'll now have a new shell, inside the docker container, with the source code mounted from your host machine
cd frontend
npm test
```

This can be used to run any other kind of development and debugging tasks, while giving each application a totally
isolated development environment.

## Full email simulation
For a fairly thorough end-to-end test of the system, after bringing up all the services, try running this:

```sh
cage run group-mail-receiver email-from-john
```

The group-mail-receiver lambda is integrated here as a "task pod" which means that rather than running in the background,
it's a task that we can run manually whenever we need it. In this case, we're running a lambda which triggers a series
of events ultimately leading to emails being sent (local only, no *actual* emails get sent here). If everything worked,
you should be able to look inside the `sent-email` directory and see that an email was just sent successfully.

## Other useful commands

These cage commands may come in handy while developing:

- `cage status`: see the status of all services in the project
- `cage logs -f <service-or-pod-name>`: tail the logs of the given application or pod
- `cage`: Run cage with no arguments to see everything it can do!

## Enabling shell completion for cage

Cage can autogenerate a script that enables auto-completion of its own commands. Run this for instructions:

```sh
cage generate completion --help
```

## Using the local AWS mocks

You can use the local AWS mocks similar to the real AWS services, as long as you remember to set the endpoint. For
example, to list the buckets that exist locally:

```sh
aws --endpoint-url='http://localhost:4569' s3api list-buckets
```

For convenience, some of the AWS mocks expose some data to your host machine:

 - `s3-data/`: the data store of the local S3 mock. Useful for tweaking bucket contents during development.
 - `kinesis-data`: the data store of the local kinesis mock.
 - `sent-mail`: the resulting output of successful requests to the local SES mock. Useful for verification purposes.
