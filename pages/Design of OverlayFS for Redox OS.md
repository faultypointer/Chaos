- So I absolutely have no idea how to even start this. So let me first list the resources I have
- ## Resources
	- Linux overlayfs documentation
		- https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html
	- Linux overlayfs source code
		- I have cloned that locally (and i also have looked at it but didn't have any idea what was going on)
	- Comments from james matlik on redox os issue about overlayfs
		- https://gitlab.redox-os.org/redox-os/redox/-/issues/1568
	- And thats about it i think
	- there are other resources like redox os book etc but idk how helpful they will be in design
	- The section about Scheme and Resources will be plenty helpful but I have read on that and have notes [[Redox Scheme]]
- ## Design: Version -1
	- I am pretty sure the overlay functionality would be provided through a scheme
	- I know there needs to be some sort of state to keep track of the layers
	- lets keep things simple, only one upper layer and one lower layer
	- and my mind goes blank
	- if it is actually through the scheme, then the scheme provides services through file like operations in which case for each operation I only have to think of how to implement that operation and what should the state store to make that implementation of operation possible
	- for example the open operation would normally return a file descriptor for the file whose path is passed to the open. in our case, the "fd" we return would be something that we need to store and map to maybe the file in the lower layer or maybe in the upper layer or maybe its a copyup file and all other options
	- the read, write and all other operations could then help in making the rough draft of design-0
	- so what i need to do is to first read through its code and docs to understand how each functionality is handled in linux overlayfs, then read through redox code/docs and maybe james's comment to have a rough guess of how to do that in redox
- #major-project #os #redox
-