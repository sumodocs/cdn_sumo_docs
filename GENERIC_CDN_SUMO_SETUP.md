# CDN Sumo Setup for Your Framework

We've got docs for using CDN Sumo with a variety of frameworks, but even if we don't have yours listed it doesn't mean you can't use CDN Sumo. Let us know what language and framework you're using so we can help prioritize our docs, and in the mean time read through this generalized CDN setup.

If there aren't detailed docs for your framework yet, not to worry you can still use CDN Sumo it might just take a little work on your part. If you have questions we're here to help you hello@cdnsumo.com, but we can't claim to be experts in every language and framework. You will need to do three things to integrate CDN Sumo with your application:

### 1) Pull the CDN Sumo URL from the Environment

When you provision a CDN from us, we'll return a url that you can use to access your CDN. We tell your app where the CDN is located by setting a config variable `CDN_SUMO_URL`. To be able to use CDN Sumo you'll need to be able to grab this config variable (also refered to as an environment variable).

In Ruby it looks like this:

    ENV['CDN_SUMO_URL']

In Python it looks like this:

    import os
    os.environ['CDN_SUMO_URL']

In Java it might look something like this [System.getenv](http://docs.oracle.com/javase/7/docs/api/java/lang/System.html#getenv()):

    String cdn_sumo_url = System.getenv("CDN_SUMO_URL");


If you don't know how to pull this value frome the environment for your language/framework look up "get environment variable using <my favorite> language". Once you've got the value of the environment variable now you're ready to use it.

### 2) Serve Assets from CDN URL

If you're running a website right now you can view source on the page to see that the majority of your assets (javascript, css, images, etc.) are likely being served from a relative path like:

    <img src="images/smile.png" />

What we want is to direct the browser to instead download the same asset from the same path at the CDN url, so we'll want them to serve something like this:

    <img src="http://d2ufsa5oa5lakja1.cdnsumo.com/images/smile.png" />


**Note:** Your CDN Sumo URL will be different and set by the enviornment variable `http://d2ufsa5oa5lakja1.cdnsumo.com` is just for demonstration purposes.

When a browser sees a link to `http://d2ufsa5oa5lakja1.cdnsumo.com/images/smile.png` it will tell `http://d2ufsa5oa5lakja1.cdnsumo.com` that it want's the file `images/smile.png`. If our CDN doesn't have the file yet, it will pull it from your server (at the same path) and store it, so future requests for the same file never hit your server again.

So how do you change the url for your static assets? This varies wildly by framework. It is possible to hard code this in based on your language:

Ruby using an ERB template:

    <img src='http://<%= ENV['CDN_SUMO_URL]'] %>/images/smile.png' />

Python using Jninja2 templates:

    <img src='http://{{ os.environ['CDN_SUMO_URL'] }}/images/smile.png' />

And so on.

It can be easier to make a helper function to do this automatically for you. Some frameworks already have these functions for example [Rails](http://rubyonrails.org/) has helper functions for javascript, css, and image rendering. You can then globally set the asset host using this incantation in your `config/environments/production.rb` file:

    config.action_controller.asset_host = ENV['CDN_SUMO_URL']


Check the documentation for your framework, if there are helper functions to render assets, it is likely that you can globally change the asset host on them, and if not demand this feature in the future :)


Once you have this set up properly on your code you can deploy. We recommend either testing the change on a staging application or on a not-heavily trafficked page until you get the bugs worked out.

You can [verify that your CDN is working](https://devcenter.heroku.com/articles/cdn_sumo#verifying-your-cdn-works) by following [these directions](https://devcenter.heroku.com/articles/cdn_sumo#verifying-your-cdn-works).

When your files are safely being served by our CDNs the next thing you need to worry about is invalidating old cached entries on the CDN.

### 3) Invalidating Cache via File or Path Name

Once the CDN caches a file it will hold on to it for a very long time, which is what you want. The longer the CDN holds onto that file, the more load it takes off of your server. This does leave a problems when you want to update a file, say change a background color from red to blue. If you update the file but your CDN doesn't get updated, that means your customer still sees the old file, this is bad but luckily its easy to avoid this problem.

Since the CDN caches based on the path and filename, if you change either of them the CDN will serve and cache the new file. Here is an example.

If your site is serving the file `smile.png` through our CDN:

    <img src="http://d2ufsa5oa5lakja1.cdnsumo.com/images/smile.png" />

If you change the path:

    <img src="http://d2ufsa5oa5lakja1.cdnsumo.com/v2/images//smile.png" />

Or the filename:

    <img src="http://d2ufsa5oa5lakja1.cdnsumo.com/images/smile-123asdf.png" />

Then the CDN will consider this a different file and serve the new file instead of the old one. So you have two options, when deploying new files you can change the path or change the filename.

#### 3.1) Change the Filename

Some frameworks support "fingerprinting" or "hashing" files to change their filenames. If your framework supports this, great! Changing the filename is the best method for invalidating a CDN cache as the filename won't change unless the contents change. This means that your cache will stay intact between deploys. If your framework doesn't support this or have any libraries that easily support this jump to 3.2 changing the path is much easier to program yourself.

Rails supports this feature by allowing you to toggle a flag to "digest" assets `config/environments/production.rb`

    config.assets.digest = true

So if your application has a css file called `application.css` and running an MD5 ([cryptographic hash](http://en.wikipedia.org/wiki/Cryptographic_hash_function)) on the file produces `a67mmfsa05owa5lrakqja1m`, then Rails will serve the file as:

    application-a67mmfsa05owa5lrakqja1m.css

If you change anything in the file like changing the color "blue" to "red" or changing tabs to spaces, it will generate a different "fingerprint" such as `b97amrsb84hdyvmai283jvhvua` and the resulting file name will be:

    application-b97amrsb84hdyvmai283jvhvua.css

Since your CDN sees these as two different files, it will serve the new updated file. If your framework supports this type of file hashing, it is the best way to invalidate your cache. Otherwise jump on down to the next section.


#### 3.2) Change the Path

When your application is being deployed you could make a folder based on a timestamp or a uuid like:

    public/assets-1359051725/

Then you can copy your images, css, and javascript files under this sub folder automatically on deploy:

    public/assets-1359051725/images
    public/assets-1359051725/javascripts
    public/assets-1359051725/stylesheets

Now every time you deploy you'll get a unique path name, you just need to make sure that assets are pointing to the correct path in the browser:

    <img src='http://{{ os.environ['CDN_SUMO_URL'] }}/assets-1359051725/images/smile.png' />

Of course you'll want to avoid hard coding that path so it is different on each deploy. Keep in mind that if you deploy frequently, it will cause your users to have to re-download files each time. This method is simpler but still has issues.


#### Out of Band Assets

One thing to watch out for on either of these methods is making sure images you send in emails are still valid, since an email may be opened weeks or months later. The easiest way around this is to send text only emails. Otherwise you might want to maintain a separate non-timestamped folder or unhashed-filename and serve your emails without going through a CDN. Some frameworks like Rails take care of this for you, if you're coding support from scratch, this is something you'll need to consider.


#### Finish

We're working on more fully featured documentation for more frameworks and languages, hopefully in the mean time we hope this information has been useful to you. If you have questions about these instructions, or want to provide feedback or even help with framework specific docs please contact us: hello@cdnsumo.com.



©CDN Sumo 2013, All Rights Reserved