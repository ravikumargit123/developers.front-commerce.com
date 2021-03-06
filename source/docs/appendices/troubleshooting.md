---
id: troubleshooting
title: Troubleshooting
---

Over the years, many developers have used Front-Commerce. A learning curve for such a solution exist, and we have detected recurring errors that may be difficult to troubleshoot for developers new to Front-Commerce or starting a new project.

This page contains the most common errors you may encounter, along with information that will help to solve them quickly.

## My server is not starting

> I cannot prepare the application or make it start

1. check the stdout/stderr of your server and the `/logs` folder, there may be useful information
2. try to prefix your command with `DEBUG="front-commerce:scripts"` to enter verbose mode (e.g: `DEBUG="front-commerce:scripts" npm run start`)
3. try to remove your `.front-commerce` and `node_modules` directories, run `npm install` and try again… `¯\_(ツ)_/¯`

## Authentication issue

> I am not logged in after entering valid credential information in the login form

1. check the stdout/stderr of your server and the `/logs` folder, there may be useful information
2. have you cleared your cookies?
3. does the response send you a cookie with a valid domain?
4. are secrets for your backend correctly set?
5. is the session store correctly set? (see `config/sessions.js`)
    * for Filesystem storage: ensure you have the permissions to write session files (the default location is `.front-commerce/sessions`)
    * using another storage: is its configuration valid?
    * having multiple Front-Commerce processes: ensure that you are using a session store that's compatible with a distributed architecture (ex: redis, memcached, etc.).

## Redirection loop

> When visiting any page, I get redirected infinitely

1. check the stdout/stderr of your server and the `/logs` folder, there may be useful information
2. what is the output of a `curl -I http://example.com` (replace with the url causing issues)
3. it could be a redirection from the shop detection mechanism that redirects to the default one because it cannot find one matching the current url
    1. turn on the configuration debug mode using `DEBUG="experimental:front-commerce:config"`
    2. is the list of `validUrls` correct? Are urls from `config/stores.js` valid?
    3. is the `url` logged the one you used in your browser? If it has the port in it, it's likely because your server proxy/load balancer is not well configured. Please configure it so it adds `X-Forwarded-Proto`, `X-Forwarded-Port` and `X-Forwarded-For` headers ([classic headers for proxies and load balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/x-forwarded-headers.html)). [Example configuration for nginx proxy](https://calvin.me/forward-ip-addresses-when-using-nginx-proxy/).
4. it is redirecting to HTTPS although I don't have HTTPS on my server
    1. if it is your production server, please fix this! This is a severe security issue for your website and your users. It can also negatively impact your SEO and the trust of your users.
    2. if you know what you are doing, please make sure that your proxy sets `X-Forwarded-Proto` with `https` ([Example configuration for nginx proxy](https://calvin.me/forward-ip-addresses-when-using-nginx-proxy/))

## JavaScript is not loading on my site

1. which browser are you using? You can use https://www.whatismybrowser.com/ to get detailed information from the browser.
2. is this browser supported by the list defined in the `browserslist` key of your `package.json` file?
3. do you see any fatal error in your browser console (DevTools) or in `logs/client.log`?

## SSR is broken

> In Dev mode, I see [the SSR Fallback page](/docs/advanced/theme/server-side-rendering.html#SSR-Fallback-when-things-go-wrong)

1. check the stdout/stderr of your server and the `/logs` folder, there may be useful information
2. are all pages broken or only some of them?
3. ensure that there is [no code using browser APIs loaded server side](/docs/advanced/theme/server-side-rendering.html#There-is-no-window-on-the-server)(e.g: Google Maps)

## My JS bundle too big

> When I look at the total size of JS assets on a single page, it is over my budget (> 500Ko for instance)

Build your application locally with the appropriate debug flag: `DEBUG=front-commerce:webpack npm run build`

* look for big libraries and try to avoid their usage client side if possible, or find smaller alternatives (e.g: [date-fns](https://date-fns.org/) over [moment.js](https://momentjs.com/))
* look for libraries achieving the same task, and see if you couldn't adapt your code to only use one of them
* ensure that there is no import of server side code in your client bundle
* look for libraries duplicated across different chunks, they may be candidates to code splitting using loadable components

See https://developers.google.com/web/fundamentals/performance/webpack/monitor-and-analyze for further information

## The data returned by my GraphQL server is wrong

> I expected GraphQL to return me some data but got something different

1. ensure that the data is also incorrect if you execute the same query in your `/playground` (only available if `FRONT_COMMERCE_ENV !== "production"`)
2. start your server with `DEBUG=axios npm run start`. It will show you HTTP calls to remote systems, so you could reproduce them to ensure that the remote API returns valid data
3. if data comes from ElasticSearch, start your server with `DEBUG="front-commerce:elasticsearch" npm run start` to view all ElasticSearch query. Try to run them manually to ensure they are correct and that indexed data are too
4. ensure your cache is up to date. If using redis, run [`redis-cli flushall`](https://redis.io/commands/flushall) to empty all keys
5. check for the resulting data in your GraphQL resolvers by adding `console.log()`s so you can follow where data are incorrectly transformed

## Another issue?

Please contact our support or open an issue describing the encountered problem along with your environment using `npx envinfo --system --binaries`
