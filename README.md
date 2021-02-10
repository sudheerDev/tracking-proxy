# Mixpanel Tracking Proxy Server
An Nginx config to proxy Mixpanel's Ingestion API. This can be used to set up your own servers to load the JS library and proxy tracking requests to Mixpanel.

Related content:
- [Mixpanel Ingestion API Documentation](https://developer.mixpanel.com/reference/ingestion-api)
- [Mixpanel JavaScript Documentation](https://developer.mixpanel.com/docs/javascript)

## Installation

There are a few ways you can use this repo to deploy a server that can be use to proxy Mixpanel API requests: one-click deploy to cloud, build a docker image, or copy and paste the nginx settings to your own nginx config file.

#### Option 1: One-click Deploy
   - [Run on Google Cloud](https://deploy.cloud.run)
   - [Deploy to DigitalOcean](https://cloud.digitalocean.com/apps/new?repo=https://github.com/jbwyme/mixpanel-proxy-test/tree/main)
   
#### Option 2: Docker Image
   Assuming you have Docker installed on your system, you can do the following:
   
   1. Clone the repo
   2. Build the Docker image: `docker build -t mixpanel-proxy`
   3. Run a container using the image: `docker run --name mxpl -d -p 8080:80 mixpanel-nginx`
   4. Visit `http://localhost:8080`
   
   You should see: 
   ![image](https://user-images.githubusercontent.com/556882/107558458-6ba75580-6ba0-11eb-89de-749b895b7399.png)

#### Option 3: Add locations to your existing Nginx config
    If you're application servers are already running nginx, you can copy and paste the locations from the [nginx.conf](https://github.com/mixpanel/tracking-proxy/blob/master/nginx.conf) file in this repo.


## Using the proxy server with the Mixpanel JS library
   
#### 1. Load the Mixpanel JS library from the proxy domain

   _Note: This is only required if you are NOT bundling the Mixpanel JS library into your source code (via our npm module or otherwise)._
   
   Add the following variable in your code right before the Mixpanel JS snippet:

   ```js
   // Set this variable
   const MIXPANEL_CUSTOM_LIB_URL = "https://<YOUR_PROXY_DOMAIN>/lib.min.js";
   
   // Before this snippet
   (function(f,b){if(!b.__SV){var e,g,i,h;window.mixpanel=b;b._i=[];b.init=function(e,f,c){function g(a,d){var b=d.split(".");2==b.length&&(a=a[b[0]],d=b[1]);a[d]=function(){a.push([d].concat(Array.prototype.slice.call(arguments,0)))}}var a=b;"undefined"!==typeof c?a=b[c]=[]:c="mixpanel";a.people=a.people||[];a.toString=function(a){var d="mixpanel";"mixpanel"!==c&&(d+="."+c);a||(d+=" (stub)");return d};a.people.toString=function(){return a.toString(1)+".people (stub)"};i="disable time_event track track_pageview track_links track_forms track_with_groups add_group set_group remove_group register register_once alias unregister identify name_tag set_config reset opt_in_tracking opt_out_tracking has_opted_in_tracking has_opted_out_tracking clear_opt_in_out_tracking start_batch_senders people.set people.set_once people.unset people.increment people.append people.union people.track_charge people.clear_charges people.delete_user people.remove".split(" ");
for(h=0;h<i.length;h++)g(a,i[h]);var j="set set_once union unset remove delete".split(" ");a.get_group=function(){function b(c){d[c]=function(){call2_args=arguments;call2=[c].concat(Array.prototype.slice.call(call2_args,0));a.push([e,call2])}}for(var d={},e=["get_group"].concat(Array.prototype.slice.call(arguments,0)),c=0;c<j.length;c++)b(j[c]);return d};b._i.push([e,f,c])};b.__SV=1.2;e=f.createElement("script");e.type="text/javascript";e.async=!0;e.src="undefined"!==typeof MIXPANEL_CUSTOM_LIB_URL?
MIXPANEL_CUSTOM_LIB_URL:"file:"===f.location.protocol&&"//cdn.mxpnl.com/libs/mixpanel-2-latest.min.js".match(/^\/\//)?"https://cdn.mxpnl.com/libs/mixpanel-2-latest.min.js":"//cdn.mxpnl.com/libs/mixpanel-2-latest.min.js";g=f.getElementsByTagName("script")[0];g.parentNode.insertBefore(e,g)}})(document,window.mixpanel||[]);
   ```
   
#### 2. Configure the Mixpanel JS client to make requests to your proxy domain
   
   Take the domain that exposes your proxy server and specify it as the value of the `api_host` config option when you initialize the Mixpanel JS SDK.

   ```js
   mixpanel.init("<YOUR_PROJECT_TOKEN>", {api_host: "https://<YOUR_PROXY_DOMAIN>"})
   ```
   
## Full Example
```index.html
<html>
    <head>
        <title>Mixpanel Tracking Proxy Demo</title>
        <script type="text/javascript">
            /**
             * Configuration Variables - CHANGE THESE!
             */
            const MIXPANEL_PROJECT_TOKEN = YOUR_MIXPANEL_PROJECT_TOKEN; // e.g. "67e8bfdec29d84ab2d36ae18c57b8535"
            const MIXPANEL_PROXY_DOMAIN = YOUR_PROXY_DOMAIN; // e.g. "https://proxy-eoca2pin3q-uc.a.run.app"
            
            /**
             * Set the MIXPANEL_CUSTOM_LIB_URL - No need to change this
             */
            const MIXPANEL_CUSTOM_LIB_URL = MIXPANEL_PROXY_DOMAIN + "/lib.min.js";
            
            /**
             * Load the Mixpanel JS library asyncronously via the js snippet
             */
            (function(f,b){if(!b.__SV){var e,g,i,h;window.mixpanel=b;b._i=[];b.init=function(e,f,c){function g(a,d){var b=d.split(".");2==b.length&&(a=a[b[0]],d=b[1]);a[d]=function(){a.push([d].concat(Array.prototype.slice.call(arguments,0)))}}var a=b;"undefined"!==typeof c?a=b[c]=[]:c="mixpanel";a.people=a.people||[];a.toString=function(a){var d="mixpanel";"mixpanel"!==c&&(d+="."+c);a||(d+=" (stub)");return d};a.people.toString=function(){return a.toString(1)+".people (stub)"};i="disable time_event track track_pageview track_links track_forms track_with_groups add_group set_group remove_group register register_once alias unregister identify name_tag set_config reset opt_in_tracking opt_out_tracking has_opted_in_tracking has_opted_out_tracking clear_opt_in_out_tracking start_batch_senders people.set people.set_once people.unset people.increment people.append people.union people.track_charge people.clear_charges people.delete_user people.remove".split(" ");
for(h=0;h<i.length;h++)g(a,i[h]);var j="set set_once union unset remove delete".split(" ");a.get_group=function(){function b(c){d[c]=function(){call2_args=arguments;call2=[c].concat(Array.prototype.slice.call(call2_args,0));a.push([e,call2])}}for(var d={},e=["get_group"].concat(Array.prototype.slice.call(arguments,0)),c=0;c<j.length;c++)b(j[c]);return d};b._i.push([e,f,c])};b.__SV=1.2;e=f.createElement("script");e.type="text/javascript";e.async=!0;e.src="undefined"!==typeof MIXPANEL_CUSTOM_LIB_URL?
MIXPANEL_CUSTOM_LIB_URL:"file:"===f.location.protocol&&"//cdn.mxpnl.com/libs/mixpanel-2-latest.min.js".match(/^\/\//)?"https://cdn.mxpnl.com/libs/mixpanel-2-latest.min.js":"//cdn.mxpnl.com/libs/mixpanel-2-latest.min.js";g=f.getElementsByTagName("script")[0];g.parentNode.insertBefore(e,g)}})(document,window.mixpanel||[]);
            
            /**
             * Initialize a Mixpanel instance using your project token and proxy domain
             */
            mixpanel.init(MIXPANEL_PROJECT_TOKEN, {debug: true, api_host: MIXPANEL_PROXY_DOMAIN});
            
            /**
             * Track an event when the page is loaded
             */
            mixpanel.track("[Proxy Demo] Page loaded");
        </script>
    </head>
    <body>
        <button onclick="mixpanel.track('[Proxy Demo] Button clicked')">Track event</button>
    </body>
</html>
```