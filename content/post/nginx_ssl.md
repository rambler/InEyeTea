+++
date = "2015-02-09T08:52:10-06:00"
draft = true
title = "NginX/SSL/Node.js"

+++

# Using NginX to do SSL for Node.js apps

Some notes from my second pass "SSLifying" a Node.js application. By this I mean simply to create an SSL endpoint for your application. In our case, the same application actually listens on both ports 80 and 443 and redirects to 443 if a connection is made on 80. We have been doing that but the additional constraint is that we must place the endpoint directly on the host running the application (all traffic must be encrypted).

## The easy way that doesn't meet our requirements

Our starting point here is a Node.js app that has been wrapped via an ELB. If you're working in AWS and your goal is simply to provide an SSL endpoint for your application, that's one easy way. To set this up, create an ELB for your application with a single associated instance, and make a listener that uses HTTPS as a protocol on the outside and HTTP on the inside. Amazon will happily take your certificate and private key and authenticate for you, then take off the SSL wrapper and fetch your traffic in plaintext.

Yay!

Unless you have regulatory requirements that data in transit be encrypted. Then you have to explicitly configure TCP rather than HTTPS on both ends and have something on your node that will do the SSL termination.

There are several ways to do this:

* NginX
* Apache
* Node.JS
* Many more...

Anyway, we decided to use NginX for SSL termination and forward the traffic upstream.

## Configuring NginX

Well, we're using Chef, so I'm going to include some Chef code snippets. First, NginX has the ability to include a directory of config snippets (`conf.d`) so we'll use that:

```ruby
directory '/etc/nginx/conf.d' do
	owner 'root'
	group 'root'
	action :create
	recursive true
end

cookbook_file '/etc/nginx/conf.d/myapp.conf' do
	mode 0644
end
```

Here is the relevant content of that file:

```nginx
upstream my_upstream {
  	server 127.0.0.1:8081;
  	keepalive 64;
}

server {

	listen   443;

	ssl    on;
	ssl_certificate     /etc/nginx/certs/bundle.crt;
	ssl_certificate_key /etc/nginx/certs/my.com.pem;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

	...
	}

	location / {
	  	proxy_redirect off;
	  	proxy_set_header	X-Real-IP $remote_addr;
	  	proxy_set_header   	X-Forwarded-For $proxy_add_x_forwarded_for;
	  	proxy_set_header   	Host $http_host;
	  	proxy_set_header   	X-NginX-Proxy true;
	  	proxy_set_header   	Connection "";
	  	proxy_http_version 	1.1;
	  	proxy_cache 		one;
	  	proxy_cache_key 	my$request_uri$scheme;
	  	proxy_pass         	http://my_upstream;
	}
}
```

Once this file is in place, we are free to start the NginX service (through the standard Chef cookbook, no special handling required).

## Configuring your application

Now, all you have to do is make your app listen on the upstream port, here 8081.


