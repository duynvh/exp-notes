# Introduction to JS

> Source: https://javascript.info/intro

- `JS make web pages alive`
- Don't need preparation or compilation to run.
- Need a special program call `JS engine` to run.
	- Engine read(parse) the scripts
	- It converts("compiles") scripts to machine code
	- Then machine code run

- JS doesn't provide low level access to memory or CPU
- JS's capabilities depends on the environment it's running in.
	- In browser, JS can do anything with webpage, interaction with user, webserver. 
	- With Nodejs, it can access files, perform network requests, ...
	
- JS in browser **CAN'T DO**:
	- Read/write arbitrary file on the hard disk, ...
	- Different tabs/windows do not know each other
	- JS can communicate over the net to the server where the current page from. But its ability to receive from other sites/domains is crippled. Though possible, it requires explicit agreement(expressed in HTTP header) from the other site.

- JS can be used in browser, server, mobile app.