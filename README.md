# domain-for-sale
single server platform for multiple domains that are for sale

Usage:
Use NGINX to setup server blocks for each domain:

Server {
  server_name mydomain.com;
  [...]
}
Server {
  server_name myotherdomain.com;
  [...]
}

Then run the application at your desired port, here I use port 8080.
Then set the server block for each domain to use the application port and listen to port 80, 
I recommend using an upstream.
(You don't really need https for this, it's just a waste of money.)


###
## UPSTREAMS
###
upstream domainForSale {
	server		127.0.0.1:8080 max_fails=0 fail_timeout=10s;
	keepalive	512;
}

####
## SERVERS
####

#mydomain
Server {
  server_name   mydomain.com;
  listen        80;
  
  location / {
	proxy_set_header 	Upgrade $http_upgrade;
	proxy_set_header 	Connection "upgrade";
	proxy_set_header 	X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header 	X-Real-IP $remote_addr;
	proxy_set_header 	Host $host;
	proxy_set_header 	X-NginX-Proxy true;

	proxy_next_upstream 	error timeout http_500 http_502 http_503 http_504;
	proxy_http_version 	1.1;

	proxy_pass 		http://domainForSale/;
	proxy_redirect 		off;
  }
}
Server {
  server_name   myotherdomain.com;
  listen        80;
  
  location / {
    proxy_set_header 		  Upgrade $http_upgrade;
		proxy_set_header 		  Connection "upgrade";
		proxy_set_header 		  X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header 		  X-Real-IP $remote_addr;
		proxy_set_header 		  Host $host;
     proxy_set_header 		X-NginX-Proxy true;

		proxy_next_upstream 	error timeout http_500 http_502 http_503 http_504;
		proxy_http_version 		1.1;

	  proxy_pass 				  http://domainForSale/;
    proxy_redirect 			off;
  }
}
