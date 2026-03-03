To migrate a standard nextcloud setup to Podman/Docker, you need to create the container setup and copy data from the old system to the new. 

I am going to start with a new Alma 10 VM for my setup. 

The VM will be configured using Ansible. 

First, we need to make a user on the new server and give it passwordless sudo access and copy ssh keys over from our control server. 

