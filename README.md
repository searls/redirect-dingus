# Searls's Redirect Dingus app

Problem statement: You have a domain you want to redirect to some other domain. HTTPS redirects require both an active DNS zone, valid HTTPS certificate, and an SNI endpoint. This is not a free or automatic at many DNS hosts. My Redirect Dingus app can be deployed and configured once to support arbitrarily many domains and subdomain mappings, and can run on a single affordable [eco dyno](https://devcenter.heroku.com/articles/eco-dyno-hours).

What is it? It's a tiny Heroku [nginx](https://en.wikipedia.org/wiki/Nginx) app that simply reads a couple environment variables and uses them to map request hostnames to your intended redirect targets for cases when you have some number of domains and subdomains that should redirect to some other canonical domain.

For instance, Becky owns [buildwithbecky.com](https://buildwithbecky.com), but wants it to redirect to [betterwithbecky.com](https://betterwithbecky.com). And I own [possyparty.com](https://possyparty.com) just in case anyone misspells [posseparty.com](https://posseparty.com).

So here's how to use this repo:

1. Fork it
2. [Create a new Heroku app](https://devcenter.heroku.com/articles/creating-apps) and provision an eco dyno to run nginx (see the repo's [Procfile](/Procfile))
3. Add the [nginx buildpack](https://github.com/heroku/heroku-buildpack-nginx) with something like `heroku buildpacks:add --app your-app-name heroku-community/nginx`
4. Configure redirect domains (see [below](#configuring-your-redirect-domains))
5. Set up HTTPS by [enabling Heroku automated certificate management](https://devcenter.heroku.com/articles/automated-certificate-management)
6. Define domains (see [below](#configure-custom-domains))

## Configuring your redirect domains

First, set your default/catch-all redirect domain to the `REDIRECT_DEFAULT_DOMAIN` environment variable:

```sh
heroku config:set -a your-app-name REDIRECT_DEFAULT_DOMAIN="www.betterwithbecky.com"
```

Then, if you have multiple domains to redirect, you can set them to the `REDIRECT_DOMAIN_MAP` environment variable as a JSON object with a single command like this:

```sh
heroku config:set -a your-app-name REDIRECT_DOMAIN_MAP='{
  "members.buildwithbecky.com": "members.betterwithbecky.com",
  "gram.buildwithbecky.com": "gram.betterwithbecky.com",
  "admin.buildwithbecky.com": "admin.betterwithbecky.com",
  "www.buildwithbecky.com": "www.betterwithbecky.com",
  "buildwithbecky.com": "betterwithbecky.com",
  "www.possyparty.com": "www.posseparty.com",
  "possyparty.com": "posseparty.com"
}'
```

## Configure custom domains

Each [root domain](https://devcenter.heroku.com/articles/custom-domains#add-a-custom-root-domain) and subdomain [wildcard domain](https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain) need to be specified (and effectively "claimed", as each domain can only be owned by a single Heroku application at a time) in the Heroku dashboard or CLI.

```sh
heroku domains:add yourdomain.com -a your-app-name
heroku domains:add *.yourdomain.com -a your-app-name
```

In my case, the custom domains look like this in the dashboard:

![Heroku domain configuration showing two root and two wildcard domains](https://github.com/user-attachments/assets/af293cd8-7aae-4adb-bff6-9ae30674440e)

Once those have been specified, point each root domain to the goofily-named target generated by Heroku with an `ALIAS` record (because you're using [DNSimple](https://dnsimple.com), right?) and each wildcard domain with a `CNAME` record.

In the case of POSSE Party, that looks like this in my DNS settings:

![DNSimple settings for POSSE Party](https://github.com/user-attachments/assets/3cdd8a3e-230f-43a8-b1da-87e80672f8f9)
