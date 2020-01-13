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

