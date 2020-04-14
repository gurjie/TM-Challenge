This script will pull back domains related to a given domain. It will then attempt to resolve an IP for any of the domains, and then associate any IPs with a given domain. With each of the IPs, we then query to censys.io IPv4 api to discover any properties, protocls, and services running on the target IP.

The script has has 3 main components:

1) Interact with the Censys API in order to search certificates with the names related to a domain given as input, and thereafter pull back domain names related to thoughtmachine

2) An optional step, and applied if less than 20 domains were pulled back in step 1; we form a google search query composed of our domain, but negating any domains pulled back in step 1. This is so that we can search for any related domains fetched by google but not pulled back from the Censys search in step 1

3) query the censys ipv4 api, and pull back attributes based on censys searches for the IP

This is very extensible, and it's exciting to know that the IPs retrieved can be used as input to other APIs e.g. AbuseIPDB or Virustotal, since any RESTful response pulled back from their APIs, ingested, and simply attributed to an IpObject.

-----------------------------------------------------------------------------------

Prerequisites:
	- Install Docker + ensure the Docker daemon is running on your machine; https://docs.docker.com/get-docker/
	- Can verify it's running by typing 'docker --version' in cmd prompt  
	- A Censys.io API Key and Secret; Create an account and go to 'My Account' --> API

-----------------------------------------------------------------------------------

Usage:
	1) Open a command prompt with elevated privelages and cd into the directory containing the dockerfile and script

	2) Open api_secrets and replace the default CENSYS_API_KEY and CENSYS_SECRET with your own tokens. These are then passed as env variables to the Docker image	

	3) If it's your first time using the script, (on either windows or linux) run:
	
		 docker build -t intel . && docker run --env-file api_secrets.list --rm intel thoughtmachine.net -ng
	
	
	- Every usage thereafter you can simply run: 
	
		docker run --env-file api_secrets.list --rm intel thoughtmachine.net -ng


	The basic syntax is as follows:
		
		docker run --env-file [api_secrets.list] --rm intel [domain.to.analyse] [-ng]

	[api_secrets.list]	= Secrets file containing Censys API Authentication tokens, only need to change the values in the file. Passed as env vars to docker.
	[domain.to.analyse]	= The domain the script is going to pull back subdomains and attributes of any resolved IPs for
	[-ng]			= The No Google flag, to omit google search as a mechanism to discover subdomains

Output: 
	- JSON showing all domains ('domain' within json) found, any of their associated IPs given as a list ('ips' within json), and then Censys IPv4 output for each IP related to the domain. 
	
	- The following are the fields fetched from the IPv4 lookup and present within the json if fetched. Just general information and basic services:
		'tags',        
		'ports',
		'protocols',
		'80.http.get.headers.server',
		'80.http.get.metadata.product',
		'80.http.get.title',
	        '443.https.get.title',
		'25.smtp.starttls.banner',
		'80.http.get.server',	
		'21.ftp.banner.banner',
		'22.ssh.v2.banner.raw'

------------------------------------------------------------------------------------

Tradeoffs:
	- We limit the results pulled back from google to 100, and also only interact with google if the Censys step discovered less than 20 domains
	- Enhanced features v Complexity; We're only pulling back a default set of IPv4 fields from the Censys IPv4 fields. Would be really nice to have a set of command line options that load in a predefined set of fields, depending on what information you're interested in finding out about an server. Very achievable.
	- JSON output is dumped straight to command line instead of an output file, in order to save complexity of mounting a docker volume when running
	- Speed of google searches....We're limited to how many google searches we can do in short bursts of time in order to not trigger a google Captcha

Improvements / further work:
	- *Reverse DNS lookups a subnet of IPs fetched from IPv4 api, to discover related domains pulled back from the reverse lookup* - would make tool incredibly powerful; time constraints unfortunately
	- Use DNS Zone transfer records to perform full domain discovery on servers that we pull back an IP for from initial 
	- Need to do a regex check to determine if the hostname entered is in a valid format for a hostname
	- We could give the user the option to ignore any IPs resolved, so they can omit IPs given by ISP DNS Hijacking when nxdomain given
	- Configurable search limits for scraping domains on google is a bonus
	- Configurable fields to pull back from Censys IPv4 would be nice 
	- IPv4 will look up IPs that may have already been search, quick fix by maintaining a list of IPs already searched. Would act as a cache.

takeaways: 
	- Loads of scope for improvement as noted above; hoping to work a bit on this in my own time for the futur
	- Learnt a huge amount around Docker, Python, and API interaction during this process
	- Making your own tool seems like the best way to learn

------------------------------------------------------------------------------------

- From running the tool against thoughtmachine.net, gathered that thoughtmachine.net itself is deployed on an AWS cloudfront. 
- It's possible that thoughtmachine's running a gsuite deployment internally, due to the fact that the server behind intranet.thoughtmachine.net shows as ghs server. Or perhaps domain was setup under google domains, according to research...
- Nothing interesting apart from services over 80 and 443 which is really standard, and there doesn't seem to be too much information disclosed. Maybe more in depth subdomain finding techniques would have revealed more.

Output from 'thoughtmachine.net' as the target domain....

{"domain": "www.prophecy.thoughtmachine.net", "ips": []}
{"domain": "k8s-proto.thoughtmachine.net", "ips": []}
{"domain": "www.prophecy-dev.thoughtmachine.net", "ips": []}
{"domain": "www.phabricator.thoughtmachine.net", "ips": []}
{"domain": "prophecy.thoughtmachine.net", "ips": []}

{"domain": "intranet.thoughtmachine.net", "ips": [{"ip": "172.217.169.83", "data": [{"tags": ["http"], "80.http.get.title": "Error 404 (Not Found)!!1", "80.http.get.headers.server": "ghs", "80.http.get.metadata.product": "Hosted Site", "ports": [80], "protocols": ["80/http"]}]}]}

{"domain": "prophecy-dev.thoughtmachine.net", "ips": []}
{"domain": "phabricator.thoughtmachine.net", "ips": []}

{"domain": "thoughtmachine.net", "ips": [{"ip": "13.224.239.49", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.64", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.81", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.22", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}]}

{"domain": "doge.thoughtmachine.net", "ips": [{"ip": "212.36.160.18", "data": [{"tags": ["http", "https"], "ports": [443], "protocols": ["443/https"], "443.https.get.title": "Forbidden"}]}]}

{"domain": "www.thoughtmachine.net", "ips": [{"ip": "13.224.239.64", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.81", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.49", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.22", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}]}

{"domain": "www.thoughtmachine.net", "ips": [{"ip": "13.224.239.64", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.81", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.49", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}, {"ip": "13.224.239.22", "data": [{"tags": ["http"], "80.http.get.title": "ERROR: The request could not be satisfied", "80.http.get.headers.server": "CloudFront", "80.http.get.metadata.product": "CloudFront", "ports": [80], "protocols": ["80/http"]}]}]}
