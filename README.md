This project shows how to use [Serverless Framework Compose](https://github.com/serverless/compose) to deploy frontend websites (React, VueJS, etc.).

It contains 2 examples:

- [simple-website/](./simple-website): a simple website
- [fullstack-app/](./fullstack-app): a React website and an API

# Documentation

Serverless Framework can deploy multiple services via a `serverless-compose.yml` configuration file ([learn more](https://www.serverless.com/framework/docs/guides/compose)).

The `serverless-compose.yml` file can now also deploy new types of services, called *components*.

### Deploying websites

The first component available is "website", which deploys frontend websites like React or VueJS. To use it, install it via NPM:

```sh
npm i @serverless/compose @serverless-components/website
```

You can then use it in `serverless-compose.yml`:

```yaml
# serverless-compose.yml
services:
  website:
    component: '@serverless-components/website'
    path: public
```

On `serverless deploy`, the files in the `public/` directory will be deployed as a public website.

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

### How it works

On `serverless deploy` the website files are uploaded to an S3 bucket. That bucket serves the website publicly.

The first deployment takes care of creating the S3 bucket via **CloudFormation**. The next deployments only take seconds as they only upload the changed files.

When setting up a custom domain however, `serverless deploy` will also set up a CloudFront distribution automatically. That will make the website ready for production with HTTPS, CDN caching at the edge, as well as automatic security HTTP headers.

Note: server-side rendering (SSR) is not supported (yet). Let us know if you are interested!

### Building the website

The website component can optionally build the website before deploying. This is configured via the `build` option:

```yaml
# serverless-compose.yml
services:
  website:
    component: '@serverless-components/website'
    path: my-website
    build:
      cmd: npm run build
      outputDir: dist
```

In the example above, `serverless deploy` will run `npm run build` in the my-website directory. It will then upload the `dist` subfolder.

The main benefit is that we can now inject dynamic environment variables, for example the API URL, into our published website. For example:

```yaml
services:

  # Serverless Framework service
  api:
    path: api

  website:
    component: @serverless-components/website
    path: website
    build:
      cmd: npm run build
      outputDir: build
      environment:
        # Inject the API URL in the React website
        REACT_APP_API_URL: ${api.HttpApiUrl}
```

### Frameworks

#### React

Here is an example on how to build and deploy a React website:

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
    build:
      cmd: npm run build
      outputDir: build
```

#### VueJS

Here is an example on how to build and deploy a VueJS website:

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
    build:
      cmd: npm run build
      outputDir: dist
```

#### NextJS

NextJS can be deployed [without SSR, as a static website](https://nextjs.org/docs/advanced-features/static-html-export):

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
    build:
      cmd: npx next build && npx next export
      outputDir: out
```

### Output variables

### Configuration reference

## Sharing Compose state
