# Docker Compose for Kong API Gateway with TLS support

This project was created in order to have an optimal configuration to deploy [Kong API Gateway](https://docs.konghq.com/gateway/2.8.x/get-started/comprehensive/) using a [Docker Compose](https://docs.docker.com/compose/compose-file/compose-file-v3).

**TLS** support is also implemented to allow you to launch **Kong** with an **SSL certificate** and call the **proxy** or the **administration API** in **HTTPS**.

Follow all the steps, and you will have everything working in a very simple way.

```bash
$ cd path/to/workspace
$ git clone https://github.com/akanass/kong.git | git@github.com:akanass/kong.git
```

Make sure you have the [latest version of Docker](https://docs.docker.com/get-docker/) installed on your machine so that the commands used are the same.

## Docker Compose default configuration

By default, this `docker-compose.yml` will launch **Kong** without **TLS** support and [Postgresql v11](https://www.postgresql.org/docs/11/index.html).

If you want a more recent version of **Postgresql**, you can go up to **version 13** but not more because there is currently an issue with **Kong** compatibility as shown in the open issue [here](https://github.com/Kong/kong/issues/8259#issuecomment-1103600703).

To do this, you just have to change the image in this place:

```yaml
services:
  kong-database:
    image: postgres:13-alpine
```

**Postgresql** database backup data will be put in the `./postgresql` directory created on the fly when launching the container.

## Launch Kong API Gateway

```bash
$ cd path/to/workspace/kong
$ docker compose up -d
```

Ensure that you can send requests to the **gateway’s Admin API** using `cURL`.

View the current configuration by issuing the following command in a terminal window:

```bash
$ curl -i -X GET http://<admin-hostname>:8001
```

The current configuration returns.

**Kong API Gateway** will run on port `8000`.

## Launch Kong API Gateway with TLS support

To be in the same conditions as in production, it is preferable to launch **Kong** with **TLS** support even in the development phase so as not to have any surprises.

To do this, you will have to provide **Kong but also all the services of your stack**, the **SSL certificate** and its **private key** so that you can communicate in **HTTPS**.

The easiest way to generate this data is to follow the tutorial on this [repository](https://github.com/akanass/self-signed-certificate-with-custom-ca).

Once the **SSL certificate** and its **private key** have been generated, you can copy them to the `./ssl` directory of this project by renaming them `kong-crt.pem` and `kong-key.pem`.

In order for the **SSL certificate** and its **private key** to be used by **Kong** in the container, their **owner** must be changed to that of the user `kong` of the container:

```bash
$ cd path/to/workspace/kong
$ sudo chown 100:65533 ./ssl/*
```

**Don't forget** to also use the generated **SSL certificate** and its **private key** or the `.p12` file in your **services** and **applications** in order to have everything in **HTTPS**.

You must now change some configuration data in the `docker-compose.yml` file in order to map on the **HTTPS ports** and that the **SSL certificate** and its **private key** are taken into account:

```yaml
services:
  kong-gateway:
    ports:
#      - '8000:8000' # comment proxy http port
      - '8443:8443' # uncomment proxy https port
#      - '8001:8001' # comment admin http port
      - '8444:8444' # uncomment admin https port
    environment: # uncomment all SSL data
      - 'KONG_SSL_CERT=/usr/local/custom/kong/ssl/kong-crt.pem'
      - 'KONG_SSL_CERT_KEY=/usr/local/custom/kong/ssl/kong-key.pem'
      - 'KONG_ADMIN_SSL_CERT=/usr/local/custom/kong/ssl/kong-crt.pem'
      - 'KONG_ADMIN_SSL_CERT_KEY=/usr/local/custom/kong/ssl/kong-key.pem'
```

When done, you can launch **Kong** with **TLS** support:

```bash
$ cd path/to/workspace/kong
$ docker compose up -d
```

Ensure that you can send requests to the **gateway’s Admin API** using `cURL`.

View the current configuration by issuing the following command in a terminal window:

```bash
$ curl -i -X GET https://<admin-hostname>:8444
```

The current configuration returns.

**Kong API Gateway** will run on port `8443`.

**ATTENTION:** If you use [Postman](https://www.postman.com/downloads/) to make your requests, you must add the **CA** generated previously in the **settings/certificates** section of **Postman** so that the **HTTPS** requests are validated.
