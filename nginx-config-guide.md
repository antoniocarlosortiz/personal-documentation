##FILENAME: /etc/nginx/sites-available/crescent_moon.conf

Included here are notes to serve as guide on how NGINX works.

```bash
upstream crescent_moon_app_server {
  # fail_timeout=0 means we always retry an upstream even if it failed
  # to return a good HTTP response (in case the Unicorn master nukes a
  # single worker for timing out).
​
  server unix:/webapps/crescent_moon/run/gunicorn.sock fail_timeout=0; #this is where the request is redirected from port 80.
}
​
server {
​
    listen   80; #will listen to requests from port 80. ex: 127:0.0.1:80
    server_name .crescent-moon.tk; #will only be considered if there are multiple server block directive lisening to 80. If nothing matches, nginx will get the first written server block or the one with a default_server.
​
    client_max_body_size 4G;
​
    access_log /webapps/crescent_moon/logs/nginx-access.log;
    error_log /webapps/crescent_moon/logs/nginx-error.log;


#nginx checks the uri of the request. if it has /static/ e.g.,  http://localhost/static/example.png, then it will look for the location block matching that.
#location matching priority is from longest to shortest prefix ex: http://localhost/static/example.png matches /static/ and / but since /static/ is longer then it will be matched first.
#alias specifies the path to be called. ex: http://localhost/static/dog/cat/example.png will call /webapps/crescent_moon/crescent_moon/static/dog/cat/example.png
#root is the same as alias but it includes the location parameter part. ex: http://localhost/media/dog/cat/example.png will call /webapps/crescent_moon/crescent_moon/media/media/dog/cat/example.png
 
    location /static/ {
        alias   /webapps/crescent_moon/crescent_moon/static/;
    }
    
    location /media/ {
        alias   /webapps/crescent_moon/crescent_moon/media/;
    }
​
    location / {
        # an HTTP header important enough to have its own Wikipedia entry:
        #   http://en.wikipedia.org/wiki/X-Forwarded-For
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
​
        # enable this if and only if you use HTTPS, this helps Rack
        # set the proper protocol for doing redirects:
        # proxy_set_header X-Forwarded-Proto https;
​
        # pass the Host: header from the client right along so redirects
        # can be set properly within the Rack application
        proxy_set_header Host $http_host;
​
        # we don't want nginx trying to do something clever with
        # redirects, we set the Host: header above already.
        proxy_redirect off;
​
        # set "proxy_buffering off" *only* for Rainbows! when doing
        # Comet/long-poll stuff.  It's also safe to set if you're
        # using only serving fast clients with Unicorn + nginx.
        # Otherwise you _want_ nginx to buffer responses to slow
        # clients, really.
        # proxy_buffering off;
​
        # Try to serve static files from nginx, no point in making an
        # *application* server like Unicorn/Rainbows! serve static files.
        if (!-f $request_filename) {
            proxy_pass http://crescent_moon_app_server; #proxy_pass redirects the request from 80 to the specified upstream.
            break;
        }
    }
​
    # Error pages
    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root /webapps/crescent_moon/crescent_moon/static/;
    }
}
