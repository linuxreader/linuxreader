
You can find a detailed installation guide here [https://developer.cisco.com/docs/modeling-labs/#!system-requirements](https://developer.cisco.com/docs/modeling-labs/#!system-requirements "https://developer.cisco.com/docs/modeling-labs/#!system-requirements")

Sign up for CML [https://www.cisco.com/c/en/us/products/cloud-systems-management/modeling-labs/index.html](https://www.cisco.com/c/en/us/products/cloud-systems-management/modeling-labs/index.html "https://www.cisco.com/c/en/us/products/cloud-systems-management/modeling-labs/index.html")

Install VMWare Player [https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html "https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html")

Set up is pretty easy, just go through the installer. Leave all defaults.

Set up for non-commercial use

Download CML files [https://software.cisco.com/download/home/286290254/type/286290305/release/CML-Personal 2.2.3](https://software.cisco.com/download/home/286290254/type/286290305/release/CML-Personal%202.2.3 "https://software.cisco.com/download/home/286290254/type/286290305/release/CML-Personal%202.2.3")

.ova and .iso

And copy your license key from your account page [https://learningnetworkstore.cisco.com/myaccount](https://learningnetworkstore.cisco.com/myaccount "https://learningnetworkstore.cisco.com/myaccount")

Open and import the .ova

Go into your vm settings and make sure you provide enough RAM and CPU, also add the iso file here.

Also select connect at power on

Note: Intel VT or AMD-V must be enabled in your BIOS for this setup to work

I also had to use NAT to get this to work.

Power on your VM

Once loaded, your screen should look like this:

Click inside of the vm and press enter to continue

Press tab to highlight "Accept EULA" and press enter

Press enter to continue

Note the navigation commands and press enter again

Define CML and Linux root username and passwords

Press enter to use DHCP to get an IP address

Press okay

You should now be here:

Note the HTTPs address and enter it into your browser

Add the license from earlier:

[https://www.ryadel.com/en/resize-extend-disk-partition-unallocated-disk-space-linux-centos-rhel-ubuntu-debian/](https://www.ryadel.com/en/resize-extend-disk-partition-unallocated-disk-space-linux-centos-rhel-ubuntu-debian/ "https://www.ryadel.com/en/resize-extend-disk-partition-unallocated-disk-space-linux-centos-rhel-ubuntu-debian/")
