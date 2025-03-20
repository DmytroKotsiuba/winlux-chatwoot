# Guide to Chatwoot's local installation and setup

## Prerequisites

Before you start, **be sure you have installed**:

* Redis [[link to download](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/)]
* Ruby [[link to download](https://www.ruby-lang.org/en/downloads/)] **NOTE: !!!Install ruby exactly 3.3.3 version!!!**
* Docker [[link to download](https://www.docker.com/products/docker-desktop/)]
* Docker-compose [[link to download](https://docs.docker.com/compose/install/)]
* Node.js not older that 20 version [[link to download](https://nodejs.org/en/download/)]

## Installation

### 1. Fork the repository

Fork the repository to your GitHub account. You can do it by clicking the `Fork` button on the top right corner of the repository page.

**NOTE**: Because the Chatwoot's stable code is regarded on branch `3.x` you need to uncheck the "Copy the develop branch only" option.

[![Screenshot-2025-03-18-at-11-58-22.png](https://i.postimg.cc/SQPqyfgs/Screenshot-2025-03-18-at-11-58-22.png)](https://postimg.cc/HjbFzXDD)

After you fork the repository, you will be redirected to the forked repository page. Unfortunately, forking the repository also fetches *all* the branches from the original repository and there a lot of them. You need to delete all the branches except the `3.x`, `master` and `develop` branches.

#### How to delete all the branches except the `3.x`, `master` and `develop` branches

To automate this process, you can use the following script:

```bash
git checkout master && git branch -r | grep -v 'origin/master$' | grep -v 'origin/develop$' | grep -v 'origin/3.x$' | grep -v 'origin/HEAD' | sed 's/origin\///' | xargs -I {} git push origin --delete {}
```

This script will checkout the `master` branch and then delete all the branches except the `master`, `develop` and `3.x` branches.

Do not forget to prune the remote branches:

```bash
git remote prune origin
```

You should now see only the `3.x`, `master` and `develop` branches in the repository.

### 2. Clone the repository

Clone the repository to your local machine.

Via HTTPS:
```bash
git clone https://github.com/DmytroKotsiuba/chatwoot.git
```

Via SSH:

```bash
git clone git@github.com:DmytroKotsiuba/chatwoot.git
```

You should now see the `chatwoot` directory in your local machine.

### 3. Install dependencies

Navigate to the `chatwoot` directory and install the dependencies:

```bash
cd chatwoot
make burn
```

### 4. Setup the environment variables

Navigate to the `chatwoot` directory and create a `.env` file:

```bash
cd chatwoot
cp .env.example .env
```

The changes you need to make are:

```
POSTGRES_DATABASE=chatwoot
POSTGRES_HOST=localhost
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=postgres

REDIS_URL=redis://redis:6379/0

or

REDIS_URL=redis://localhost:6379

RAILS_ENV=development
```

### 5. Setup the docker-compose.yml file

Navigate to the `chatwoot` directory and open the `docker-compose.yml` file.

```bash
cd chatwoot

# open the docker-compose.yml file

nano docker-compose.yml
```

You need to change the `POSTGRES_PASSWORD` to the password you set in the `.env` file.

```
POSTGRES_PASSWORD=postgres
```

OPTIONAL: Postgres version in 3.x branch is very old, version 12. Change the `image` to the version of the postgres image you have installed. For example:

```
image: postgres:15 or image: postgres:16
```

In result, your postgres changes on `docker-compose.yml` file should look like this:

```
  postgres:
    image: postgres:15
    restart: always
    ports:
      - '5432:5432'
    volumes:
      - postgres:/data/postgres
    environment:
      - POSTGRES_DB=chatwoot
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
```

### 6. Build the docker images for development

Navigate to the `chatwoot` directory and build the docker images for development:

```bash
# build base image first

docker compose build base
```

```bash
# build the rest of the images

docker compose build
```

```bash
# start the application

docker compose up -d
```

### 7. Setup rails server

Navigate to the `chatwoot` directory and setup the rails server:

```bash
cd chatwoot
# run db migrations
make db
# fireup the server
foreman start -f Procfile.dev
```

### 8. Open the application

Open the application in your browser:

```
http://localhost:3000
```

You should see the Chatwoot's login page.

[![Screenshot-2025-03-18-at-12-50-33.png](https://i.postimg.cc/L6S45Yhf/Screenshot-2025-03-18-at-12-50-33.png)](https://postimg.cc/SjZbtK6N)

Write the credentials to login:

```
Email: john@acme.inc
Password: Password1!
```

If the credentials are correct, you should see the Chatwoot's dashboard.

[![Screenshot-2025-03-18-at-12-55-41.png](https://i.postimg.cc/v8pRZcHy/Screenshot-2025-03-18-at-12-55-41.png)](https://postimg.cc/LhBQy41Q)

## Conclusion

You have successfully installed and setup Chatwoot on your local machine.

## Troubleshooting

### 1. Sidekiq is not launching

If you see the following error:

```bash
Connection refused - connect(2) for 127.0.0.1:6379 (redis://localhost:6379) sidekiq
```

Check:
- Redis is running from Docker Compose
- Redis URL is correct in the `.env` file

### 2. Chatwoot is not launching

Check if you launced the docker compose file with the following command:

- Check if you launched Docker Compose with the following command:
  ```bash
  docker compose up -d
  ```
- Check if you launched the Chatwoot itself:
  ```bash
  foreman start -f Procfile.dev
  ```
