## Chapter 4 - Ansible Playbooks

### Power plays {#chap06.xhtml_leanpub-auto-power-plays}

Like many other configuration management solutions, Ansible uses a
metaphor to describe its configuration files. They are called
'playbooks', and they list a set of tasks ('plays' in Ansible parlance)
that will be run against a particular server or set of servers. In
American football, a team follows a set of pre-written playbooks as the
basis for a bunch of plays they execute to try to win a game. In
Ansible, you write playbooks (a list of instructions describing the
steps to bring your server to a certain configuration state) that are
then *play*ed on your servers.

Playbooks are written in
[YAML](http://docs.ansible.com/YAMLSyntax.html), a simple human-readable
syntax popular for defining configuration. Playbooks may be included
within other playbooks, and certain metadata and options cause different
plays or playbooks to be run in different scenarios on different
servers.

Ad-hoc commands alone make Ansible a powerful tool; playbooks turn
Ansible into a top-notch server provisioning and configuration
management tool.

What attracts most DevOps personnel to Ansible is the fact that it is
easy to convert shell scripts (or one-off shell commands) directly into
Ansible plays. Consider the following script, which installs Apache on a
RHEL/CentOS server:

**Shell Script**

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 # Install Apache.
2 yum install --quiet -y httpd httpd-devel
3 # Copy configuration files.
4 cp /path/to/config/httpd.conf /etc/httpd/conf/httpd.conf
5 cp /path/to/config/httpd-vhosts.conf /etc/httpd/conf/httpd-vhosts.conf
6 # Start Apache and configure it to run at boot.
7 service httpd start
8 chkconfig httpd on</code></pre>
</div>
</figure>

To run the shell script (in this case, a file named `shell-script.sh`
with the contents as above), you would call it directly from the command
line:

<figure class="code">
<div class="highlight">
<pre><code># (From the same directory in which the shell script resides).
$ ./shell-script.sh</code></pre>
</div>
</figure>

**Ansible Playbook**

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 ---
 2 - hosts: all
 3 
 4   tasks:
 5     - name: Install Apache.
 6       command: yum install --quiet -y httpd httpd-devel
 7     - name: Copy configuration files.
 8       command: &gt;
 9         cp /path/to/config/httpd.conf /etc/httpd/conf/httpd.conf
10     - command: &gt;
11         cp /path/to/config/httpd-vhosts.conf /etc/httpd/conf/httpd-vhosts.conf
12     - name: Start Apache and configure it to run at boot.
13       command: service httpd start
14     - command: chkconfig httpd on</code></pre>
</div>
</figure>

To run the Ansible Playbook (in this case, a file named `playbook.yml`
with the contents as above), you would call it using the
`ansible-playbook` command:

<figure class="code">
<div class="highlight">
<pre><code># (From the same directory in which the playbook resides).
$ ansible-playbook playbook.yml</code></pre>
</div>
</figure>

Ansible is powerful in that you quickly transition to using playbooks if
you know how to write standard shell commands---the same commands you've
been using for years---and then as you get time, rebuild your
configuration to take advantage of Ansible's helpful features.

In the above playbook, we use Ansible's `command` module to run standard
shell commands. We're also giving each play a 'name', so when we run the
playbook, the play has human-readable output on the screen or in the
logs. The command module has some other tricks up its sleeve (which
we'll see later), but for now, be assured shell scripts are translated
directly into Ansible playbooks without much hassle.

<aside class="tip blurb">

The greater-than sign (`>`) immediately following the `command:` module
directive tells YAML "automatically quote the next set of indented lines
as one long string, with each line separated by a space". It helps
improve task readability in some cases. There are different ways of
describing configuration using valid YAML syntax, and these methods are
discussed in-depth in the [YAML Conventions and Best
Practices](#chap15.xhtml_yaml-best-practices) section in Appendix B.

This book uses three different task-formatting techniques: For tasks
which require one or two simple parameters, Ansible's shorthand syntax
(e.g. `yum: name=apache2 state=present`) is used. For most uses of
`command` or `shell`, where longer commands are entered, the `>`
technique mentioned above is used. For tasks which require many
parameters, YAML object notation is used---placing each key and variable
on its own line. This assists with readability and allows for version
control systems to easily distinguish changes line-by-line.

</aside>

The above playbook will perform *exactly* like the shell script, but you
can improve things greatly by using some of Ansible's built-in modules
to handle the heavy lifting:

**Revised Ansible Playbook - Now with idempotence!**

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 ---
 2 - hosts: all
 3   sudo: yes
 4 
 5   tasks:
 6     - name: Install Apache.
 7       yum: name={{ item }} state=present
 8       with_items:
 9       - httpd
10       - httpd-devel
11     - name: Copy configuration files.
12       copy:
13         src: &quot;{{ item.src }}&quot;
14         dest: &quot;{{ item.dest }}&quot;
15         owner: root
16         group: root
17         mode: 0644
18       with_items:
19       - src: &quot;/path/to/config/httpd.conf&quot;
20         dest: &quot;/etc/httpd/conf/httpd.conf&quot;
21       - src: &quot;/path/to/config/httpd-vhosts.conf&quot;
22         dest: &quot;/etc/httpd/conf/httpd-vhosts.conf&quot;
23     - name: Make sure Apache is started and configure it to run at boot.
24       service: name=httpd state=started enabled=yes</code></pre>
</div>
</figure>

Now we're getting somewhere. Let me walk you through this simple
playbook:

1.  The first line, `---`, is how we mark this document as using YAML
    syntax (like using `<html>` at the top of an HTML document, or
    `<?php` at the top of a block of PHP code).
2.  The second line, `- hosts: all` defines the first (and in this case,
    only) *play*, and tells Ansible to run the play on `all` hosts that
    it knows about.
3.  The third line, `sudo: yes` tells Ansible to run all the commands
    through `sudo`, so the commands will be run as the root user.
4.  The fifth line, `tasks:`, tells Ansible that what follows is a list
    of tasks to run as part of this playbook.
5.  The first task begins with `name: Install Apache.`. `name` is not a
    module that does something to your server; rather, it's a way of
    giving a human-readable description to the play that follows. Seeing
    "Install Apache" is more relevant than seeing "yum name=httpd
    state=present"... but if you drop the name line completely, that
    won't cause any problem.
    - We use the `yum` module to install Apache. Instead of the command
      `yum -y install httpd httpd-devel`, we can describe to Ansible
      exactly what we want. Ansible will take the `items` array we pass
      in (`{{ variable }}` references a variable in Ansible's
      playbooks). We tell yum to make sure the packages we define are
      installed with `state=present`, but we could also use
      `state=latest` to ensure the latest version is installed, or
      `state=absent` to make sure the package is *not* installed.
    - Ansible allows simple lists to be passed into tasks using
      `with_items`: Define a list of items below, and each line will be
      passed into the play, one by one. In this case, each of the items
      will be substituted for the `{{ item }}` variable.
6.  The second task again starts with a human-readable name (which could
    be left out if you'd like).
    - We use the `copy` module to copy files from a source (on our local
      workstation) to a destination (the server being managed). We could
      also pass in more variables, like file metadata including
      ownership and permissions (`owner`, `group`, and `mode`).
    - In this case, we are using an array with multiple elements for
      variable substitution; you use the syntax
      `{var1: value, var2: value}` to define each element (it can have
      as many variables as you want within, or even nested levels of
      variables!). When you reference the variables in the play, you use
      a dot to access the variable within the item, so `{{ item.var1 }}`
      would access the first variable. In our example, `item.src`
      accesses the `src` in each item.
7.  The third task also uses a name to describe it in a human-readable
    format.
    - We use the `service` module to describe the desired state of a
      particular service, in this case `httpd`, Apache's http daemon. We
      want it to be running, so we set `state=started`, and we want it
      to run at system startup, so we say `enabled=yes` (the equivalent
      of running `chkconfig httpd on`).

The great thing about the way we've reformatted this list of commands is
that Ansible keeps track of the state of everything on all our servers.
If you run the playbook the first time, it will provision the server by
ensuring Apache is installed and running, and your custom configuration
is in place.

Even better, the *second* time you run it (if the server is in the
correct state), it won't actually do anything besides tell you nothing
has changed. So, with this one short playbook, you're able to provision
and ensure the proper configuration for an Apache web server.
Additionally, running the playbook with the `--check` option (see the
next section below) verifies the configuration matches what's defined in
the playbook, without actually running the tasks on the server.

If you ever want to update your configuration, or install another httpd
package, either update the file locally or add the package to the
`with_items` list and run the playbook again. Whether you have one or a
thousand servers, all of their configurations will be updated to match
your playbook---and Ansible will tell you if anything ever changes
(you're not making ad-hoc changes on individual production servers, *are
you*?).

### Running Playbooks with `ansible-playbook` {#chap06.xhtml_leanpub-auto-running-playbooks-with-ansible-playbook}

If we run the playbooks in the examples above (which are set to run on
`all` hosts), then the playbook would be run against every host defined
in your Ansible inventory file (see Chapter 1's [basic inventory file
example](#chap03.xhtml_basic-inventory)).

#### Limiting playbooks to particular hosts and groups {#chap06.xhtml_leanpub-auto-limiting-playbooks-to-particular-hosts-and-groups}

You can limit a playbook to specific groups or individual hosts by
changing the `hosts:` definition. The value can be set to `all` hosts, a
`group` of hosts defined in your inventory, multiple groups of hosts
(e.g. `webservers,dbservers`), individual hosts (e.g.
`atl.example.com`), or a mixture of hosts. You can even do wild card
matches, like `*.example.com`, to match all subdomains of a top-level
domain.

You can also limit the hosts on which the playbook is run via the
`ansible-playbook` command:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml --limit webservers</code></pre>
</div>
</figure>

In this case (assuming your inventory file contains a `webservers`
group), even if the playbook is set to `hosts: all`, or includes hosts
in addition to what's defined in the `webservers` group, it will only be
run on the hosts defined in `webservers`.

You could also limit the playbook to one particular host:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml --limit xyz.example.com</code></pre>
</div>
</figure>

If you want to see a list of hosts that would be affected by your
playbook before you actually run it, use `--list-hosts`:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml --list-hosts</code></pre>
</div>
</figure>

Running this should give output like:

<figure class="code">
<div class="highlight">
<pre><code>playbook: playbook.yml

  play #1 (all): host count=4
    127.0.0.1
    192.168.24.2
    foo.example.com
    bar.example.com</code></pre>
</div>
</figure>

(Where `count` is the count of servers defined in your inventory, and
following is a list of all the hosts defined in your inventory).

#### Setting user and sudo options with `ansible-playbook` {#chap06.xhtml_leanpub-auto-setting-user-and-sudo-options-with-ansible-playbook}

If no `user` is defined alongside the `hosts` in a playbook, Ansible
assumes you'll connect as the user defined in your inventory file for a
particular host, and then will fall back to your local user account
name. You can explicitly define a remote user to use for remote plays
using the `--remote-user` (`-u`) option:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml --remote-user=johndoe</code></pre>
</div>
</figure>

In some situations, you will need to pass along your sudo password to
the remote server to perform commands via `sudo`. In these situations,
you'll need use the `--ask-sudo-pass` (`-K`) option. You can also
explicitly force all tasks in a playbook to use sudo with `--sudo`.
Finally, you can define the sudo user for tasks run via `sudo` (the
default is root) with the `--sudo-user` (`-U`) option.

For example, the following command will run our example playbook with
sudo, performing the tasks as the sudo user `janedoe`, and Ansible will
prompt you for the sudo password:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml --sudo --sudo-user=janedoe --ask-sudo-pass</code></pre>
</div>
</figure>

If you're not using key-based authentication to connect to your servers
(read my warning about the security implications of doing so in Chapter
1), you can use `--ask-pass`.

#### Other options for `ansible-playbook` {#chap06.xhtml_leanpub-auto-other-options-for-ansible-playbook}

The `ansible-playbook` command also allows for some other common
options:

- `--inventory=PATH` (`-i PATH`): Define a custom inventory file
  (default is the default Ansible inventory file, usually located at
  `/etc/ansible/hosts`).
- `--verbose` (`-v`): Verbose mode (show all output, including output
  from successful options). You can pass in `-vvvv` to give every minute
  detail.
- `--extra-vars=VARS` (`-e VARS`): Define variables to be used in the
  playbook, in `"key=value,key=value"` format.
- `--forks=NUM` (`-f NUM`): Number for forks (integer). Set this to a
  number higher than 5 to increase the number of servers on which
  Ansible will run tasks concurrently.
- `--connection=TYPE` (`-c TYPE`): The type of connection which will be
  used (this defaults to `ssh`; you might sometimes want to use `local`
  to run a playbook on your local machine, or on a remote server via
  cron).
- `--check`: Run the playbook in Check Mode ('Dry Run'); all tasks
  defined in the playbook will be checked against all hosts, but none
  will actually be run.

There are some other options and configuration variables that are
important to get the most out of `ansible-playbook`, but this should be
enough to get you started running the playbooks in this chapter on your
own servers or virtual machines.

<aside class="information blurb">

The rest of this chapter uses more realistic Ansible playbooks. All the
examples in this chapter are in the [Ansible for DevOps GitHub
repository](https://github.com/geerlingguy/ansible-for-devops), and you
can clone that repository to your computer (or browse the code online)
to follow along more easily. The GitHub repository includes Vagrantfiles
with each example, so you can build the servers on your local host using
Vagrant.

</aside>

### Real-world playbook: CentOS Node.js app server {#chap06.xhtml_leanpub-auto-real-world-playbook-centos-nodejs-app-server}

The first example, while being helpful for someone who might want to
post a simple static web page to a clunky old Apache server, is not a
good representation of a real-world scenario. I'm going to run through
more complex playbooks that do many different things, most of which are
actually being used to manage production infrastructure today.

The first playbook will configure a CentOS server with Node.js, and
install and start a simple Node.js application. The server will have a
very simple architecture:

<figure class="image center" style="width: 40%;">
<img src="images/4-playbook-nodejs.png" style="width: 100%;"
alt="Node.js app on CentOS." />
<figcaption aria-hidden="true">Node.js app on CentOS.</figcaption>
</figure>

To start things off, we need to create a YAML file (`playbook.yml` in
this example) to contain our playbook. Let's keep things simple:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3 
4   tasks:</code></pre>
</div>
</figure>

First, define a set of hosts (`all`) on which this playbook will be run
(see the section above about limiting the playbook to particular groups
and hosts), then tell Ansible that what follows will be a list of tasks
to run on the hosts.

#### Add extra repositories {#chap06.xhtml_leanpub-auto-add-extra-repositories}

Adding extra package repositories (yum or apt) is one thing many admins
will do before any other work on a server to ensure that certain
packages are available, or are at a later version than the ones in the
base installation.

In the shell script below, we want to add both the EPEL and Remi
repositories, so we can get some packages like Node.js or later versions
of other necessary software (these examples presume you're running
RHEL/CentOS 7.x):

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 # Import Remi GPG key - see: http://rpms.famillecollet.com/RPM-GPG-KEY-remi
 2 wget http://rpms.famillecollet.com/RPM-GPG-KEY-remi \
 3   -O /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
 4 rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
 5 
 6 # Install Remi repo.
 7 rpm -Uvh --quiet \
 8   http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
 9 
10 # Install EPEL repo.
11 yum install epel-release
12 
13 # Install Node.js (npm plus all its dependencies).
14 yum --enablerepo=epel install node</code></pre>
</div>
</figure>

This shell script uses the rpm command to import the EPEL and Remi
repository GPG keys, then adds the repositories, and finally installs
Node.js. It works okay for a simple deployment (or by hand), but it's
silly to run all these commands (some of which could take time or stop
your script entirely if your connection is flaky or bad) if the result
has already been achieved (namely, two repositories and their GPG keys
have been added).

<aside class="information blurb">

If you wanted to skip a couple steps, you could skip adding the GPG
keys, and just run your commands with `--nogpgcheck` (or, in Ansible,
set the `disable_gpg_check` parameter of the yum module to `yes`), but
it's a good idea to leave this enabled. GPG stands for *GNU Privacy
Guard*, and it's a way that developers and package distributors can sign
their packages (so you know it's from the original author, and hasn't
been modified or corrupted). Unless you *really* know what you're doing,
don't disable security settings like GPG key checks.

</aside>

Ansible makes things a little more robust. Even though the following is
slightly more verbose, it performs the same actions in a more structured
way, which is simpler to understand, and works with variables other
nifty Ansible features we'll discuss later:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 9     - name: Import Remi GPG key.
10       rpm_key: &quot;key={{ item }} state=present&quot;
11       with_items:
12         - &quot;http://rpms.famillecollet.com/RPM-GPG-KEY-remi&quot;
13 
14     - name: Install Remi repo.
15       command: &quot;rpm -Uvh --force {{ item.href }} creates={{ item.creates }}&quot;
16       with_items:
17         - href: &quot;http://rpms.famillecollet.com/enterprise/remi-release-6.rpm&quot;
18           creates: &quot;/etc/yum.repos.d/remi.repo&quot;
19 
20     - name: Install EPEL repo.
21       yum: name=epel-release state=present
22 
23     - name: Ensure firewalld is stopped (since this is a test server).
24       service: name=firewalld state=stopped
25 
26     - name: Install Node.js and npm.
27       yum: name=npm state=present enablerepo=epel
28 
29     - name: Install Forever (to run our Node.js app).
30       npm: name=forever global=yes state=latest</code></pre>
</div>
</figure>

Let's walk through this playbook step-by-step:

1.  `rpm_key` is a very simple Ansible module that takes and imports an
    RPM key from a URL or file, or the key id of a key that is already
    present, and ensures the key is either present or absent (the
    `state` parameter). We're importing one key, for Remi's repository.
2.  Since Ansible doesn't have a built-in rpm module, we use the rpm
    command with Ansible's `command` module, which allows us to do two
    things:
    1.  Use the `creates` parameter to tell Ansible when to *not* run
        the command (in this case, we tell Ansible what file is present
        after the `rpm` command successfully completes).
    2.  Use a multidimensional array of items (`with_items`) to define
        URLs and the resulting files checked with `creates`.
3.  `yum` installs the EPEL repository (much simpler than the two-step
    process we had to follow to get Remi's repository installed!).
4.  Since this server is being used only for test purposes, we disable
    the system firewall so it won't interfere with testing (using the
    `service` module).
5.  `yum` installs Node.js (along with all the required packages for
    `npm`, Node's package manager) if it's not present, and allows the
    EPEL repo to be searched via the `enablerepo` parameter (you could
    also explicitly *disable* a repository using `disablerepo`).
6.  Since NPM is now installed, we use Ansible's `npm` module to install
    a Node.js utility, `forever` to launch our app and keep it running.
    Setting `global` to `yes` tells NPM to install the `forever` node
    module in `/usr/lib/node_modules/` so it will be available to all
    users and Node.js apps on the system.

We're beginning to have a nice little Node.js app server set up. Let's
set up a little Node.js app that responds to HTTP requests on port 80.

#### Deploy a Node.js app {#chap06.xhtml_leanpub-auto-deploy-a-nodejs-app}

The next step is to install a simple Node.js app on our server. First,
we'll create a really simple Node.js app by creating a new folder,
`app`, in the same folder as your playbook.yml. Create a new file,
`app.js`, in this folder, with the following contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 // Load the express module.
 2 var express = require(&#39;express&#39;),
 3 app = express.createServer();
 4 
 5 // Respond to requests for / with &#39;Hello World&#39;.
 6 app.get(&#39;/&#39;, function(req, res){
 7     res.send(&#39;Hello World!&#39;);
 8 });
 9 
10 // Listen on port 80 (like a true web server).
11 app.listen(80);
12 console.log(&#39;Express server started successfully.&#39;);</code></pre>
</div>
</figure>

Don't worry about the syntax or the fact that this is Node.js. We just
need a quick example to deploy. This example could've been written in
Python, Perl, Java, PHP, or another language, but since Node is a very
simple language (JavaScript) that runs in a very simple and lightweight
environment, it's a nice (and easy) language to use when testing things
or prodding your server.

Since this little app is dependent on Express (a simple http framework
for Node), we also need to tell NPM about this dependency via a
`package.json` file in the same folder as `app.js`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 {
2   &quot;name&quot;: &quot;examplenodeapp&quot;,
3   &quot;description&quot;: &quot;Example Express Node.js app.&quot;,
4   &quot;author&quot;: &quot;Jeff Geerling &lt;geerlingguy@mac.com&gt;&quot;,
5   &quot;dependencies&quot;: {
6     &quot;express&quot;: &quot;3.x.x&quot;
7   },
8   &quot;engine&quot;: &quot;node &gt;= 0.10.6&quot;
9 }</code></pre>
</div>
</figure>

Now, add the following to your playbook, to copy the entire app to the
server, and then have NPM download the required dependencies (in this
case, `express`):

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>34     - name: Ensure Node.js app folder exists.
35       file: &quot;path={{ node_apps_location }} state=directory&quot;
36 
37     - name: Copy example Node.js app to server.
38       copy: &quot;src=app dest={{ node_apps_location }}&quot;
39 
40     - name: Install app dependencies defined in package.json.
41       npm: path={{ node_apps_location }}/app</code></pre>
</div>
</figure>

First, we ensure the directory where our app will be installed exists,
using the `file` module. The `{{ node_apps_location }}` variable used in
each command can be defined under a `vars` section at the top of our
playbook, in your inventory, or on the command line when calling
`ansible-playbook`.

Second, we copy the entire app folder up to the server, using Ansible's
`copy` command, which intelligently distinguishes between a single file
or a directory of files, and recurses through the directory, similar to
recursive scp or rsync.

<aside class="tip blurb">

Ansible's `copy` module works very well for single or small groups of
files, and recurses through directories automatically. If you are
copying hundreds of files, or deeply-nested directory structures, `copy`
will get bogged down. In these situations, consider using the
`synchronize` module if you need to copy a full directory, or
`unarchive` if you want to copy up an archive and have it expanded in
place on the server.

</aside>

Third, we use `npm` again, this time, with no extra arguments besides
the path to the app. This tells NPM to parse the package.json file and
ensure all the dependencies are present.

We're *almost* finished! The last step is to start the app.

#### Launch a Node.js app {#chap06.xhtml_leanpub-auto-launch-a-nodejs-app}

We'll now use `forever` (which we installed earlier) to start the app.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>43     - name: Check list of running Node.js apps.
44       command: forever list
45       register: forever_list
46       changed_when: false
47 
48     - name: Start example Node.js app.
49       command: &quot;forever start {{ node_apps_location }}/app/app.js&quot;
50       when: &quot;forever_list.stdout.find(&#39;{{ node_apps_location }}/app/app.js&#39;) == \
51 -1&quot;</code></pre>
</div>
</figure>

In the first play, we're doing two new things:

1.  `register` creates a new variable, `forever_list`, to be used in the
    next play to determine when to run the play. `register` stashes the
    output (stdout, stderr) of the defined command in the variable name
    passed to it.
2.  `changed_when` tells Ansible explicitly when this play results in a
    change to the server. In this case, we know the `forever list`
    command will never change the server, so we just say `false`---the
    server will never be changed when the command is run.

The second play actually starts the app, using Forever. We could also
start the app by calling `node {{ node_apps_location }}/app/app.js`, but
we would not be able to control the process easily, and we would also
need to use `nohup` and `&` to avoid Ansible hanging on this play.

Forever tracks the Node apps it manages, and we use Forever's `list`
option to print a list of running apps. The first time we run this
playbook, the list will obviously be empty---but on future runs, if the
app is running, we don't want to start another instance of it. To avoid
that situation, we tell Ansible when we want to start the app with
`when`. Specifically, we tell Ansible to start the app only when the
app's path in *not* in the `forever list` output.

#### Node.js app server summary {#chap06.xhtml_leanpub-auto-nodejs-app-server-summary}

At this point, you have a complete playbook that will install a simple
Node.js app which responds to HTTP requests on port 80 with "Hello
World!".

To run the playbook on a server (in our case, we could just set up a new
VirtualBox VM for testing, either via Vagrant or manually), use the
following command (pass in the `node_apps_location` variable via the
command):

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml \
--extra-vars=&quot;node_apps_location=/usr/local/opt/node&quot;</code></pre>
</div>
</figure>

Once the playbook has finished configuring the server and deploying your
app, visit `http://hostname/` in a browser (or use `curl` or `wget` to
request the site), and you should see the following:

<figure class="image center" style="width: 60%;">
<img src="images/4-nodejs-home.png" style="width: 100%;"
alt="Node.js Application home page." />
<figcaption aria-hidden="true">Node.js Application home
page.</figcaption>
</figure>

Simple, but very powerful. We've configured an entire Node.js
application server In fewer than fifty lines of YAML!

<aside class="tip blurb">

The entire example Node.js app server playbook is in this book's code
repository at <https://github.com/geerlingguy/ansible-for-devops>, in
the `nodejs` directory.

</aside>

### Real-world playbook: Ubuntu LAMP server with Drupal {#chap06.xhtml_leanpub-auto-real-world-playbook-ubuntu-lamp-server-with-drupal}

At this point, you should be getting comfortable with Ansible playbooks
and the YAML syntax used to define them. Up to this point, most examples
have assumed you're working with a CentOS, RHEL, or Fedora server.
Ansible plays nicely with other flavors of Linux and BSD-like systems as
well. In the following example, we're going to set up a traditional LAMP
(Linux, Apache, MySQL, and PHP) server using Ubuntu 12.04 to run a
Drupal website.

<figure class="image center" style="width: 40%;">
<img src="images/4-playbook-drupal.png" style="width: 100%;"
alt="Drupal LAMP server." />
<figcaption aria-hidden="true">Drupal LAMP server.</figcaption>
</figure>

#### Include a variables file, and discover `pre_tasks` and `handlers` {#chap06.xhtml_leanpub-auto-include-a-variables-file-and-discover-pretasks-and-handlers}

To make our playbook more efficient and readable, let's begin the
playbook (named `playbook.yml`) by instructing Ansible to load in
variables from a separate `vars.yml` file:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3 
4   vars_files:
5     - vars.yml</code></pre>
</div>
</figure>

Using one or more included variable files cleans up your main playbook
file, and lets you organize all your configurable variables in one
place. At the moment, we don't have any variables to add; we'll define
the contents of `vars.yml` later. For now, create the empty file, and
continue on to the next section of the playbook, `pre_tasks`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>7   pre_tasks:
8     - name: Update apt cache if needed.
9       apt: update_cache=yes cache_valid_time=3600</code></pre>
</div>
</figure>

Ansible lets you run tasks before or after the main set of tasks using
`pre_tasks` and `post_tasks`. In this case, we need to ensure that our
apt cache is updated before we run the rest of the playbook, so we have
the latest package versions on our server. We use Ansible's `apt` module
and tell it to update the cache if it's been more than 3600 seconds (1
hour) since the last update.

With that out of the way, we'll add another new section to our playbook,
`handlers`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>11   handlers:
12     - name: restart apache
13       service: name=apache2 state=restarted</code></pre>
</div>
</figure>

`handlers` are special kinds of tasks you run at the end of a group of
tasks by adding the `notify` option to any of the tasks in that group.
The handler will only be called if one of the tasks notifying the
handler makes a change to the server (and doesn't fail), and it will
only be notified at the *end* of the group of tasks.

To call this handler, add the option `notify: restart apache` after
defining the rest of a play. We've defined this handler so we can
restart the `apache2` service after a configuration change, which will
be explained below.

<aside class="information blurb">

Just like variables, handlers and tasks may be placed in separate files
and included in your playbook to keep things tidy (we'll discuss this in
chapter 6). For simplicity's sake, though, the examples in this chapter
are shown as in a single playbook file. We'll discuss different playbook
organization methods later.

</aside>
<aside class="information blurb">

By default, Ansible will stop all playbook execution when a task fails,
and won't even notify any handlers that may need to be triggered. In
some cases, this leads to unintended side effects. If you want to make
sure handlers always run after a task uses `notify` to call the handler,
even in case of playbook failure, add `--force-handlers` to your
`ansible-playbook` command.

</aside>

#### Basic LAMP server setup {#chap06.xhtml_leanpub-auto-basic-lamp-server-setup}

The first step towards building an application server that depends on
the LAMP stack is to build the actual LAMP part of it. This is the
simplest process, but still requires a little extra work for our
particular server. We want to install Apache, MySQL and PHP, but we'll
also need a couple other dependencies, and we want a particular version
of PHP (5.5), which is only available in an extra apt repository.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>15   tasks:
16     - name: Get software for apt repository management.
17       apt: name={{ item }} state=present
18       with_items:
19         - python-apt
20         - python-pycurl
21 
22     - name: Add ondrej repository for later versions of PHP.
23       apt_repository: repo=&#39;ppa:ondrej/php5&#39; update_cache=yes
24 
25     - name: &quot;Install Apache, MySQL, PHP, and other dependencies.&quot;
26       apt: name={{ item }} state=present
27       with_items:
28         - git
29         - curl
30         - sendmail
31         - apache2
32         - php5
33         - php5-common
34         - php5-mysql
35         - php5-cli
36         - php5-curl
37         - php5-gd
38         - php5-dev
39         - php5-mcrypt
40         - php-apc
41         - php-pear
42         - python-mysqldb
43         - mysql-server
44 
45     - name: Disable the firewall (since this is for local dev only).
46       service: name=ufw state=stopped
47 
48     - name: &quot;Start Apache, MySQL, and PHP.&quot;
49       service: &quot;name={{ item }} state=started enabled=yes&quot;
50       with_items:
51         - apache2
52         - mysql</code></pre>
</div>
</figure>

In this playbook, I've decided to add a simple prefix to each named
play, so I can more easily follow the playbook's progress when it's
running. I've begun with the common LAMP setup:

1.  Install a couple helper libraries which allow Python to manage apt
    more precisely (`python-apt` and `python-pycurl` are required for
    the `apt_repository` module to do its work).
2.  Since the default apt repositories for Ubuntu 12.04 don't include
    PHP 5.4.x (or any later versions), install ondrej's `PHP5-oldstable`
    repository, containing PHP 5.4.25 (at the time of this writing) and
    other associated PHP packages.
3.  Install all the required packages for our LAMP server (including all
    the php5 extensions we need to run Drupal).
4.  Disable the firewall entirely, for testing purposes. If on a
    production server or any server exposed to the Internet, you should
    instead have a restrictive firewall only allowing access on ports
    22, 80, 443, and other necessary ports.
5.  Start up all the required services, and make sure they're enabled to
    start on system boot.

#### Configure Apache {#chap06.xhtml_leanpub-auto-configure-apache}

The next step is configuring Apache so it will work correctly with
Drupal. Out of the box, Apache doesn't have mod_rewrite enabled on
Ubuntu 12.04. To remedy that situation, you can use the command
`sudo a2enmod rewrite`, but Ansible has a handy `apache2_module` module
that will simplify the task.

Additionally, we need to add a VirtualHost entry to tell Apache where
the site's document root is, and any other options for the site.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>54     - name: Enable Apache rewrite module (required for Drupal).
55       apache2_module: name=rewrite state=present
56       notify: restart apache
57 
58     - name: Add Apache virtualhost for Drupal 8 development.
59       template:
60         src: &quot;templates/drupal.dev.conf.j2&quot;
61         dest: &quot;/etc/apache2/sites-available/{{ domain }}.dev.conf&quot;
62         owner: root
63         group: root
64         mode: 0644
65       notify: restart apache
66 
67     - name: Symlink Drupal virtualhost to sites-enabled.
68       file:
69         src: &quot;/etc/apache2/sites-available/{{ domain }}.dev.conf&quot;
70         dest: &quot;/etc/apache2/sites-enabled/{{ domain }}.dev.conf&quot;
71         state: link
72       notify: restart apache
73 
74     - name: Remove default virtualhost file.
75       file:
76         path: &quot;/etc/apache2/sites-enabled/000-default&quot;
77         state: absent
78       notify: restart apache</code></pre>
</div>
</figure>

The first command enables all the required Apache modules by symlinking
them from `/etc/apache2/mods-available` to `/etc/apache2/mods-enabled`.

The second command copies a Jinja2 template we define inside the
templates folder to Apache's `sites-available` folder, with the correct
owner and permissions. Additionally, we `notify` the `restart apache`
handler, because copying in a new VirtualHost means Apache needs to be
restarted to pick up the change.

Let's look at our Jinja2 template (denoted by the extra `.j2` on the end
of the filename), `drupal.dev.conf.j2`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 &lt;VirtualHost *:80&gt;
 2     ServerAdmin webmaster@localhost
 3     ServerName {{ domain }}.dev
 4     ServerAlias www.{{ domain }}.dev
 5     DocumentRoot {{ drupal_core_path }}
 6     &lt;Directory &quot;{{ drupal_core_path }}&quot;&gt;
 7         Options FollowSymLinks Indexes
 8         AllowOverride All
 9     &lt;/Directory&gt;
10 &lt;/VirtualHost&gt;</code></pre>
</div>
</figure>

This is a fairly standard Apache VirtualHost definition, but we have a
few Jinja2 template variables mixed in. The syntax for printing a
variable in a Jinja2 template is the same syntax we use in our Ansible
playbooks---two brackets around the variable's name (like so:
`{{ variable }}`).

There are three variables we will need (`drupal_core_version`,
`drupal_core_path`, and `domain`), so add them to the empty `vars.yml`
file we created earlier:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 # The core version you want to use (e.g. 6.x, 7.x, 8.0.x).
3 drupal_core_version: &quot;8.0.x&quot;
4 
5 # The path where Drupal will be downloaded and installed.
6 drupal_core_path: &quot;/var/www/drupal-{{ drupal_core_version }}-dev&quot;
7 
8 # The resulting domain will be [domain].dev (with .dev appended).
9 domain: &quot;drupaltest&quot;</code></pre>
</div>
</figure>

Now, when Ansible reaches the play that copies this template into place,
the Jinja2 template will have the variable names replaced with the
values `/var/www/drupal-8.0.x-dev` and `drupaltest` (or whatever values
you'd like!).

The last two tasks (lines 12-19) enable the VirtualHost we just added,
and remove the default VirtualHost definition, which we no longer need.

At this point, you could start the server, but Apache will likely throw
an error since the VirtualHost you've defined doesn't yet exist (there's
no directory at `{{ drupal_core_path }}` yet!). This is why using
`notify` is important---instead of adding a play after these three steps
to restart Apache (which will fail the first time you run the playbook),
notify will wait until after we've finished all the other steps in our
main group of tasks (giving us time to finish setting up the server),
*then* restart Apache.

#### Configure PHP with `lineinfile` {#chap06.xhtml_lineinfile-php}

We briefly mentioned `lineinfile` earlier in the book, when discussing
file management and ad-hoc task execution. Modifying PHP's configuration
is a perfect way to demonstrate `lineinfile`'s simplicity and
usefulness:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>80     - name: Enable upload progress via APC.
81       lineinfile:
82         dest: &quot;/etc/php5/apache2/conf.d/20-apcu.ini&quot;
83         regexp: &quot;^apc.rfc1867&quot;
84         line: &quot;apc.rfc1867 = 1&quot;
85         state: present
86       notify: restart apache</code></pre>
</div>
</figure>

Ansible's `lineinfile` module does a simple task: ensures that a
particular line of text exists (or doesn't exist) in a file.

In this example, we need to enable APC's `rfc1867` option so Drupal can
use APC's file upload progress tracking (there are better ways of doing
this, but for our simple server, this will suffice).

First, we tell `lineinfile` the location of the file, in the `dest`
parameter. Then, we give a regular expression (Python-style) to define
what the line looks like (in this case, the line starts with the exact
phrase "apc.rfc1867"---we had to escape the period since it is a special
character in regular expressions). Next, we tell `lineinfile` exactly
how the resulting line should look. Finally, we explicitly state that we
want this line to be present (with the `state` parameter).

Ansible will take the regular expression, and see if there's a matching
line. If there is, Ansible will make sure the line matches the `line`
parameter. If not, Ansible will add the line as defined in the `line`
parameter. Ansible will only report a change if it had to add or change
the line to match `line`.

#### Configure MySQL {#chap06.xhtml_leanpub-auto-configure-mysql}

The next step is to remove MySQL's default test database, and create a
database (named for the domain we specified earlier) for our Drupal
installation to use.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>88     - name: Remove the MySQL test database.
89       mysql_db: db=test state=absent
90 
91     - name: Create a database for Drupal.
92       mysql_db: &quot;db={{ domain }} state=present&quot;</code></pre>
</div>
</figure>

MySQL installs a database named `test` by default, and it is recommended
that you remove the database as part of MySQL's included
`mysql_secure_installation` tool. The first step in configuring MySQL is
removing this database. Next, we create a database named
`{{ domain }}`---the database is named the same as the domain we're
using for the Drupal site.

<aside class="information blurb">

Ansible works with many databases out of the box (MongoDB, MySQL,
PostgreSQL, Redis and Riak as of this writing). In MySQL's case, Ansible
uses the MySQLdb Python package (`python-mysqldb`) to manage a
connection to the database server, and assumes the default root account
credentials ('root' as the username with no password). Obviously,
leaving this default would be a bad idea! On a production server, one of
the first steps should be to change the root account password, limit the
root account to localhost, and delete any nonessential database users.

If you use different credentials, you can add a .my.cnf file to your
remote user's home directory containing the database credentials to
allow Ansible to connect to the MySQL database without leaving passwords
in your Ansible playbooks or variable files. Otherwise, you can prompt
the user running the Ansible playbook for a MySQL username and password.
This option, using prompts, will be discussed later in the book.

</aside>

#### Install Composer and Drush {#chap06.xhtml_leanpub-auto-install-composer-and-drush}

Drupal has a command-line companion in the form of Drush. Drush is
developed independently of Drupal, and provides a full suite of CLI
commands to manage Drupal. Drush, like most modern PHP tools, integrates
with external dependencies defined in a `composer.json` file which
describes the dependencies to Composer.

We could just download Drupal and perform some setup in the browser by
hand at this point, but the goal of this playbook is to have a
fully-automated and idempotent Drupal installation. So, we need to
install Composer, then Drush:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 94     - name: Download Composer installer.
 95       get_url:
 96         url: https://getcomposer.org/installer
 97         dest: /tmp/composer-installer.php
 98         mode: 0755
 99 
100     - name: Run Composer installer.
101       command: &gt;
102         php composer-installer.php
103         chdir=/tmp
104         creates=/usr/local/bin/composer
105 
106     - name: Move Composer into globally-accessible location.
107       shell: &gt;
108         mv /tmp/composer.phar /usr/local/bin/composer
109         creates=/usr/local/bin/composer</code></pre>
</div>
</figure>

The first two commands download and run Composer's php-based installer,
which generates a 'composer.phar' PHP application archive in `/tmp`.
This archive is then copied (using the `mv` shell command) to the
location `/usr/local/bin/composer` so we can use the simple `composer`
command to install all of Drush's dependencies. The latter two commands
commands are set to run only if the `/usr/local/bin/composer` file
doesn't already exist (using the `creates` parameter).

<aside class="tip blurb">

Why use `shell` instead of `command`? Ansible's `command` module is the
preferred option for running commands on a host (when an Ansible module
won't suffice), and it works in most scenarios. However, `command`
doesn't run the command via the remote shell `/bin/sh`, so options like
`<`, `>`, `|`, and `&`, and local environment variables like `$HOME`
won't work. `shell` allows you to pipe command output to other commands,
access the local environment, etc.

There are two other modules which assist in executing shell commands
remotely: `script` executes shell scripts (though it's almost always a
better idea to convert shell scripts into idempotent Ansible
playbooks!), and `raw` executes raw commands via SSH (it should only be
used in circumstances where you can't use one of the other options).

It's best to use an Ansible module for every task. If you have to resort
to a regular command-line command, try the the `command` module first.
If you require the options mentioned above, use `shell`. Use of `script`
or `raw` should be exceedingly rare, and won't be covered in this book.

</aside>

Now, we'll install Drush using the latest version from GitHub:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>111     - name: Check out drush master branch.
112       git:
113         repo: https://github.com/drush-ops/drush.git
114         dest: /opt/drush
115 
116     - name: Install Drush dependencies with Composer.
117       shell: &gt;
118         /usr/local/bin/composer install
119         chdir=/opt/drush
120         creates=/opt/drush/vendor/autoload.php
121 
122     - name: Create drush bin symlink.
123       file:
124         src: /opt/drush/drush
125         dest: /usr/local/bin/drush
126         state: link</code></pre>
</div>
</figure>

Earlier in the book, we cloned a git repository using an ad-hoc command.
In this case, we're defining a play that uses the `git` module to clone
Drush from its repository URL on GitHub. Since we want the master
branch, pass in the `repo` (repository URL) and `dest` (destination
path) parameters.

After Drush is downloaded to `/opt/drush`, we use Composer to install
all the required dependencies. In this case, we want Ansible to run
`composer install` in the directory `/opt/drush` (this is so Composer
finds Drush's `composer.json` file automatically), so we pass along the
parameter `chdir=/opt/drush`. Once Composer is finished, the file
`/opt/drush/vendor/autoload.php` will be created, so we use the
`creates` parameter to tell Ansible to skip this step if the file
already exists (for idempotency).

Finally, we create a symlink from `/usr/local/bin/drush` to the
executable at `/opt/drush/drush`, so we can call the `drush` command
anywhere on the system.

#### Install Drupal with Git and Drush {#chap06.xhtml_leanpub-auto-install-drupal-with-git-and-drush}

We'll use `git` again to clone Drupal to the Apache document root we
defined earlier in our virtual host configuration, then we'll run
Drupal's installation via Drush, and fix a couple other file permissions
issues so Drupal loads correctly within our VM.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>128     - name: Check out Drupal Core to the Apache docroot.
129       git:
130         repo: http://git.drupal.org/project/drupal.git
131         version: &quot;{{ drupal_core_version }}&quot;
132         dest: &quot;{{ drupal_core_path }}&quot;
133 
134     - name: Install Drupal.
135       command: &gt;
136         drush si -y --site-name=&quot;{{ drupal_site_name }}&quot; --account-name=admin
137         --account-pass=admin --db-url=mysql://root@localhost/{{ domain }}
138         chdir={{ drupal_core_path }}
139         creates={{ drupal_core_path }}/sites/default/settings.php
140       notify: restart apache
141 
142     # SEE: https://drupal.org/node/2121849#comment-8413637
143     - name: Set permissions properly on settings.php.
144       file:
145         path: &quot;{{ drupal_core_path }}/sites/default/settings.php&quot;
146         mode: 0744
147 
148     - name: Set permissions on files directory.
149       file:
150         path: &quot;{{ drupal_core_path }}/sites/default/files&quot;
151         mode: 0777
152         state: directory
153         recurse: yes</code></pre>
</div>
</figure>

First, we cloned Drupal's git repository, using the `version` defined in
our `vars.yml` file as `drupal_core_version`. The `git` module's
`version` parameter defines the branch (`master`, `8.0.x`, etc.), tag
(`1.0.1`, `7.24`, etc.), or individual commit hash (`50a1877`, etc.) to
clone.

Next, we used Drush's `si` command (short for `site-install`) to run
Drupal's installation (which configures the database, runs some
maintenance, and sets some default configuration settings for the site).
We passed in a few variables, like the `drupal_core_version` and
`domain`; we also added a `drupal_site_name`, so add that variable to
your `vars.yml` file:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>10 # Your Drupal site name.
11 drupal_site_name: &quot;D8 Test&quot;</code></pre>
</div>
</figure>

Also, Drupal's installation process results in the creation of a
'settings.php' file, so we use the location of that file with the
`creates` parameter to let Ansible know if the site's already installed
(so we don't accidentally try installing it again!). Once the site is
installed, we also restart Apache for good measure (using `notify`
again, like we did when updating Apache's configuration).

The final two tasks set permissions on Drupal's settings.php and files
folder to 744 and 777, respectively.

#### Drupal LAMP server summary {#chap06.xhtml_leanpub-auto-drupal-lamp-server-summary}

To run the playbook on a server (either via a local VM for testing or on
another server), use the following command:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook playbook.yml</code></pre>
</div>
</figure>

After the playbook completes, if you access the server at
http://drupaltest.dev/ (assuming you've pointed drupaltest.dev to your
server or VM's IP address), you'll see Drupal's default home page, and
you could login with 'admin'/'admin'. (Obviously, you'd set a secure
password on a production server!).

A similar server configuration, running Apache, MySQL, and PHP, can be
used to run many popular web frameworks and CMSes besides Drupal,
including Symfony, Wordpress, Joomla, Laravel, etc.

<aside class="tip blurb">

The entire example Drupal LAMP server playbook is in this book's code
repository at <https://github.com/geerlingguy/ansible-for-devops>, in
the `drupal` directory.

</aside>

### Real-world playbook: Ubuntu Apache Tomcat server with Solr {#chap06.xhtml_leanpub-auto-real-world-playbook-ubuntu-apache-tomcat-server-with-solr}

Apache Solr is a fast and scalable search server optimized for full-text
search, word highlighting, faceted search, fast indexing, and more. It's
a very popular search server, and it's pretty easy to install and
configure using Ansible. In the following example, we're going to set up
Apache Solr using Ubuntu 12.04 and Apache Tomcat.

<figure class="image center" style="width: 40%;">
<img src="images/4-playbook-solr.png" style="width: 100%;"
alt="Apache Solr Server." />
<figcaption aria-hidden="true">Apache Solr Server.</figcaption>
</figure>

#### Include a variables file, and more `pre_tasks` and `handlers` {#chap06.xhtml_leanpub-auto-include-a-variables-file-and-more-pretasks-and-handlers}

Just like the previous LAMP server example, we'll begin this playbook
(again named `playbook.yml`) by telling Ansible our variables will be in
a separate `vars.yml` file:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3 
4   vars_files:
5   - vars.yml</code></pre>
</div>
</figure>

Let's quickly create the `vars.yml` file, while we're thinking about it.
Create the file in the same folder as your Solr playbook, and add the
following contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 download_dir: /tmp
2 solr_dir: /opt/solr
3 solr_version: 4.10.4
4 solr_sha256sum: ac3543880f1b591bcaa962d7508b528d7b42e2b5548386197940b704629ae851</code></pre>
</div>
</figure>

These two variables define two paths we'll use while downloading and
installing Apache Solr.

Back in our playbook, after the `vars_files`, we also need to make sure
the apt cache is up to date, using `pre_tasks` like the previous
example:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>7   pre_tasks:
8     - name: Update apt cache if needed.
9       apt: update_cache=yes cache_valid_time=3600</code></pre>
</div>
</figure>

Like the Drupal playbook, we again use `handlers` to define certain
tasks that are notified by tasks in the `tasks` section. This time, we
just need a handler to restart `tomcat7`, the Java servlet container
that powers Apache Solr:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>11   handlers:
12     - name: restart tomcat
13       service: name=tomcat7 state=restarted</code></pre>
</div>
</figure>

We can call this handler with the option `notify: restart tomcat` in any
play in our playbook.

#### Install Apache Tomcat 7 {#chap06.xhtml_leanpub-auto-install-apache-tomcat-7}

It's easy enough to install Tomcat 7 on an Ubuntu Precise server; there
are packages in the default apt repositories, so we just need to make
sure they're installed, and that the `tomcat7` service is enabled and
started:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>15   tasks:
16     - name: Install Tomcat 7.
17       apt: &quot;name={{ item }} state=present&quot;
18       with_items:
19         - tomcat7
20         - tomcat7-admin
21 
22     - name: Ensure Tomcat 7 is started and enabled on boot.
23       service: name=tomcat7 state=started enabled=yes</code></pre>
</div>
</figure>

That was easy enough! We used the `apt` module to install two packages,
`tomcat7` and `tomcat7-admin` (so we can log into Tomcat's
administrative backend), then started `tomcat7` and set it to start when
the system boots.

#### Install Apache Solr {#chap06.xhtml_leanpub-auto-install-apache-solr}

Ubuntu 12.04 includes a package for Apache Solr, but it installs a very
old version, so we'll install the latest version of Solr from source.
The first step is downloading the source:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>25     - name: Download Solr.
26       get_url:
27         url: &quot;http://apache.osuosl.org/lucene/solr/{{ solr_version }}/solr-{{ so\
28 lr_version }}.tgz&quot;
29         dest: &quot;{{ download_dir }}/solr-{{ solr_version }}.tgz&quot;
30         sha256sum: &quot;{{ solr_sha256sum }}&quot;</code></pre>
</div>
</figure>

When downloading files from remote servers, the `get_url` module
provides more flexibility and convenience than raw `wget` or `curl`
commands.

You have to pass `get_url` a `url` (the source of the file to be
downloaded), and a `dest` (the location where the file will be
downloaded). If you pass a directory to the `dest` parameter, Ansible
will place the file inside, but will always re-download the file on
subsequent runs of the playbook (and overwrite the existing download if
it has changed). To avoid this extra overhead, we give the full path to
the downloaded file.

We also use `sha256sum`, an optional parameter, for peace of mind; if
you are downloading a file or archive that's critical to the
functionality and security of your application, it's a good idea to
check the file to make sure it is exactly what you're expecting.
`sha256sum` compares a hash of the data in the downloaded file to a
256-bit hash that you specify (use `shasum -a 256 /path/to/file` to get
the `sha256sum` of a file). If the checksum doesn't match the supplied
hash, Ansible will fail and discard the freshly-downloaded (and invalid)
file.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>31     - name: Expand Solr.
32       command: &gt;
33         tar -C /tmp -xvzf {{ download_dir }}/solr-{{ solr_version }}.tgz
34         creates={{ download_dir }}/solr-{{ solr_version }}/dist/solr-{{ solr_ver\
35 sion }}.war
36 
37     - name: Copy Solr into place.
38       command: &gt;
39         cp -r {{ download_dir }}/solr-{{ solr_version }} {{ solr_dir }}
40         creates={{ solr_dir }}/dist/solr-{{ solr_version }}.war</code></pre>
</div>
</figure>

We need to expand the Apache Solr archive, then copy it into place. For
both of these steps, use the built-in `tar` and `cp` utilities (with the
appropriate options) to do the work. Setting `creates` tells Ansible to
skip these steps in subsequent runs, since the Solr war file will
already be in place.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>41     # Use shell so commands are passed in correctly.
42     - name: Copy Solr components into place.
43       shell: &gt;
44         cp -r {{ item.src }} {{ item.dest }}
45         creates={{ item.creates }}
46       with_items:
47         # Solr example configuration and war file.
48         - src: &quot;{{ solr_dir }}/example/webapps/solr.war&quot;
49           dest: &quot;{{ solr_dir }}/solr.war&quot;
50           creates: &quot;{{ solr_dir }}/solr.war&quot;
51 
52         - src: &quot;{{ solr_dir }}/example/solr/*&quot;
53           dest: &quot;{{ solr_dir }}/&quot;
54           creates: &quot;{{ solr_dir }}/solr.xml&quot;
55 
56         # Solr log4j logging configuration.
57         - src: &quot;{{ solr_dir }}/example/lib/ext/*&quot;
58           dest: &quot;/var/lib/tomcat7/shared/&quot;
59           creates: &quot;/var/lib/tomcat7/shared/log4j-1.2.16.jar&quot;
60 
61         - src: &quot;{{ solr_dir }}/example/resources/log4j.properties&quot;
62           dest: &quot;/var/lib/tomcat7/shared/classes&quot;
63           creates: &quot;/var/lib/tomcat7/shared/classes/log4j.properties&quot;
64       notify: restart tomcat</code></pre>
</div>
</figure>

The next task copies into place certain directories and files required
to run Apache Solr.

Nothing too special here, but this example illustrates the use of
comments within `with_items` lists to help clarify the items in the
list. We could've added each command as its own task, but doing it this
way reduces the total number of Ansible tasks and allows us to move the
`with_items` list to an external variable if desired.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>66     - name: Ensure solr example directory is absent.
67       file:
68         path: &quot;{{ solr_dir }}/example&quot;
69         state: absent
70 
71     - name: Set up solr data directory.
72       file:
73         path: &quot;{{ solr_dir }}/data&quot;
74         state: directory
75         owner: tomcat7
76         group: tomcat7</code></pre>
</div>
</figure>

The latest version of Apache Solr searches through all the directories
inside `{{ solr_dir }}` recursively, loading any potential search
configuration it finds. Since we copied over one of the examples to use
as the server's default search core, Solr would see it as a duplicate of
one of the examples and crash. So, we use the `file` module with a
`path` to the example directory to make sure the directory is gone
(`state=absent`).

After removing the example directory (and in future runs, ensuring it's
still gone), we set up the data directory where Solr will store index
data, ensuring it exists as a directory and is owned by the `tomcat7`
user and group.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>78     - name: Configure solrconfig.xml for new data directory.
79       lineinfile:
80         dest: &quot;{{ solr_dir }}/collection1/conf/solrconfig.xml&quot;
81         regexp: &quot;^.*&lt;dataDir.+$&quot;
82         line: &quot;&lt;dataDir&gt;${solr.data.dir:{{ solr_dir }}/data}&lt;/dataDir&gt;&quot;
83         state: present</code></pre>
</div>
</figure>

As we found [earlier](#chap06.xhtml_lineinfile-php), `lineinfile` is a
helpful module for ensuring consistent configuration file settings with
idempotence. In this case, we need to make sure the `<dataDir>` line in
our default search core's configuration file is set to a specific value.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>85     - name: Set permissions for solr home.
86       file:
87         path: &quot;{{ solr_dir }}&quot;
88         recurse: yes
89         owner: tomcat7
90         group: tomcat7</code></pre>
</div>
</figure>

To set ownership options on the entire contents of the `{{ solr_dir }}`
correctly, we use the `file` module with the `recurse` parameter set to
`yes`. This is equivalent to the shell command
`chown -R tomcat7:tomcat7 {{ solr_dir }}`.

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>92   - name: Add Catalina configuration for solr.
93     template:
94       src: templates/solr.xml.j2
95       dest: /etc/tomcat7/Catalina/localhost/solr.xml
96       owner: root
97       group: tomcat7
98       mode: 0644
99     notify: restart tomcat</code></pre>
</div>
</figure>

The final task copies a template file (`solr.xml.j2`) to the remote
host, substituting variables via Jinja2 syntax, and sets the file's
ownership and permissions as needed for Tomcat.

Before the task runs, the local template file will need to be created.
Create a 'templates' folder in the same directory as your Apache Solr
playbook, and create a new file named `solr.xml.j2` inside, with the
following contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 &lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
2 &lt;Context docBase=&quot;{{ solr_dir }}/solr.war&quot; debug=&quot;0&quot; crossContext=&quot;true&quot;&gt;
3   &lt;Environment name=&quot;solr/home&quot; type=&quot;java.lang.String&quot; \
4   value=&quot;{{ solr_dir }}&quot; override=&quot;true&quot;/&gt;
5 &lt;/Context&gt;</code></pre>
</div>
</figure>

Run the playbook with `$ ansible-playbook playbook.yml`, and after a few
minutes (depending on your server's Internet connection speed), you
should be able to access the Solr admin interface at
http://example.com:8080/solr (where 'example.com' is your server's
hostname or IP address).

#### Apache Solr server summary {#chap06.xhtml_leanpub-auto-apache-solr-server-summary}

The configuration we used when deploying Apache Solr allows for a multi
core setup, so you could add more 'search cores' via the admin interface
(as long as the directories and core schema configuration is in place in
the filesystem), and have multiple indexes for multiple websites and
applications.

A playbook similar to the one above is used as part of the
infrastructure for [Hosted Apache Solr](http://hostedapachesolr.com/), a
service I run which hosts Apache Solr search cores for Drupal websites.

<aside class="tip blurb">

The entire example Apache Solr server playbook is in this book's code
repository at <https://github.com/geerlingguy/ansible-for-devops>, in
the `solr` directory.

</aside>

### Summary {#chap06.xhtml_leanpub-auto-summary-3}

At this point, you should be getting comfortable with Ansible's *modus
operandi*. Playbooks are the heart of Ansible's configuration management
and provisioning functionality, and the same modules and similar syntax
can be used with ad-hoc commands for deployments and general server
management.

Now that you're familiar with playbooks, we'll explore more advanced
concepts in building playbooks, like organization of tasks,
conditionals, variables, and more. Later, we'll explore the use of
playbooks with roles to make them infinitely more flexible and to save
time setting up and configuring your infrastructure.

<figure class="code">
<div class="highlight">
<pre><code> _________________________________________
/ If everything is under control, you are \
\ going too slow. (Mario Andretti)        /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||</code></pre>
</div>
</figure>


