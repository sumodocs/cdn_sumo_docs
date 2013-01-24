# CDN Sumo Setup Docs

Congratulations on taking the first steps to speed up your site, and better serve your customers by using CDN Sumo. This repo contains all the framework specific documentation that isn't covered on Heroku's [Dev Center](https://devcenter.heroku.com/articles/cdn_sumo). If you're familiar with CDNs and CDN Sumo you can skip to "Docs by Framework" below.

## What is a CDN?

A CDN is used to distribute content around the world so that it is closer to your customers, and closer content means faster content. By using a CDN to serve static assets of your website, you can speed up page load time and reduce load on your website at the same time. When you provision CDN Sumo a CDN will be created just for you, all you have to do is configure your app to use it.


## Provisioning CDN Sumo

CDN Sumo can be attached to a Heroku application via the CLI:

    $ heroku addons:add cdn_sumo
    -----> Adding cdn_sumo to sharp-mountain-4005... done, v18 (free)

Once CDN Sumo has been added a `CDN_SUMO_URL` setting will be available in the app configuration and will contain the URL used to host your assets. This can be confirmed using the `heroku config:get` command.

    $ heroku config:get CDN_SUMO_URL
    d2ufsa5oa5lakja1.cdnsumo.com

**Note:** It can take up to 10 minutes to fully provision your CDN, in the mean time your `CDN_SUMO_URL` is set to your own herokuapp.com domain so that you can use the environment variable without having to wait. When your CDN is ready the variables are swapped and will receive a confirmation email.

## How does a CDN work with my app?

The CDN that you receive from CDN Sumo dynamically grabs and caches assets from your domain, so you don't need to sync. When you provision a CDN with CDN Sumo you will set the config var `CDN_SUMO_URL`. You can then set up your app so that it instructs browsers to deliver content from this domain instead of your own server. This works especially well if you are using a framework like the Rails asset pipeline to deliver assets.

## Docs by Framework

- [Ruby on Rails 3.2 +](https://devcenter.heroku.com/articles/cdn_sumo#cdn-sumo-with-the-rails-asset-pipeline)


## General Documentation

If you don't see your Framework listed above, we have detailed documentation that may help you integrate CDN Sumo with another Framework.

- [CDN Sumo Setup for Your Framework](https://github.com/sumodocs/cdn_sumo_docs/blob/master/GENERIC_CDN_SUMO_SETUP.md)

## Feedback

If you see something inaccurate or confusing please open a pull request or drop us a line and we will fix it. If you want to request documentation for a specific framework please let us know hello@cdnsumo.com so we can better prioritize our workload. If you've integrated CDN Sumo with a framework not already listed here, please reach out to us with details so we can help others.

©CDN Sumo 2013, All Rights Reserved