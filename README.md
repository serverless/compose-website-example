This project shows how to use [Serverless Framework Compose](https://github.com/serverless/compose) to deploy frontend websites (React, VueJS, etc.).

It contains 2 examples:

- [simple-website](./simple-website): how to deploy a simple website without anything else
- [fullstack-app](./fullstack-app): how to deploy a React website and an API

# Documentation

Serverless Framework can deploy multiple services via a `serverless-compose.yml` configuration file ([learn more](https://www.serverless.com/framework/docs/guides/compose)).

The `serverless-compose.yml` file can now also deploy new types of services, called *components*.

The first component available is `@serverless-components/website`, which deploys frontend websites like React or VueJS. Here is how it can be used in `serverless-compose.yml`:

```yaml
# serverless-compose.yml
services:
  website:
    component: '@serverless-components/website'
    # Deploys the website files of the `public/` directory
    path: public
```

Websites can be deployed alone (as shown above), or alongside other Serverless Framework services:

```yaml
# serverless-compose.yml
services:

  # This is a Serverless Framework service
  api:
    # There is a `serverless.yml` file in the `api/` folder
    path: api

  # This is a website component
  website:
    component: '@serverless-components/website'
    path: public
```
