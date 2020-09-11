# dockerized-postgres

From [here](https://blog.zentaly.com/how-to-run-postgres-in-docker/).

If you need to run Postgres database for development needs you can just install it manually on local machine. But you should be aware that this procedure will require some level of knowledge about Postgres installation and maintaining procedures.

On the other hand there is much more simple way - run Postgres database inside Docker container. Docker effectively incapsulates deployment, administration and configuration procedures. So if you want to deploy Postgres locally with minimum efforts Docker is the best choice. All you need to do is just start pre-build Docker container and you will have Postgres database ready for your service.

Here is my github repo to build Docker container with embedded Postgres database: https://github.com/alexdik/dockerized-postgres.

If you don't have Docker yet, you can download and install it from official site: https://docs.docker.com/install/

To build Docker container you will need to run 3 simple commands in terminal:

git clone https://github.com/alexd84/dockerized-postgres.git
docker build -t postgres dockerized-postgres
docker run -di -p 5432:5432 postgres
123

Here you are cloning project from github, building container and launching it.

So now you can verify Postgres database instance running on your local machine:

telnet 127.0.0.1 5432
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
1234

To authorise into Postgres database following default credentials should be used: postgres/postgres.
How is it working? (for those who actually care)

Repository you just clonned contains 3 files:

    Dockerfile
    entrypoint.sh
    pg_hba.conf

Dockerfile

This is main file which instructs Docker to create new container based on Ubuntu image, download Postgres distributive, configure and proceed to entrypoint.sh script:

FROM ubuntu:14.04

RUN apt-get update -y
RUN apt-get install -y wget

RUN sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main 9.5" >> /etc/apt/sources.list.d/pgdg.list'
RUN wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

RUN apt-get update -y
RUN apt-get install postgresql-9.5 postgresql-contrib-9.5 -y

RUN mv /etc/postgresql/9.5/main/pg_hba.conf /etc/postgresql/9.5/main/pg_hba.conf.backup
COPY pg_hba.conf /etc/postgresql/9.5/main/pg_hba.conf
RUN echo "listen_addresses = '*'" >> /etc/postgresql/9.5/main/postgresql.conf

EXPOSE 5432

COPY entrypoint.sh /
ENTRYPOINT sudo /entrypoint.sh
12345678910111213141516171819

entrypoint.sh

Here we start Postgres database and setting default login and password for access:

sudo service postgresql start
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'postgres'"
tail
123

The last tail command is needed to assure that entrypoint.sh script never completes and it effectively makes container to run in background, otherwise it will stop execution immediately after startup.
pg_hba.conf

local   all             postgres                                peer
host    all             all             127.0.0.1/32            md5
host    all             all             0.0.0.0/0               md5

1234

This file is exclusively used to configure Postgres to accept connections from remote hosts. By default Postgres permits external connections only from localhost, so when running in Docker you cant access it from your host machine. Thus we add pg_hba.conf configuration file to permit all inbound connections from any machines.
Summary

So now you have got Postgres database running locally without deep knowledge about it's internals. What is also useful to know that there are pre-build Docker containers for almost all kind of applications you can imagine like email server or Hadoop cluster. And so you can effectively rely on them to start complex applications in minutes and save time by avoiding manual installation and configuration.
