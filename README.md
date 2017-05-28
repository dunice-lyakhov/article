Website Optimization. In pursuit of speed.

The modern world of the Web is rapidly evolving and the demands of users are developing along with it. Many people try to improve the design, add new functionality or display more content on web pages. This is all very important, but site visitors tend to take care more about the speed of downloading websites. In addition, the page load time becomes a more important factor when it comes to ranking in the search engines. Therefore, the page load time is an important parameter of any site.

According to analyst company *Kiss Metrics*, about half of Internet users are convinced that the optimal time to load a site page is 2 seconds. And they are ready to leave the site, if in 3 seconds it still has not booted. 79% of Internet users who had problems downloading a site, subsequently refuse to re-visit. And about 44% of them will share their negative experiences with their friends and acquaintances.

Since 2010, Google takes into account the speed of downloading a site in the search rankings. Since then, the speed at which someone can view content from search results is a factor that influences it.

In this connection, the problem of optimization and acceleration of sites arises. We will tell you how to solve it with an example of our project using the following **_Nginx_** / **_uWSGI_** / **_Django_** / **_PostgreSQL_** / **_jQuery_** / **_Gulp_** technology stack.

And before proceeding to optimization, you should determine parameters by which we estimate the speed of a site. One of such parameters is [TTFB (Time to First Byte)](https://en.wikipedia.org/wiki/Time_To_First_Byte). TTFB is affected by almost everything: network problems and delays, incoming traffic volume, web server settings, volume and content optimizations. In a well-functioning system, the TTFB value can be less than 100 milliseconds (ms) for static content and 200-500 for dynamic content. For our site TTFB averaged 3 seconds.

![Imgur](http://i.imgur.com/hzjUCeP.png)

First we decided to optimize Nginx. And the first step was to update it to the latest stable version.

```console
$ sudo apt-get autoremove --purge nginx nginx-common
$ sudo touch /etc/apt/sources.list.d/nginx.list
$ sudo printf "\ndeb http://nginx.org/packages/mainline/ubuntu/ trusty nginx" >> /etc/apt/sources.list.d/nginx.list
$ sudo printf "\ndeb-src http://nginx.org/packages/mainline/ubuntu/ trusty nginx" >> /etc/apt/sources.list.d/nginx.list
$ wget -q -O- http://nginx.org/keys/nginx_signing.key | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install nginx
```

After that, we enabled and configured content compression with [gzip](http://nginx.org/en/docs/http/ngx_http_gzip_module.html).

```apacheconf
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 5;
gzip_min_length 256;
gzip_http_version 1.1;
gzip_types
       text/plain
       text/css
       application/json
       application/javascript
       application/x-javascript
       text/xml
       application/xml
       application/atom+xml
       application/xml+rss
       font/opentype
       image/svg+xml
       image/x-icon
       image/gif
       image/jpeg
       image/png
       text/javascript;
```

These two simple steps have already significantly reduced the TTFB. In order to further increase the efficiency of Nginx, we have configured its parameters for our servers and features of our project.

```apacheconf
sendfile on;
tcp_nopush on;
tcp_nodelay on;

types_hash_max_size 2048;
client_body_buffer_size 10K;
client_header_buffer_size 1k;
client_max_body_size 8m;
large_client_header_buffers 4 8k;

client_body_timeout 10;
client_header_timeout 10;
reset_timedout_connection on;
keepalive_timeout 65;
keepalive_requests 100;
send_timeout 2;

open_file_cache max=200000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;

ssl_session_cache   shared:SSL:10m;
ssl_session_timeout 5m;
ssl_prefer_server_ciphers on;
ssl_stapling on;
ssl_buffer_size 4k;
resolver 8.8.8.8;

access_log off;
error_log /var/log/nginx/error.log crit;
```

The next point was the optimization of [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/). For this, we have configured the query caching for it. All this allowed to reduce the average TTFB by 2 seconds.

![Imgur](http://i.imgur.com/XktkuCN.png)

To further increase the speed of the site, it was decided to use caching. Django comes with a reliable and easy-to-configure caching system. All you need to do to configure the cache in Django is to specify where the cached data should reside - in the database, on the file system or directly in memory.

```python
CACHES = {
   'default': {
       'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
       'LOCATION': '127.0.0.1:11211'
   },
   'cache_machine': {
       'BACKEND': 'caching.backends.memcached.MemcachedCache',
       'LOCATION': '127.0.0.1:11211',
       'KEY_PREFIX': 'weee:'
   },
}
```

In our project, we used the fastest and most effective cache type - [Memcached](https://www.memcached.org/). Memcached stores the data directly in the RAM. Also it is used by such sites as Facebook and Wikipedia. The use of the cache reduced the average TTFB to 300-600 ms.

![Imgur](http://i.imgur.com/QRgJAvM.png)

So we made steps with the greatest impact in terms of reducing TTFB. What next? Can we further reduce the processing time of requests? Yes, we can! And for this we need to optimize the site code.

Before you begin to optimize the code, you need to find out the "bottlenecks". To do this, you need to conduct profiling and collect statistics. After you evaluate the processing time of queries, you will see which pages or functions need optimization. We used the [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/stable/) to perform this task.

For our project, we optimized some search algorithms and templates for highly loaded pages, as well as queries to the database and the base class of *Django Paginator*. This allowed us to reduce the average TTFB to 200-450 ms.

![Imgur](http://i.imgur.com/HJBLC4V.png)

After that we worked on accelerating page rendering on the client side. For this we used [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) and delayed loading of images, and also optimized the styles, highlighting their critical part.

All optimization work for our project took about 150 hours. But all these efforts greatly accelerated our site and made our customers happy.

Best regards,

**Nikita Lyakhov**,
Software developer Dunice
nikita.lyakhov@dunice.ru

