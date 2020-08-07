A customer wanted to have about 100 minisites, all the same application, in 5 different variants, on a single server.

Besides adding sites dynamically, there was a requirement that a site could change its variant or theme using the backoffice.

So we had essentially 5 directories of templates, and we needed django to load templates from the right directory for each site.

I didn't find a nice way to do it. Not as easy as setting the url router based on site in a middleware.

Django assumes that loading a template by name always loads the same template for all requests to all sites.

The solution involved a little monkeypatching, and it worked great. It still works.

Too bad the business idea of the customer did not pan out.
