# GnosisDB

Ethereum blockchain providers expose a common interface for querying, but to query blockchain providers directly for tournament-related info would be much too slow. That's why we've developed [GnosisDB](https://github.com/gnosis/gnosisdb), which syncs up with and queries a blockchain provider directly, filtering related data into an indexed database which provides great boosts in speed when querying the database.

Let's follow along with the GnosisDB setup instructions. First, ensure that [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) are installed.

Then, make sure you're not inside of a repository directory, clone GnosisDB, and `cd` into the newly cloned repo:

```sh
cd .. # assuming you're still in the olympia-token directory you created previously
git clone git@github.com:gnosis/gnosisdb.git
cd gnosisdb
```

Then, have Docker build the container images.

```sh
docker-compose build --force-rm
```

You may see the following error:

```text
$ docker-compose build --force-rm
Building ipfs
ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?

If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.
```

If so, it may be that your current user does not have permission to connect to the Docker daemon. If that's the case, consider [this answer](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo#477554), or run `docker-compose` with `sudo`.

If the build succeeds, we next want to configure the web container instance by adding a superuser so that we have admin access to the database:

```sh
docker-compose run web bash
```

This makes us the root user of a shell inside of the web container:

```text
root@e7f35cc2dc18:/gnosisdb# 
```

Using that shell, we migrate the database schema and create a superuser:

```sh
python manage.py migrate
python manage.py createsuperuser
```

Now, `exit` the web container shell and spin up all the GnosisDB Docker images:

```sh
docker-compose up
```
