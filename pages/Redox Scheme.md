## Overview
	- resources in redox os are accessed using the *scheme-rooted path*
	- /scheme/<scheme_name>/<resource_name>
	- <scheme_name> denotes the type of resource as well as the scheme provides
	- ### Scheme Provider
		- it is a process in user space that listens for requests on that scheme
		- when the provider is started, it register to the kernel by opening the scheme in the *Root Scheme*
		- provides file-like operations. How the operation is implemented is up to the provider
	- **Root Scheme**
		- denoted by ":"
		- container for all other scheme
	- **Event Scheme**
		- allows scheme provider or other client programs to listen for events on a file descriptor
	- ## Registering New Scheme
		- New schemes are registered in the kernel by opening new file in root scheme
		- `File::Open(":new_scheme")`
		- The returns a message passing channel between the kernel and the scheme provider in the form of a file descriptor
		- The file operations from the clients are converted to message packets which can be read and responded to by the scheme provider using the file descriptor
	- ## Providing a Scheme
		- Create the scheme: `File::Open(":myscheme")`. This returns a file descriptor
		  logseq.order-list-type:: number
		- Open a file descriptor for each resource that is required to provide service
		  logseq.order-list-type:: number
		- open file descriptor for timer if necessary
		  logseq.order-list-type:: number
		- open file descriptor for event scheme: `File::Open(/scheme/event")`
		  logseq.order-list-type:: number
		- move to null namespace: `setrens(0, 0)`
		  logseq.order-list-type:: number
		- Write to event file descriptor to register each file descriptor the provider will listen to
		  logseq.order-list-type:: number
		- LOOP
		  logseq.order-list-type:: number
			- block waiting for an event to read. Simple scheme provider can just do a blocking read on the scheme file descriptor
			  logseq.order-list-type:: number
			- read the event to determine the type: timer, resource event, scheme event
			  logseq.order-list-type:: number
			- if resource event, perform the necessary action
			  logseq.order-list-type:: number
			- if scheme event, read the message packet from file descriptor and call the `handler`. The packet will indicate if its an `open`, `read`, `write` etc
			  logseq.order-list-type:: number
				- an `open` will include the name of the item to be opened which can be parsed by the scheme provider to determine the exact resource. The scheme will allocate a handle for the resource with numbered descriptor. The descriptor is used by the scheme to look up the `handle` data structure
				  logseq.order-list-type:: number
				- a `read`, `write` etc will be handled by the scheme using `handle` number
				  logseq.order-list-type:: number
			- after all requests are handled, loop through every handle to see if any queued request are complete. A response is sent for each completed request
			  logseq.order-list-type:: number
			- Set timer to handle device timeout if appropriate
			  logseq.order-list-type:: number
	-
- ## How its implemented in Kernel?
- ## How other services(like redoxfs) implement it?
-
-
- #redox #major-project