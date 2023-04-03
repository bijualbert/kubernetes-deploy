## Kubernetes auto-deployments (EXPERIMENTAL)

This is a repository that builds a Docker Image with all scripts needed to
deploy to Kubernetes from GitLab CI.

It is used to give the [GitLab Demo](https://about.gitlab.com/handbook/sales/demo/) which contains detailed instructions to reproduce.

It basically consists of two stages:
1. Build stage where a Docker Image is built,
2. Deploy stage where a previously built Docker Image is run on Kubernetes and
   exposed on hostname.

### Build stage

The build script does:
1. Check if the repository has a `Dockerfile`,
2. If yes, use `docker build` to build Docker Image,
3. If no, use [herokuish](https://github.com/gliderlabs/herokuish) to build
   and package a buildpack based application,
4. Login to GitLab Container Registry,
5. Push build image to GitLab Container Registry.

### Deploy stage

The deploy script does:
1. Create a new namespace if it does not exist already.
1. Deploy Postgres database with preconfigured username, password and database name.
1. Deploy an application with most recent Docker Image.
1. Create or update ingress to expose the application under hostname.

### PostgreSQL support

During deployment automatically `PostgreSQL` is provisioned unless `DISABLE_POSTGRES` is specified.
We currently use preconfigured credentials. These credentials are used for defining `DATABASE_URL`
of format: `postgres://user:password@postgres-host:postgres-port/postgres-database`.

### Requirements

1. GitLab Runner using Docker or Kubernetes executor with privileged mode enabled.
2. Service account for existing Kubernetes cluster.
3. DNS wildcard domain to host deployed applications.

### Limitations

1. Public and private docker images can be deployed, but credentials are accessible during deployment.
1. There is no ability to pass environment variables to deployed application.
1. Provisioned database uses immutable storage: all data will be lost after container restart.

### Variables

1. `DISABLE_POSTGRES: "yes"`: disable automatic deployment of PostgreSQL,
1. `POSTGRES_USER: "my-user"`: use custom username for PostgreSQL,
1. `POSTGRES_PASSWORD: "password"`: use custom password for PostgreSQL,
1. `POSTGRES_DB: "my database"`: use custom database name for PostgreSQL,

### Examples

You can see existing working examples:
1. [Ruby](https://gitlab.com/gitlab-examples/ruby-openshift-example/)

### How to contribute?

Simply fork this repository. As soon as you push your changes,
the new docker image with all scripts will be built.
You can then start using your own docker image hosted on your Container Registry.

### How to use it?

Basically, configure the Kubernetes Service in your project settings and
copy-paste [this `.gitlab-ci.yml`](https://gitlab.com/gitlab-org/gitlab-ci-yml/blob/master/autodeploy/Kubernetes.gitlab-ci.yml).

### Remarks

This project uses latest version of `kubectl` in order to fix problems with `kubectl rollout status`.
As of today it is `v1.7.0-alpha-3`.

### License

MIT, GitLab, 2016-2017
