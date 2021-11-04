This repository contains unofficial patterns, sample code, or tools to help developers build more effectively with [Fauna][fauna]. All [Fauna Labs][fauna-labs] repositories are provided “as-is” and without support. By using this repository or its contents, you agree that this repository may never be officially supported and moved to the [Fauna organization][fauna-organization].

[fauna]: https://www.fauna.com/
[fauna-labs]: https://github.com/fauna-labs
[fauna-organization]: https://github.com/fauna

---

# Edge Gateway (Cloudflare)

> Generated from template 
> [https://github.com/cloudflare/worker-template-router](https://github.com/cloudflare/worker-template-router)

---

This is a sample "Edge API Gateway" using Cloudflare Workers to provide routing for `GET`, `PUT` and `POST` `/users`
rest API endpoints. The routes proxy requests to "resolvers" on the Fauna side, which are discussed in
[this project](https://github.com/fauna-labs/vue-fauna-edge-api).

## Setup

1. [Install wrangler](https://developers.cloudflare.com/workers/cli-wrangler/install-update).
   
2. Login to your Cloudflare account
   ```
   wrangler login
   ``` 
   This will open a web page asking you to login to Cloudflare. 
   After you complete the login process, there should be an API Token in your session for wrangler

3. Edit `wrangler.toml` found in this folder (`/edge/cloudflare/wrangler.toml`):
   * Provide a value for your `account_id`.
   * (Optional) Override the value in `name` if you need to.
   * Set the `FAUNADB_DOMAIN` environment variable to one of the following values depending on which
    [Region Group](https://docs.fauna.com/fauna/current/api/fql/region_groups#how-to-use-region-groups)  
    your database is in.

      | Region Group       | FAUNADB_DOMAIN |
      | ------------------ | --------------------------- |
      | Classic            | `db.fauna.com`              |
      | United States (US) | `db.us.fauna.com`           |
      | Europe (EU)        | `db.eu.fauna.com`           |

4. Publish to Cloudflare workers
   ```
   wrangler publish
   ```

## Next Steps

At this point, `GET`, `PUT` and `POST` `/users` should be live but you still need to deploy the resolvers 
(Fauna User Defined Functions, UDFs) on Fauna. Hop over to 
[this project](https://github.com/fauna-labs/vue-fauna-edge-api#1-create-fauna-resources)
and complete the stepss **1. Create Fauna Resources** and **2. Setup External Authentication with Auth0**
to finish the setup.

---

## (Optional) Deploy to a custom domain
### Prerequisites
Be sure to have already 
[added a Site and Domain](https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website)
in Cloudflare.

### Configure DNS and route
In Clouldflare:
1. Click on your Site settings.
2. Click [DNS]. Then click [Add record]:
   * Type = **A**
   * Name = *Enter a value....e.g. `app`*
   * IPv4 address = **192.0.2.1**
    > This is a bogus value but is needed to add the record into the DNS. `192.0.2.1` is chosen because it is rarely used. 
      Any value will do. In the next step we'll route the DNS record to the deployed Worker (`192.0.2.1` will be overridden).
3. Save
4. Click [Workers]. Then click [Add route]
   * Route = The route matching the DNS record defined above plus a wildcard path (`/*`) *e.g.* `app.mydomain.com/*` 
   * Worker = *Select the worker deployed previously*
    > The router code in the Worker will handle all the routing. So we are routing everything (`/*`) to it.
5. Save

---

## (Optional) Use Cloudflare "Workers Sites" to deploy the sample Vue app to your Domain
A Sample Vue app that accompanies this "Edge API Gateway" project has been provided
[here](https://github.com/fauna-labs/vue-fauna-edge-api). This section describes how to use Cloudflare's
[Workers Sites](https://developers.cloudflare.com/workers/platform/sites) to deploy the sample Vue app.

### Prerequisites
Be sure to have already:
1. [Compiled and minified](https://github.com/fauna-labs/vue-fauna-edge-api#4-setup-the-spa)
   the Vue app before beginning.
   * **The compiled assets will be in the `/dist` folder of the project**. 
2. [Added a Site and Domain](https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website)
   in Cloudflare.
3. Deployed the API to your custom domain in the previous step.

### Deploy the Vue app
1. Create a new project directory and run this Wranger command:
   ```bash
   wrangler init --site vue-app-edge-gateway
   ```
   This command generates a `wrangler.toml` file and a `workers-site` directory.
2. Edit the `wrangler.toml` file:
   * Add (or overwrite) an entry `account_id=<<your Cloudflare account id>>`.
   * Edit `bucket` to point to the `/dist` directory of the [sample Vue app](https://github.com/fauna-labs/vue-fauna-edge-api). 
3. You can preview your site by running
   ```
   wranger dev
   ```
4. Publish to Cloudflare workers
   ```
   wrangler publish
   ```
5. Note down the url it was deployed to. The value is derived from the `name` configuration in `wrangler.toml`
   and your Cloudflare Workers subdomain. i.e. `https://<name>.<subdomain>`.
   e.g. `https://vue-app-edge-gateway.your-subdomain.workers.dev`

### Serve the SPA and API from the same Cloudflare Worker
1. Head back to the this project's `wrangler.toml` file and edit the value for `SPA_HOST`. 
   * Set the value equal to the hostname of the deployed SPA. e.g. `vue-app-edge-gateway.your-subdomain.workers.dev`
2. Publish the worker one more time:
   ```
   wrangler publish
   ```
> The SPA should  now be at the "`/`" path of your custom domain. And the APIs at `/users`.