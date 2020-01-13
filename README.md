# Comp 3010 notes

## How to upload your edits to this page
1. Fork this project
2. In your own Fork, make your edits in README.md
3. Create a pull request
> Note: no copyrightable material will be accepted into this repo (no in-class pictures of powerpoint slides), futhermore, no discussion of assignments or gradeable material will be accepted into this document regardless if it was discussed in class or not

## How to upload your own document without making edits to this page
1. Fork this project
2. Add your own notes in a seperate document anywhere on this page
3. Create a pull request
> Note: no copyrightable material will be accepted into this repo (no in-class pictures of powerpoint slides), futhermore, no discussion of assignments or gradeable material will be accepted into this project regardless if it was discussed in class or not

# Day 0

### What is a distributed system
A distributed system is a **program** that consists of multiple parts **running on more than 1 computer** interconnected via a network

### There are 2 main models of distributed computed
1. client-server model
   * In this model we have **servers** that deliver some service to **client** computers
2. peer-to-peer systems
   * System are both servers and clients and will perform both rolls at the same time


# Day 1

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

# Day 2
#### What is a namespace?
A namespace is a collection of rules for constructing names.

*Some examples*
- social insurance # (xxx-xxx-xxx)
- IP address (xxx.xxx.xxx.xx)
- domain name (www.xxx.xx)
- URL's (protocol://user:pw@hostname:port/URI (Uniform Resource Identifier is a Unix path to a resource))
  
 ### Requirements for resource location
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
# Day 4

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
