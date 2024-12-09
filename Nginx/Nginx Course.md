# Introduction
Nginx ("_engine x_") is an HTTP web server, reverse proxy, content cache, load balancer, TCP/UDP proxy server, and mail proxy server

[Documentation Link](https://nginx.org/en/docs/)

Nginx has one master process and several worker processes. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests.

The way nginx and its modules work is determined in the configuration file. By default, the configuration file is named `nginx.conf` and placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`.
![[Pasted image 20241206223953.png]]
How we think a request to a server looks like
![[Pasted image 20241206224051.png]]
How it actually is (also known as a Reverse Proxy)

- **Serving Static Content:** Nginx excels at efficiently delivering static files like HTML, CSS, and JavaScript, ensuring fast loading times and optimal performance.  
    
- **Dynamic Content:** While traditionally used for static content, Nginx can also serve dynamic content by working in conjunction with application servers like PHP-FPM, Python, or Ruby on Rails.

	![[Pasted image 20241206224214.png]]
	Nginx is also used as a Load Balancer
	
- **Load Balancing:** Nginx can distribute incoming traffic across multiple servers, improving performance, scalability, and reliability.  
    
- **Caching:** It can cache static content, reducing server load and improving response times.  
    
- **Security:** By acting as a proxy, Nginx can protect backend servers from direct attacks and provide additional security measures.  
	![[Pasted image 20241206224249.png]]
	Nginx is also used for Encryption

# Terminology

So in our Nginx server we can change its behavior using a single file i.e. configuration file

**Configuration File:** A text file where Nginx directives are defined to configure its behavior.
And configuration file consists of modules which are controlled by directives and those are of 2 types
- **Context** : - A configuration block that defines a virtual server, specifying its domain name, port number, and other settings.

- **Directives** : - A command used to configure Nginx's behavior, such as listening on a specific port, defining server blocks, or setting up location blocks.

**Worker Process:** A process responsible for handling client requests. Nginx typically runs multiple worker processes to improve performance and scalability.

**Reverse Proxy:** A server that forwards client requests to backend servers and returns the responses to the clients.

**Upstream Server:** A backend server that handles requests forwarded by Nginx

**Rate Limiting:** The technique of limiting the number of requests that a client can make within a specific time period.

**Session Persistence:** Ensuring that request is delivered to same backend server during a session

# Proxy

A Proxy server acts as an intermediary between your device and the internet. When you request a webpage or other online resource, your request is sent to the proxy server first. The proxy server then fetches the requested content and forwards it to your device.

There are different types of proxies 
### Forward Proxy
![[Pasted image 20241207135232.png]]
Forward proxy is client facing proxy, it sits between client and the internet. Its primarily serves client by forwarding their request to server  
- Often used to bypass censorship or access geo-restricted content.
- Acts on behalf of the client and hides the client's IP address
- It can be use for
	- Access control : Block access to certain websites or restrict internet usage within a company
	- Security: Scan fir any viruses and block malicious website/IP
	- Monitoring: In theory, it could log employee's web activity 
	- Caching Resources: It can be used to cache to decrease bandwidth and reduce traffic
	![[Pasted image 20241207140013.png]]


### Reverse Proxy

Reverse Proxy is a server facing proxy, it also act as a intermediary for request from clients but it primarily serves the server, sitting in front of one or more web servers
- It acts on behalf of the server and hides the server's IP address
- Often used for load balancing, caching, and security.
- Acts on behalf of the server and hides the server's IP address
- It can be used for
	- Load balancing: It is a functionality for distributing incoming request across the severs
	- It has all the functionalities of a forward proxy
	- It can filter requests for malicious activity before they reach servers and it can ensure SSL encryption are enable and detect many security threat
	![[Pasted image 20241207142233.png]]

# Why do 
## we need reverse proxy if we already have cloud load balancer, are they a replacement of reverse proxies, why we have to load balance them twice??

So i actuality we will need both cloud load balancer and reverse proxy, we will use both of them like this
![[Pasted image 20241207142657.png]]

While cloud balancer will be used as a entry point to our internal network, reverse proxy will be used ad a entry point to our server network, this is a layered approach that will make our our approach more secure and scalable

And for "why we have to load balance them twice" well reverse proxy has more granular and fine grain load balancing options and it has flexible routing rules that will result into more intelligent routing 

While Cloud load balancer distribute on certain algorithms that often operate on layer 4 ( transport layer based on ip and ports ) like whoever is least busy, Reverse proxy can do layer 7 load balancing based on session data (for session persistence ), cookies, URL paths, session data, request headers, etc 

Reverse proxy has SSL/TLS termination capabilities and it can also inspect to make more accurate load balancing decisions, it can be very useful in micro-services architecture where you might have to perform load balancing based on path or URL and you have to forward it to a specific micro-service
![[Pasted image 20241207161712.png]]

In conclusion, **In many common use cases, an AWS ALB is sufficient. However, for complex architectures, specific performance or security requirements, or to integrate with legacy systems, adding a reverse proxy like Nginx can provide significant benefits.**

# How-to

## make nginx serve static content (in local env)

First we have to install nginx to our machine we can use
```
sudo apt update
sudo apt install nginx
```

Then we need to edit config file to make it listen to port 8080 and serve our index file on it 
``` index.html
<!DOCTYPE html>
<html>
<head>
    <title>Nginx Server in EC2</title>
</head>
<body>
    <h1>This is a Nginx Server in an EC2 Instance</h1>
</body>
</html>
```

``` config file

http {
	server {
		listen 8080;
		root <path of folder that has index.html >;
	}
}

event {}

```
And that's it
#### Mime types

Now if you want to add CSS to it so you add this in header tag 
```
<link rel="stylesheet" href="style.css">
```

It will not work cause this CSS file is being passed as text 
![[Pasted image 20241209202516.png]]

So to pass it as a CSS file you have to add types block in your http 
Types block will specifies that if a file is passed with X extension it should be treated as a X file

```
http {
	types {
		text/css css;
		text/html html;
	}
	
	server {
		listen 8080;
		root <path of folder that has index.html >;
	}
}
```

But doing this for all the required file and types will be inefficient so to accommodate common type nginx has a **default Mime type**, this is a file that has common type
So we can use include directive to include mime types

```
http {
	include mime.types
	
	server {
		listen 8080;
		root <path of folder that has index.html >;
	}
}
```

## make nginx a reverse proxy

## make nginx a forward proxy

## make nginx a load balancer 


