
- [What is a distributed system](#what-is-a-distributed-system)
- [There are 2 main models of distributed computed](#there-are-2-main-models-of-distributed-computed)
- [How to solve resource location (4 solutions)](#how-to-solve-resource-location-4-solutions)
	- [Requirements for resource location](#requirements-for-resource-location)
	- [What is a namespace?](#what-is-a-namespace)
- [3 architecture design consideration for the clien/server model](#3-architecture-design-consideration-for-the-clienserver-model)
- [Division of Labour](#division-of-labour)
	- [3 factors to consider when deciding how to distribute functionality *(in order of importance)*](#3-factors-to-consider-when-deciding-how-to-distribute-functionality-in-order-of-importance)
- [Python](#python)
- [HTTP (hyper text transfer protocol)](#http-hyper-text-transfer-protocol)
- [HTTP Messages](#http-messages)
- [The post method](#the-post-method)
- [Server side processing:](#server-side-processing)
- [Get version](#get-version)
- [State Management](#state-management)
	- [Cookies](#cookies)
	- [Cookie functionality](#cookie-functionality)
		- [requests:](#requests)
		- [CGI (common gateway interface):](#cgi-common-gateway-interface)
		- [session data](#session-data)


### What is a distributed system
A distributed system is a **program** that consists of multiple parts **running on more than 1 computer** interconnected via a network

### There are 2 main models of distributed computed
1. client-server model
   * In this model we have **servers** that deliver some service to **client** computers
2. peer-to-peer systems
   * System are both servers and clients and will perform both rolls at the same time




### How to solve resource location (4 solutions)
Getting the exact location of a server to access a service can be a tricky business.  Say you want to go to google, google has many servers and are constantly changing, we need a way to easily locate it .

1. Hard-code machine address
* lacks flexiblity. Tedius/not practicle to change when server changes
2. Configuration files containing machine addresses
* Not pratical for large and external project.  Server operators must send its address in a config file every time a new server is added
3. Send a broadcast message with addresses
* Every system hearing the message has the message
4. Use a lookup service
* Assign a global, uniqueand human understandable name to each resource
* Usign a Domain Name Service (DNS), get the location of the server from the name



  
 #### Requirements for resource location
 1. location tranparency - We need to able to see the location of the resource (server)
	* The name of the resource dosen't indicate our resource location
		* That way we can makes the resources movable, we can reorganise the software on the fly witout changing a whole wack of settings on each connecting machine
	*	e.g **Network file system(NFS)**, allows you to access your inix home directory regardless of the client machine
		*	This means files are stored on a server and clients can access these files without even knowing there's a server
	*	Bad implementation example Windows File sharing
		*	Explicity map a file system to a drive name
		*	This means if we want to access a server file, we must map that server to a drive letter and user must go to the drive to access files
	* Even in NFS at some level the real location of the server must be known, NFS requires sys admin to handle "mounting" of file systems, (aka entering the location of the server into the system, then the system will hide the location)
		* Location of servers are entered in the config files of the systems
2. global uniqueness 	 - there can only be one google.com
	* if we dont have uniqueness we can't use names as keys 
	* 2 ways to make sure we have uniquess
		1. Use a centralised server
			* 1 system that assigns name for the entire Internet.
			* Problem: prone to failure. Single point of failure is bad. If that one thing fails to whole system goes down. 
			* Problem : performance, bottlenecks, all traffic goes to the one system. 
			* Problem : scalibility, hard to scale one system because sometimes system are tested with a smaler audiance then expected
		2. a hierarchical name system
		
			* We'll provide uniqueness w/ local control
			* DNS (domain name system) - U of M has local control over all name ending in umanitoba.ca - they can subdivide their domain such as cs.umanitoba.ca to divide local control to an even smaller subgroup
3. Ability to access names from every possible location with good performance    
> We need to find the ip of google.com fast anywhere from the world

* DNS lookups are done hierarchically
	* Each leavel of a domain has a server for lookups.  
		* These servers will refer to other servers to resolve name 
	* Start with root name server (.com, .ca, ...) that point to server for a specific top level
			* e.g cs.umanitoba.ca will first go to the ca server then refer to umanitoba.ca server which will then refer to the cs.umanitoba.ca server
* DNS servers have main servers, replica server (copies), and caching server
* *note* DNS gets us from a name to an IP address. **We need** to ba able to go from the IP address to a computer. 
* IP gets us to the LAN, we need to use a MAC address of the ethernet card to ID computer 
4. Protocol identification need to get to the software    
*Once we have the IP from DNS we need to get to the software on the server*    
	* Done with protocol ports
	* Software  listens on the port by asking th OS to **bind** it to the port 
	* There are well known ports (0-1023) & general use ports (1024-65535)
		* We can do whatever we want on generl use ports but well known ports are typically reserves to certain task but we could use it for whatever we want technically
	* we can't use ports 0-1023 unless we have root level access
		* http is on port 80
		* ssh is on port 22
		
#### What is a namespace?
A namespace is a collection of rules for constructing names.

*Some examples*
- social insurance # (xxx-xxx-xxx)
- IP address (xxx.xxx.xxx.xx)
- domain name (www.xxx.xx)
- URL's (protocol://user:pw@hostname:port/URI (Uniform Resource Identifier is a Unix path to a resource))


### 3 architecture design consideration for the clien/server model
1. State management - stateful vs stateless    
*(state means to have memory)*
* State enable complex functionality by maintaining info about its client
	* ex.// keeping track of what is added to a cart
* Problem: what happens when there is an error
	* We need to implement a recovery system so the user can re-start its operation after failure
* Stateless server dosen't maintain any client info and therefore much simpler in design
	* if there is a failure, no data loss theefore nothing bad happens
* Working problem: how to chose the design of a distributed file system
	* requirement: must ensure non-concurrent file access
		* we need state for this: lock for each client
			* but if error occurs we have invalid lock info (stateless would prevent these annoying errors)
		* in reality we use a hybrid approach - half stateless half stateful
			* introduce some recovery that leads to soe stateless behaviour
2. Concurrency
	* Using a multi-threaded server just make sense
		* this concept is explored in 3430
3. farming (server farms)
* When a server gets overloaded we can't just get a bgger server
* instead use a bunch of smaller, cheaper machines
	* can be locally or remotly distributed
* gives us *both* improved performance and better availability
	* if 1 server goes down, we still have N-1
	
* 2 design issues with farming
	1. each server must contain a copy of what we re serving
		* Replication issue
		* when data changes, we must chage all copies
			* This is a concsistency problem
		* must allow for scalibility
			* as # of servers grows, replication cost must stay "the-same" (we can't be inificient and use all our processing power to communicate and share data between server for replication)
	2. Client shouldn't know there are multiple servers but we should maintain client transparency 
		* 2 general solutions  
			1. we could implement a round robin DNS, every request get a turn to access the DNS.
				* User only sees one DNS
				* Domain name is mapped to multiple IP address
				* DNS chooses which IP address to redirect too
				* Each IP is one server and does the whole work
				* This is before firewall
			2.  proxy redirect (load balancing) create a server that request comes in and we forward the request to one of the server
				 * User only sees one DNS
				 * Use normal DNS, DNS goes to one IP
				 * DNS goes to one server then the server chose how to delegate task between internal servers
				 * This is after firewall
				 * Our own server will redirect work load internally
				 

### Division of Labour
> How does the client adapt to server design?

2 "school of thought"
1. Thick clients
	* Offload work from server & put processing in the clients
	* we require greater capabilities from the clients.
		* e.g we have native apps
2. Thin clients
	* Limit clients to only display functions.  All the work is done on server
	* reduces client cost & complexity
		* Require better server, more server cost
* Which one shuld we chose ?
	*  e.g point of sale terminal
		* Thin client, we want the process to be handled on site in the bank for security
	* Other cases are harder e.g. remote file system
		*  do we transfer whole files? Or do I transfer specific file blocks (aka netflix streaming) - it's somewhere in between?
		* No "right" answer when choosing between thick and thin
			* The best answer will probably invovle a bit of both.
			* We want to minimize client processing power while having useful functionalities
#### 3 factors to consider when deciding how to distribute functionality *(in order of importance)*
1. When there's a resource at a machine, processing should be done at that machine
	* e.g. db joins
2. Given a choice of dividing up processing in a # of ways, choose the one that minimises communication between systems
	* Try Sending the lease amount of information because it improves performance and reliability
3. Choose distribution that balances the computational workload
	* Given our capabilities, we should try to spread out the cost as much as we can between machines
	

			

### Python
* Scripting language - weakly typed (not bound to a specific programming language)
* easy I/O
* arrays are lists - we don't use arrays in our class, we'll mostly use list.  
	* Stacks and queues ae built-in to list in python (fundamental type)
* Hash tables are built-in python (fundamental type)
	* Key-value coding

Every python script will start with the following
``` python
#!/usr/bin/python
import sys

# read a file
myFile = open("empdata.txt","r")
# or
myFile = sys.stdin
```
with any file you can :
a) `input=f.readlines()`[reads whole file]
b)`line=f.readline()` [1 line]
c) `for line in f:` [reads a line from f and puts it in the line variable] iterates until EOF
can do:
``` python
with open("x.txt","r") as f:
	for line in f:
# auto close at the end of with block 
# (after indetation)
```
``` python
# assume we have a line read in 
totalEmps = totalEmps + 1
# There's no ++ operator in python

name,age,dept, salary = line.split(":") #split default to whitespace, no need to declare (":")

if age >55:
	over55 = over55 +1
	if salary > 40000:
		termList[name]=salary
			# a dictionary (hash table)
```

``` python
# initialise a dictionary:
termlist = {}
termlist = {'key1':444,'key2':222}
# index by key: 
termList['key1'] -> 444

# outdent to end of file processing
if totalEmps > 0 and over55 > 0:
	print "sorted list of fired emps:"
	print "\nName\t\tSalary"
	for next in sorted(termList.key()):
		print "\n{0}\t\t{1}".format(next, termlist[next])
```
std list ops:
- modifying in-place:
		-	`list.sort()`
		-	`list.reverse`
- modifying by position
		- `list.insert(index,val)`
		- `list.remove(val)`, 	`list.index(val)`,`list.count(val)`
- stacks
		- `list.append(val)`, `list.push(val)`, `list.pop()`
- subroutines:
``` python
def funcName(arg1,arg2,arg3):
	return a # a value or list
``` 
- there's no main method
		- execute line-by-line
			- skip defs until called

### HTTP (hyper text transfer protocol)
* Basic functionality
	* an http client opens a connecion and sends a request to an http server
	* the server returns a response
		* Usually the content of a file
	* After delivering the response, the server closes the connection
		* Stateless protocol
* The format of request & response messages:
	* an initial line identifying our message type
	* 0 or more header lines
	* a blank line (CR or LF *line feed*)
		* A blank line is necessary to seperate the end and start of various section
	*  an optional msg body
		* a file, query data, etc.
* A Request message contains the following:
	* The initial line has 3 spaces delimited paers
		* A method name
		*  local path to the resource
			* The URI
			* Rooted at server's current working directory
	* http version
		* of the form "HTTP/x.y"
	* example of a request
```
Get /path/to/index.html HTML/1.0
# Get is the most common request but POST & HEAD (PUT & DELETE)

``` 
* A responce request contain the following:
	* initial line has the 3 parts
		* http version
		* resoinse satus code
		* reason phrase
	* example of response message:
```
HTTP/1.0 200 OK
# 200 means ok
# 404 means not found
# 500 Internal server error (a bug in your code)
```
Header lines:
* Provide information about msg body
* Encoded in "text header format"
	* Header-name : value
* http 1.0 has 16 headers
	* 1.1 has 46 header (1.1 is more common)
* *www.w3.org/Protocols for more info about headers* - Not needed for class
* Std headers
	* User-Agent - client id some with program-name/x.y.z
		* Back in the day, it was used to return different message based on what machine the client used
	* Last-modified - resource modification date
		* Used for client caching (if version changes, then browser will re-fetch resource0
		* Use HEAD to get just the header lines   

### HTTP Messages
* initial line
* headers

message body:
* in a response it's the requested resource
* in a  request, it's user entered data
* use header lines to describe the body:
	* Content type *(is a header line)*
		* Must be a "MIME" type of data
			* text/html
			* text/json
			* image/png
	* Content-length *(also a seperate header line)*
		* \# of bytes in the body
			* Includes newlines
### The post method
* Used to send data to the server
	* Body contains a block of data
* URI is for a program to process the data
	* we can use anything to write this
* Content-type of the request is:
	* application/x-www-form-urlencoded
* eg:
```
POST /path/script.py {version numeer}
Content-Type:

name=fred&fav+colour=green%26yellow # & is to sperate name=value pairs, + (or %20) is space, %26 is the char &

```     
* can use a GET to do the same thing
	* Append URL-encoded data to the URI
	* e.g: `GET /path/script.py?name=fred...`
* Why use GET?
	* Can bookmark w/ parameters
	* can modify values in-place 
	* a problem: security...
### Server side processing:
* generate *dynamic content* base of input 
* Start with **CGI (common gateway interface)**
* How does our server know wether to return a resource or run it?
	* uses the suffix
		* Server recognises either `.html`,`.cgi`,`.php`... 
			* **Our server recognises only `.cgi` files**
			* We don't want to run arbitary code hence why we limmitit what suffix our server recognises
		* We can control where the files must reside
			* **Our homework files must reside in `/{umID}/public.html/cgi-bin/`** or `/~{umID}/cgi-bin/`

*{missed start of class}*
with a post, msg body is piped to us

```
queryText = sys.stdin.readline()

fields = {}
for keyValuePair in queryText[0].split('&'):
	(key,value) = keyValuePair.split('=')
	fields[key] = value
	
# do processing


#generate a response (piped to server)

print "Content-type:text/html"
print <- blank line
print '<html>.....'
print 'hello {0}{1}' format(fields['frame'],fields['lname'])
print  '</body></html>'

```
> this is supposed to be a hello world program **but missing the beggining of lecture**

### Get version
Anything other than our msg body is put in an enviroment variable
* Everything other then the msg body is put in an enviroment variable
```
if envirom.haskey('QUERY_STRING')
	queryText = envirom ['QUERY_STRING']
		# query text contain text after ? in URI
```
* The remainder of the code is the same as in POST code
* there also a REQUEST_METHOD that get populated 
	* (in GET, POST, HEAD,...)
> Tips for assignment: start with CGI not HTML, should not need to create a HTML file to support a program but HTML code will be create in the server

## State Management
* We need software above the server to manage state
* requirements:
	*  need to figure out who's who
		* When we have >1 connection, how do we know if this is the same or different client?
		* need to maintain "user sessions"
			* What is a session: an interaction where our user dosen't leave the site
		* 2 level of state
			* a single session
			* state across multiple sessions
				* need consistncy within & between sessions
					* requires some form of login 
* 2 general approaches to save state across multiple session
	* cookies 
	* session data object, which uses cookies
### Cookies
* a small file stored by the server on the client
* retrieved to determine if our client was previously connected, and what they were doing
* no constraints on what's in a cookie, 
	* good: we can store what we need
	* bad: clients can have sensitive info passed without knowledge

### Cookie functionality
* use http headers to pass cookies containing name/value pairs
	* one header line each
* a response sends a cookie as"
	* Set-Cookie: Name=Value 
		* [expires=date;]
		* [domain=......;]
			* Specifies site owning the cookie
			* defaults to connected server
			* browser sends all cookies of the domain for each request
		* [path - ....;]
			* subset of URI's where the cookie is valid
#### requests:
* browser matcher URI with all cookies
* generate 1 header line containing name/value pairs
	* Cookie: name<sub>1</sub>=value<sub>1</sub>;...;name<sub>n</sub>= value<sub>n</sub>
#### CGI (common gateway interface):
 ``` python
 if envirom.has-key('HTTP-COOKIE'):
	 for cookie in envirom['HTTP-COOKIE'].split(';'):
		 (key,value)=cookie.split('=',1)
		 cookie[key]=value
``` 
#### session data
* cookies have limitations: security & bandwith
	* *real apps needs **lots** of states*
* recall division of labour
	* keep datat where it's processed *(store session data @ server)*
	* store data using standard techniques:
		* files
		* db
		* NoSQL
* Still need user identification
	* require a login
	* use a cookie to remember who you are
		* login into, usually a db key (unique value)
* HTML5 has client-side db's 
	* can support a **single** session locally
	* sync to the server to go "across" sessions

