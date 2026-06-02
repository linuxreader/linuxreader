# DNF Package Groups

A package group  is a group of packages that serve a common purpose. This let's you query, install, and delete multiple packages as a single unit.

There are two types of package groups: _environment groups_ and _package groups_. 

**environment groups** are the option listed on the software selection window during RHEL installation. They provide all the necessary software to build the OS for a desired purpose. Environment groups include: 
- Server
- Server with GUI
- Minimal Install
- Workstation
- KDE Plasma Workspaces
- Virtualization Host
- Custom Operating System

**Package groups** are groups of packages that serve a common purpose. These include:
- Container Management
- Headless Management
- Console Internet Tools
- .NET Development
- RPM Development Tools
- Development Tools
- Graphical Administration Tools
- Legacy UNIX Compatibility
- Network Servers
- Scientific Support
- Security Tools
- Smart Card Support
- System Tools
- Fedora Packager
- VideoLAN Client
- Xfce
## Package Group Management

Use the `group` dnf subcommand to list, install, query, and remove groups of packages.

### Listing Available and Installed Package Groups

List all available and installed package groups from all repositories:
```bash
 dnf group list
```

The output shows both environment and package groups.
```bash
Available Environment Groups:
   Server with GUI
   Minimal Install
   Workstation
   KDE Plasma Workspaces
   Virtualization Host
   Custom Operating System
Installed Environment Groups:
   Server
Installed Groups:
   Container Management
   Headless Management
Available Groups:
   Console Internet Tools
   .NET Development
   RPM Development Tools
   Development Tools
   Graphical Administration Tools
   Legacy UNIX Compatibility
   Network Servers
   Scientific Support
   Security Tools
   Smart Card Support
   System Tools
   Fedora Packager
   VideoLAN Client
   Xfce
```



Package group
- Small bunch of RHEL packages that serve a common purpose. 
- Saves time on the deployment of individual and dependent packages. 
- Output shows installed and available package groups.

Display the number of installed and available package groups:
```bash
 sudo dnf group summary
```

List all installed and available package groups including those that
are hidden:
```bash
 sudo dnf group list hidden
```

Try *group list* with \--installed and \--available options to narrow
down the output list.
```bash
 sudo dnf group list --installed
```

List all packages that a specific package group such as Base contains:
```bash
 sudo dnf group info Base
```

-v option with the group info subcommand for more
information. 

Review group list and group info subsections of the `dnf` man pages.

### Installing and Updating Package Groups

- Creates the necessary directory structure for all the packages included in the group and all dependent packages.
- Installs the required files.
- Runs any post-installation steps.
- Attempts to update all the packages included in the group to the latest available versions. 

Install a package group called Emacs. Update if it detects an older version.
```bash
 sudo dnf -y groupinstall emacs
```

Update the *smart card support* package group to the latest version:
```bash
 dnf groupupdate "Smart Card Support"
```

Refer to the *group install* and *group update* subsections of the *dnf* command manual pages for more details.

### Removing Package Groups
- Uninstalls all the included packages and deletes all associated files and directory structure.
- Erases any dependencies

Erase the *smart card support* package group that was installed:

```bash
 sudo dnf -y groupremove 'smart card support'
```

Refer to the *remove* subsection of the *dnf* command manual pages for
more details.

### Lab: Manipulate Package Groups
Perform management operations on a package group called *system tools*. Determine if this group is already installed and if it is available for installation. List the packages it contains and install it. Remove the group along with its dependencies and confirm the removal.

1. Check whether the *system tools* package group is already installed:
```bash
 dnf group list installed
```

2. Determine if the *system tools* group is available for installation:
```bash
 dnf group list available
```

The group name is exhibited at the bottom of the list under the
available groups.

3. Display the list of packages this group contains:
```bash
 dnf group info 'system tools'
```

- All of the packages will be installed as part of the group installation.

4. Install the group:
```bash
 sudo dnf group install 'system tools'
```

5. Remove the group:
```bash
 sudo dnf group remove 'system tools' -y
```

6. Confirm the removal:
```bash
 dnf group list installed
```
