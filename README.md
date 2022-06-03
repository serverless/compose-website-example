This project shows how to use [Serverless Framework Compose](https://github.com/serverless/compose) to deploy frontend websites (React, VueJS, etc.).

It contains 2 examples:

- [simple-website/](./simple-website): a simple website
- [fullstack-app/](./fullstack-app): a React website and an API

# Documentation

Serverless Framework can deploy multiple services via a `serverless-compose.yml` configuration file ([learn more](https://www.serverless.com/framework/docs/guides/compose)).

The `serverless-compose.yml` file can now also deploy new types of services, called *components*.

## Deploying websites

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
    path: website
```

### How it works

On `serverless deploy` the website files are uploaded to an S3 bucket. That bucket serves the website publicly.

The first deployment takes care of creating the S3 bucket via **CloudFormation**. The next deployments only take seconds as they only upload the changed files.

When setting up a custom domain however, `serverless deploy` will also set up a CloudFront distribution automatically. That will make the website ready for production with HTTPS, CDN caching at the edge, as well as automatic security HTTP headers. On deployment, the CDN cache will automatically be cleared.

*Note: server-side rendering (SSR) is not supported (yet). Let us know if you are interested!*

### Building the website

The website component can optionally build the website before deploying. This is configured via the `build` option:

```yaml
# serverless-compose.yml
services:
  website:
    component: '@serverless-components/website'
    path: website
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

NextJS can be deployed [without SSR](https://nextjs.org/docs/advanced-features/static-html-export):

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

Compose lets us use [service outputs using variables](https://www.serverless.com/framework/docs/guides/compose#service-dependencies-and-variables). Website components expose the following outputs:

- `url`: the URL of the website
- `domain`: the domain of the website
- `bucketName`: the name of the bucket created to host the website
- `cname`: domain to point custom domains to in DNS records (when using a custom domain)
- `distributionId`: ID of the CloudFront distribution (when using a custom domain)

You can preview these outputs by running `serverless outputs`.

Here is an example how website outputs can be used, for example to inject the website URL in a serverless service:

```yaml
services:
  website:
    component: @serverless-components/website
    path: website

  api: # Serverless Framework service
    path: api
    params:
      websiteUrl: ${website.url}
```

### Configuration reference

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
    domain: my-website.com
    certificate: arn:aws:acm:us-east-1:1234567890:certificate/0a28e63d-d3a9-14347bfe8123
    redirectToMainDomain: true
    build:
      cmd: npm run build
      outputDir: build
      environment:
        FOO: bar
    security:
      allowIframe: true
```

#### Path

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
```

The `path` option should point to the directory containing the website. You can use `path: .` to point to the current directory.

All files in that directory will be deployed and made available publicly, unless you configure a `build` step (see below).

#### Build

Compose can optionally build the website before deploying. This is configured via the `build` option:

```yaml
services:
  website:
    component: @serverless-components/website
    path: website
    build:
      cmd: npm run build
      outputDir: build
      environment:
        FOO: bar
```

- `cmd`: the CLI command that builds the website (will run in the `path` directory)
- `outputDir`: the directory containing the built website, which will be uploaded instead of `path` (relative to the `path` directory)
- `environment`: key-value of environment variables to set in the build process

#### Custom domains

It is possible to set a custom domain via the `domain` and `certificate` options:

```yaml
services:
  website:
    # ...
    domain: my-website.com
    # ARN of an ACM certificate for the domain, registered in us-east-1
    certificate: arn:aws:acm:us-east-1:1234567890:certificate/0a28e63d-d3a9-14347bfe8123
```

With the configuration above, running `serverless deploy` will set up a CloudFront CDN distribution, configured with the custom domain `my-website.com`, using the provided HTTPS certificate.

After running `serverless outputs`, you should see the following output in the terminal:

```
website:
  ...
  url: https://my-website.com
  cname: s13hocjp.cloudfront.net
```

Configure your DNS records to create a CNAME entry that points your domain to the `xxx.cloudfront.net` domain. After a few minutes/hours, the domain should be available.

##### HTTPS certificate

To create an HTTPS certificate:

- Open [the ACM Console](https://console.aws.amazon.com/acm/home?region=us-east-1#/wizard/) in the `us-east-1` region (CDN certificates **must be** in us-east-1, regardless of where your application is hosted)
- Click "_Request a new certificate_", add your domain name and click "Next"
- Choose a domain validation method:
    - Domain validation will require you to add CNAME entries to your DNS configuration
    - Email validation will require you to click a link in an email sent to `admin@your-domain.com`

After the certificate is created and validated, the ARN of the certificate will be displayed.

##### Multiple domains

It is possible to set up multiple domains:

```yaml
services:
  website:
    # ...
    domain:
      - my-website.com
      - app.my-website.com
```

##### Redirect all domains to a single one

It is sometimes necessary to redirect all domains to a single one. A common example is to redirect the root domain to the `www` version.

```yaml
services:
  website:
    # ...
    domain:
      - www.my-website.com
      - my-website.com
    redirectToMainDomain: true
```

The first domain in the list will be considered the main domain. In this case, `my-website.com` will redirect to `www.my-website.com`.

#### Region

The `region` option lets us deploy the S3 bucket to a specific region:

```yaml
services:
  website:
    # ...
    region: eu-west-1
```

Note that when using a custom domain, the website is served behind [the CloudFront CDN](https://aws.amazon.com/cloudfront/), which caches the website all over the world.

#### Profile

Use the `profile` option to deploy the website with a specific AWS profile:

```yaml
services:
  website:
    # ...
    profile: production
```

#### Allow iframes

When deploying to production with a custom domain, recommended HTTP security headers are added by default to the website. That means that [for security reasons](https://scotthelme.co.uk/hardening-your-http-response-headers/#x-frame-options), the website cannot be embedded in an iframe.

To allow embedding the website in an iframe, enable it explicitly:

```yaml
services:
  website:
    # ...
    security:
      allowIframe: true
```

## Sharing Compose state

Even though CloudFormation is used for deploying, Compose also stores deployment state and outputs. That allows accelerating future deployments by remembering the state of the previous deployment.

This state is stored by default in the `.serverless/` directory, but it can also be stored remotely, which **is recommended in a team environment**.

The simplest solution to store the state remotely is via the `state: s3` autoconfiguration in `serverless-compose.yml`:

```yaml
# serverless-compose.yml
state: s3

services:
  # ...
```

On the next `serverless deploy`, the configuration above will automatically create an S3 bucket to store the state.

If you want more control, you can also set the following options in the `state` configuration:

```yaml
# serverless-compose.yml
state:
  backend: s3
  # Use a specific AWS profile to deploy and access the state bucket
  profile: infra
  # Use a specific S3 key prefix for objects stored in S3
  prefix: europe-team
```

### Manual setup

For teams that want complete control, it is possible to manually create the S3 bucket containing the state.

Once it is created, set the name of the bucket in the `existingBucket` key:

```yaml
# serverless-compose.yml
state:
  backend: s3
  existingBucket: my-serverless-state-storage
  # Use a specific AWS profile to deploy and access the state bucket (optional)
  profile: infra
  # Use a specific S3 key prefix for objects stored in S3 (optional)
  prefix: europe-team
```
