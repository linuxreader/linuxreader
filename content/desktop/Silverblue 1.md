Automated setup https://universal-blue.org/

You get all the benefits of using containers
Separates system level packages from applications.

System Level
- Desktop, kernel?
- Layering
	- Apps at system level because containers aren't as developed yet
	- Locks to the fedora version you are on

## Layered package examples
	- gnome shell extensions
	- distrobox


Uses rpm-ostree? https://coreos.github.io/rpm-ostree/administrator-handbook/

Flatpacks

Remove fedora flatpack stuff and use flathub repos instead https://flatpak.org/setup/Fedora

Systemd unit for automatic flatpack updates


Update every 4 hours to mirror ubuntu

flatseal
adjust permissions of flatpacks

check out apps.gnome.org

## Rebase into Universal Blue

Rebase onto the "unsigned" image then reboot:
`rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/silverblue-main:39` and 

Then the signed image and reboot:
`rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/silverblue-main:39`
    
Then do we you do after install, open the app store and install stuff via GUI or we 

## Bazzite Explanation

https://www.youtube.com/watch?v=BBDOhvS_3SM

Shove setup in a containerfile > put podman in a for loop > operating system

