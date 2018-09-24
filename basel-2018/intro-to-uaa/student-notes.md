# Introduction to Cloud Foundry UAA

## Introduction

The Cloud Foundry UAA (User Account & Authentication) was created for Cloud Foundry Application Runtime itself, but its influence has broadened to BOSH, CredHub, Concourse, and a growing beyond the Cloud Foundry ecosystem. Your own applications can use the UAA to authenticate your users, and authorize selective behaviour, including bridging to federated user directories such as ActiveDirectory; and applications can use the UAA to authorize access between each other.

In this 30 minute lab we will be doing the following:

* Run the UAA locally (or upon a jumpbox) using [Quaa - Quick UAA](https://github.com/starkandwayne/quaa)
* Progressively enhance some a sample application with UAA authorization

## Learning Objectives

1. The UAA is a simple Java application that runs within an environment such as Apache Tomcat, or with the Cloud Foundry Java Buildpack. It has an in-memory database; but supports PostgreSQL and MySQL for long-term data persistence.
1. The UAA has a web UI for users to login, grant authorization to client applications to their data, revoke authorization grants, and perform multi-factor authentication
1. The UAA has an HTTP API to act as an OAuth2 Authorization Server, an OpenID Connect user information provider, and for clients to configure the UAA itself (create/modify/delete new clients, users, etc).
1. Backend APIs, called Resource Servers, can delegate authentication and authorization to the UAA.
1. Client applications, such as user-facing web apps or CLIs, pass access tokens to resource servers. They do not necessarily know identity information about their human user or another application client.
1. All applications can be written in any programming language and hosted anywhere. The OAuth2 standard is well recognized and client libraries exist everywhere.

## Prerequisites

To run the UAA locally requires:

1. Java 8 / Java 1.8

    ```plain
    $ java -version
    java version "1.8.0_66"
    ```

    **NOTE:** Apache Tomcat may not work on Java 9 / Java 10; hence requirement for specifically Java 8.

1. Git

    ```plain
    $ git --version
    git version 2.17.1 (Apple Git-112)
    ```

To run the example applications requires:

1. Docker CLI and Docker Daemon running

    ```plain
    $ docker version
    Client:
     Version:           18.06.1-ce
      ...
    Server:
     Engine:
      Version:          18.06.1-ce
      ...
    ```

1. Docker image for sample application

    ```plain
    docker pull starkandwayne/uaa-example-resource-server
    ```

1. Example application source code:

    ```plain
    git clone https://github.com/starkandwayne/ultimate-guide-to-uaa-examples ~/workspace/ultimate-guide-to-uaa-examples
    ```

## Lab

Lab steps:

1. Quickly run UAA locally
1. Run a "resource server" for anonymous users

### Quickly run UAA locally

We will use the [Quick UAA Local](https://github.com/starkandwayne/quick-uaa-local/) project to download all remaining dependencies and run a local UAA.

On MacOS/Homebrew:

```plain
brew install starkandwayne/cf/quaa
```

On Linux/MacOS:

```plain
git clone https://github.com/starkandwayne/quick-uaa-local ~/workspace/quick-uaa-local
cd ~/workspace/quick-uaa-local
```

Either run `direnv allow` if prompted, or:

```plain
eval "$(bin/quaa env)"
```

On both:

To run a local UAA:

```plain
quaa up
```

### Run a "resource server" for anonymous users

The example resource server (data API to be protected by UAA later) is a list of Australian Airports.

The application supports anonymous users - those who interact with the API without any valid authorization.

We will use environment variables to configure the application with our local UAA:

```plain
quaa env

eval "$(quaa env)"
echo $UAA_URL
```

Either run with Docker, or investigate the [different Ruby/Golang implementations](https://github.com/starkandwayne/ultimate-guide-to-uaa-examples)

```plain
docker run -ti -p 9292:9292 -e UAA_URL=$UAA_URL -e UAA_CA_CERT=$UAA_CA_CERT starkandwayne/uaa-example-resource-server
```

The Airport API resource server application is now running on port `:9292`. We can interact with it using `curl`:

```plain
$ curl localhost:9292
[
  {
    "Airport ID": 3317,
    "Name": "Brisbane Archerfield Airport",
    "City": "Brisbane",
    "Country": "Australia",
    "IATA": "\\N",
    "ICAO": "YBAF",
```

If you have the `jq` CLI you can see that as an anonymous user we receive 10 results only:

```plain
$ curl localhost:9292 | jq length
10
```

See the [Ruby example source code](https://github.com/starkandwayne/ultimate-guide-to-uaa-examples/blob/master/ruby/resource-server/config.ru#L24-L27)

## Learning Objectives Review

## Beyond the Lab

* WIP book - [Ultimate Guide to UAA](https://www-staging.ultimateguidetouaa.com/)