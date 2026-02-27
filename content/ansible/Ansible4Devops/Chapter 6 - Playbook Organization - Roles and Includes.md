## Chapter 6 - Playbook Organization - Roles and Includes

So far, we've used fairly straightforward examples in this book. Most
examples are created for a particular server, and are in one long
playbook.

Ansible is flexible when it comes to organizing tasks in more efficient
ways so you can make playbooks more maintainable, reusable, and
powerful. We'll look at two ways to split up tasks more efficiently
using includes and roles, and we'll explore Ansible Galaxy, a repository
of some community-maintained roles that help configure common packages
and applications.

### Includes {#chap08.xhtml_leanpub-auto-includes}

We've already seen one of the most basic ways of including other files
in Chapter 4, when `vars_files` was used to place variables into a
separate `vars.yml` file instead of inline with the playbook:

<figure class="code">
<div class="highlight">
<pre><code>- hosts: all

  vars_files:
  - vars.yml</code></pre>
</div>
</figure>

Tasks can easily be included in a similar way. In the `tasks:` section
of your playbook, you can add `include` directives like so:

<figure class="code">
<div class="highlight">
<pre><code>tasks:
  - include: included-playbook.yml</code></pre>
</div>
</figure>

Just like with variable include files, tasks are formatted in a flat
list in the included file. As an example, the `included-playbook.yml`
could look like this:

<figure class="code">
<div class="highlight">
<pre><code>---
- name: Add profile info for user.
  copy:
    src: example_profile
    dest: &quot;/home/{{ username }}/.profile&quot;
    owner: &quot;{{ username }}&quot;
    group: &quot;{{ username }}&quot;
    mode: 0744

- name: Add private keys for user.
  copy:
    src: &quot;{{ item.src }}&quot;
    dest: &quot;/home/.ssh/{{ item.dest }}&quot;
    owner: &quot;{{ username }}&quot;
    group: &quot;{{ username }}&quot;
    mode: 0600
  with_items: ssh_private_keys

- name: Restart example service.
  service: name=example state=restarted</code></pre>
</div>
</figure>

In this case, you'd probably want to name the file `user-config.yml`,
since it's used to configure a user account and restart some service.
Now, in this and any other playbook that provisions or configures a
server, if you want to configure a particular user's account, add the
following in your playbook's `tasks` section:

<figure class="code">
<div class="highlight">
<pre><code>- include: example-app-config.yml</code></pre>
</div>
</figure>

We used `{{ username }}` and `{{ ssh_private_keys }}` variables in this
include file instead of hard-coded values so we could make this include
file reusable. You could define the variables in your playbook's inline
variables or an included variables file, but Ansible also lets you pass
variables directly into includes using normal YAML syntax. For example:

<figure class="code">
<div class="highlight">
<pre><code>- { include: user-config.yml, username: johndoe, ssh_private_keys: [] }
- { include: user-config.yml, username: janedoe, ssh_private_keys: [] }</code></pre>
</div>
</figure>

To make the syntax more readable, you can use structured variables, like
so:

<figure class="code">
<div class="highlight">
<pre><code>- include: user-config.yml
  vars:
    username: johndoe
    ssh_private_keys:
      - { src: /path/to/johndoe/key1, dest: id_rsa }
      - { src: /path/to/johndoe/key2, dest: id_rsa_2 }
- include: user-config.yml
  vars:
    username: janedoe
    ssh_private_keys:
      - { src: /path/to/janedoe/key1, dest: id_rsa }
      - { src: /path/to/janedoe/key2, dest: id_rsa_2 }</code></pre>
</div>
</figure>

Include files can even include other files, so you could have something
like the following:

<figure class="code">
<div class="highlight">
<pre><code>tasks:
  - include: user-config.yml</code></pre>
</div>
</figure>

*inside `user-config.yml`*

<figure class="code">
<div class="highlight">
<pre><code>- include: ssh-setup.yml</code></pre>
</div>
</figure>

#### Dynamic includes {#chap08.xhtml_leanpub-auto-dynamic-includes}

Until Ansible 2.0.0, includes were processed when your playbook run
started, so you couldn't do things like load a particular include when
some condition was met. Ansible 2.0.0 evaluates includes during playbook
execution, so you can do something like the following:

<figure class="code">
<div class="highlight">
<pre><code># Include extra tasks file, only if it&#39;s present at runtime.
- name: Check if extra_tasks.yml is present.
  stat: path=extras/extra-tasks.yml
  register: extra_tasks_file
  connection: local

- include: tasks/extra-tasks.yml
  when: extra_tasks_file.stat.exists</code></pre>
</div>
</figure>

If the file `tasks/extra-tasks.yml` is not present, Ansible skips the
`include`. You can even use a `with_items` loop (or any other `with_*`
loop) with includes. Includes evaluated during playback execution can
make your playbooks much more flexible!

#### Handler includes {#chap08.xhtml_leanpub-auto-handler-includes}

Handlers can be included just like tasks, within a playbook's `handlers`
section. For example:

<figure class="code">
<div class="highlight">
<pre><code>handlers:
  - include: included-handlers.yml</code></pre>
</div>
</figure>

This can be helpful in limiting the noise in your main playbook, since
handlers are usually used for things like restarting services or loading
a configuration, and can distract from the playbook's primary purpose.

#### Playbook includes {#chap08.xhtml_leanpub-auto-playbook-includes}

Playbooks can even be included in other playbooks, using the same
`include` syntax in the top level of your playbook. For example, if you
have two playbooks---one to set up your webservers (`web.yml`), and one
to set up your database servers (`db.yml`), you could use the following
playbook to run both at the same time:

<figure class="code">
<div class="highlight">
<pre><code>- hosts: all
  remote_user: root
  
  tasks:
    [...]
  
- include: web.yml
- include: db.yml</code></pre>
</div>
</figure>

This way, you can create playbooks to configure all the servers in your
infrastructure, then create a master playbook that includes each of the
individual playbooks. When you want to initialize your infrastructure,
make changes across your entire fleet of servers, or check to make sure
their configuration matches your playbook definitions, you can run one
`ansible-playbook` command!

#### Complete includes example {#chap08.xhtml_leanpub-auto-complete-includes-example}

What if I told you we could remake the 137-line Drupal LAMP server
playbook from Chapter 4 in just 21 lines? With includes, it's easy; just
break out each of the sets of tasks into their own include files, and
you'll end up with a main playbook like this:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 ---
 2 - hosts: all
 3 
 4   vars_files:
 5     - vars.yml
 6 
 7   pre_tasks:
 8     - name: Update apt cache if needed.
 9       apt: update_cache=yes cache_valid_time=3600
10 
11   handlers:
12     - include: handlers/handlers.yml
13 
14   tasks:
15     - include: tasks/common.yml
16     - include: tasks/apache.yml
17     - include: tasks/php.yml
18     - include: tasks/mysql.yml
19     - include: tasks/composer.yml
20     - include: tasks/drush.yml
21     - include: tasks/drupal.yml</code></pre>
</div>
</figure>

All you need to do is create two new folders in the same folder where
you saved the Drupal `playbook.yml` file, `handlers` and `tasks`, then
create files inside for each section of the playbook.

For example, inside `handlers/handlers.yml`, you'd have:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - name: restart apache
3   service: name=apache2 state=restarted</code></pre>
</div>
</figure>

And inside `tasks/drush.yml`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 ---
 2 - name: Check out drush master branch.
 3   git:
 4     repo: https://github.com/drush-ops/drush.git
 5     dest: /opt/drush
 6 
 7 - name: Install Drush dependencies with Composer.&quot;
 8   shell: &gt;
 9     /usr/local/bin/composer install
10     chdir=/opt/drush
11     creates=/opt/drush/vendor/autoload.php
12 
13 - name: Create drush bin symlink.
14   file:
15     src: /opt/drush/drush
16     dest: /usr/local/bin/drush
17     state: link</code></pre>
</div>
</figure>

Separating all the tasks into separate includes files means you'll have
more files to manage for your playbook, but it helps keep the main
playbook more compact. It's easier to see all the installation and
configuration steps the playbook contains, and it separates tasks into
individual, maintainable groupings. Instead of having to browse one
playbook with twenty-three separate tasks, you now maintain eight
included files with two to five tasks, each.

It's much easier to maintain small groupings of related tasks than one
long playbook. However, there's no reason to try to *start* writing a
playbook with lots of individual includes. Most of the time, it's best
to start with a monolithic playbook while you're working on the setup
and configuration details, then move sets of tasks out to included files
after you start seeing logical groupings.

You can also use tags (demonstrated in the previous chapter) to limit
the playbook run to a certain include file. Using the above example, if
you wanted to add a 'drush' tag to the included drush file (so you could
run `ansible-playbook playbook.yml --tags=drush` and only run the drush
tasks), you can change line 20 to the following:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>20 - include: tasks/drush.yml tags=drush</code></pre>
</div>
</figure>

<aside class="tip blurb">

You can find the entire example Drupal LAMP server playbook using
include files in this book's code repository at
<https://github.com/geerlingguy/ansible-for-devops>, in the `includes`
directory.

</aside>
<aside class="warning blurb">

You can't use variables for task include file names (like you could with
`include_vars` directives, e.g.
`include_vars: "{{ ansible_os_family }}.yml"` as a task, or with
`vars_files`). There's usually a better way than conditional task
includes to accomplish conditional task inclusion using a different
playbook structure, or roles, which we will discuss next.

</aside>

### Roles {#chap08.xhtml_leanpub-auto-roles}

Including playbooks inside other playbooks makes your playbook
organization a little more sane, but once you start wrapping up your
entire infrastructure's configuration in playbooks, you might end up
with something resembling Russian nesting dolls.

Wouldn't it be nice if there were a way to take bits of related
configuration, and package them together nicely? Additionally, what if
we could take these packages (often configuring the same thing on many
different servers) and make them flexible so that we can use the same
package throughout our infrastructure, with slightly different settings
on individual servers or groups of servers?

Ansible Roles can do all that and more!

Let's dive into what makes an Ansible role by taking one of the playbook
examples from Chapter 4 and splitting it into a more flexible structure
using roles.

#### Role scaffolding {#chap08.xhtml_leanpub-auto-role-scaffolding}

Instead of requiring you to explicitly include certain files and
playbooks in a role, Ansible automatically includes any `main.yml` files
inside specific directories that make up the role.

There are only two directories required to make a working Ansible role:

<figure class="code">
<div class="highlight">
<pre><code>role_name/
  meta/
  tasks/</code></pre>
</div>
</figure>

If you create a directory structure like the one shown above, with a
`main.yml` file in each directory, Ansible will run all the tasks
defined in `tasks/main.yml` if you call the role from your playbook
using the following syntax:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3   roles:
4     - role_name</code></pre>
</div>
</figure>

Your roles can live in a couple different places: the default global
Ansible role path (configurable in `/etc/ansible/ansible.cfg`), or a
`roles` folder in the same directory as your main playbook file.

<aside class="information blurb">

Another simple way to build the scaffolding for a role is to use the
command: `ansible-galaxy init role_name`. Running this command creates
an example role in the current working directory, which you can modify
to suit your needs. Using the `init` command also ensures the role is
structured correctly in case you want to someday contribute the role to
Ansible Galaxy.

</aside>

#### Building your first role {#chap08.xhtml_leanpub-auto-building-your-first-role}

Let's clean up the Node.js server example from Chapter four, and break
out one of the main parts of the configuration---installing Node.js and
any required npm modules.

Create a `roles` folder in the same directory as the main `playbook.yml`
file like, we created in Chapter 4's first example. Inside the `roles`
folder, create a new folder: `nodejs` (which will be our role's name).
Create two folders inside the `nodejs` role directory: `meta` and
`tasks`.

Inside the `meta` folder, add a simple `main.yml` file with the
following contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 dependencies: []</code></pre>
</div>
</figure>

The meta information for your role is defined in this file. In basic
examples and simple roles, you just need to list any role dependencies
(other roles that are required to be run before the current role can do
its work). You can add more to this file to describe your role to
Ansible and to Ansible Galaxy, but we'll dive deeper into meta
information later. For now, save the file and head over to the `tasks`
folder.

Create a `main.yml` file in this folder, and add the following contents
(basically copying and pasting the configuration from the Chapter 4
example):

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - name: Install Node.js (npm plus all its dependencies).
3   yum: name=npm state=present enablerepo=epel
4 
5 - name: Install forever module (to run our Node.js app).
6   npm: name=forever global=yes state=latest</code></pre>
</div>
</figure>

The Node.js directory structure should now look like the following:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 nodejs-app/
 2   app/
 3     app.js
 4     package.json
 5   playbook.yml
 6   roles/
 7     nodejs/
 8       meta/
 9         main.yml
10       tasks/
11         main.yml</code></pre>
</div>
</figure>

You now have a complete Ansible role that you can use in your node.js
server configuration playbook. Delete the Node.js app installation lines
from `playbook.yml`, and reformat the playbook so the other tasks run
first (in a `pre_tasks:` section instead of `tasks:`), then the role,
then the rest of the tasks (in the main `tasks:` section). Something
like:

<figure class="code">
<div class="highlight">
<pre><code>pre_tasks:
  # EPEL/GPG setup, firewall configuration...

roles:
  - nodejs

tasks:
  # Node.js app deployment tasks...</code></pre>
</div>
</figure>

<aside class="information blurb">

You can view the full example of this playbook in the
[ansible-for-devops code
repository](https://github.com/geerlingguy/ansible-for-devops/blob/master/nodejs-role/).

</aside>

Once you finish reformatting the main playbook, everything will run
exactly the same during an `ansible-playbook` run, with the exception of
the tasks inside the `nodejs` role being prefixed with
`nodejs | [Task name here]`.

This little bit of extra data shown during playbook runs is useful
because it automatically prefixes tasks with the role that provides
them, without you having to add in descriptions as part of the `name`
values of the tasks.

Our role isn't all that helpful at this point, though, because it still
does only one thing, and it's not really flexible enough to be used on
other servers that might need different Node.js modules to be installed.

#### More flexibility with role vars and defaults {#chap08.xhtml_leanpub-auto-more-flexibility-with-role-vars-and-defaults}

To make our role more flexible, we can make it use a list of npm modules
instead of a hardcoded value, then allow playbooks using the role to
provide their own module list variable to override our role's default
list.

When running a role's tasks, Ansible picks up variables defined in a
role's `vars/main.yml` file and `defaults/main.yml` (I'll get to the
differences between the two later), but will allow your playbooks to
override the defaults or other role-provided variables if you want.

Modify the `tasks/main.yml` file to use a list variable and iterate
through the list to install as many packages as your playbook wants:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - name: Install Node.js (npm plus all its dependencies).
3   yum: name=npm state=present enablerepo=epel
4 
5 - name: Install npm modules required by our app.
6   npm: name={{ item }} global=yes state=latest
7   with_items: node_npm_modules</code></pre>
</div>
</figure>

Let's provide a sane default for the new `node_npm_modules` variable in
`defaults/main.yml`:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 node_npm_modules:
3   - forever</code></pre>
</div>
</figure>

Now, if you run the playbook as-is, it will still do the exact same
thing---install the `forever` module. But since the role is more
flexible, we could create a new playbook like our first, but add a
variable (either in a `vars` section or in an included file via
`vars_files`) to override the default, like so:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 node_npm_modules:
2   - forever
3   - async
4   - request</code></pre>
</div>
</figure>

When you run the playbook with this custom variable (we didn't change
*anything* with our `nodejs` role), all three of the above npm modules
will be installed.

Hopefully you're beginning to see how this can be powerful!

Imagine if you had a playbook structure like:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: appservers
3   roles:
4     - yum-repo-setup
5     - firewall
6     - nodejs
7     - app-deploy</code></pre>
</div>
</figure>

Each one of the roles lives in its own isolated world, and can be shared
with other servers and groups of servers in your infrastructure.

- A `yum-repo-setup` role could enable certain repositories and import
  their GPG keys.
- A `firewall` role could have per-server or per-inventory-group options
  for ports and services to allow or deny.
- An `app-deploy` role could deploy your app to a directory
  (configurable per-server) and set certain app options per-server or
  per-group.

These things are easy to manage when you have small bits of
functionality separated into different roles. Instead of managing 100+
lines of playbook tasks, and manually prefixing every `name:` with
something like "Common \|" or "App Deploy \|", you now manage a few
roles with 10-20 lines of YAML each.

On top of that, when you're building your main playbooks, they can be
extremely simple (like the above example), enabling you to see
*everything* being configured and deployed on a particular server
without scrolling through dozens of included playbook files and hundreds
of tasks.

<aside class="tip blurb">

**Variable precedence**: Note that Ansible handles variables placed in
included files in `defaults` with less precedence than those placed in
`vars`. If you have certain variables you need to allow hosts/playbooks
to easily override, you should probably put them into `defaults`. If
they are common variables that should almost always be the values
defined in your role, put them into `vars`. For more on variable
precedence, see the aptly-named "Variable Precedence" section in the
previous chapter.

</aside>

#### Other role parts: handlers, files, and templates {#chap08.xhtml_leanpub-auto-other-role-parts-handlers-files-and-templates}

##### Handlers {#chap08.xhtml_leanpub-auto-handlers-1}

In one of the prior examples, we introduced handlers---tasks that could
be called via the `notify` option after any playbook task resulted in a
change---and an example handler for restarting Apache was given:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 handlers:
2   - name: restart apache
3     service: name=apache2 state=restarted</code></pre>
</div>
</figure>

In Ansible roles, handlers are first-class citizens, alongside tasks,
variables, and other configuration. You can store handlers directly
inside a `main.yml` file inside a role's `handlers` directory. So if we
had a role for Apache configuration, our `handlers/main.yml` file could
look like this:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - name: restart apache
3   command: service apache2 restart</code></pre>
</div>
</figure>

You can call handlers defined in a role's handlers folder just like
those included directly in your playbooks (e.g.
`notify: restart apache`).

##### Files and Templates {#chap08.xhtml_leanpub-auto-files-and-templates}

For the following examples, let's assume our role is structured with
files and templates inside `files` and `templates` directories,
respectively:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 roles/
 2   example/
 3     files/
 4       example.conf
 5     meta/
 6       main.yml
 7     templates/
 8       example.xml.j2
 9     tasks/
10       main.yml</code></pre>
</div>
</figure>

when copying a file directly to the server, add the filename or the full
path from within a role's `files` directory, like so:

<figure class="code">
<div class="highlight">
<pre><code>- name: Copy configuration file to server directly.
  copy: &gt;
    src=example.conf
    dest=/etc/myapp/example.conf
    mode=644</code></pre>
</div>
</figure>

Similarly, when specifying a template, add the filename or the full path
from within a role's `templates` directory, like so:

<figure class="code">
<div class="highlight">
<pre><code>- name: Copy configuration file to server using a template.
  template: &gt;
    src=example.xml.j2
    dest=/etc/myapp/example.xml
    mode=644</code></pre>
</div>
</figure>

The `copy` module copies files from within the module's `files` folder,
and the `template` module runs given template files through the Jinja2
templating engine, merging in any variables available during your
playbook run before copying the file to the server.

#### Organizing more complex and cross-platform roles {#chap08.xhtml_leanpub-auto-organizing-more-complex-and-cross-platform-roles}

For simple package installation and configuration roles, you can get by
with placing all tasks, variables, and handlers directly in the
respective `main.yml` file Ansible automatically loads. But you can also
*include* other files from within a role's `main.yml` files if needed.

As a rule of thumb, I keep my playbook and role task files under 100
lines of YAML if at all possible. It's easier to keep the entire set of
tasks in my head while making changes or fixing bugs. If I start nearing
that limit, I usually split the tasks into logical groupings, and
include files from the `main.yml` file.

Let's take a look at the way my `geerlingguy.apache` role is set up
(it's [available on Ansible
Galaxy](https://galaxy.ansible.com/list#/roles/428) and can be
downloaded to your roles directory with the command
`ansible-galaxy install geerlingguy.apache`; we'll discuss Ansible
Galaxy itself later).

Initially, the role's main `tasks/main.yml` file looked something like
the following (generally speaking):

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 - name: Ensure Apache is installed (via apt).
2 
3 - name: Configure Apache with lineinfile.
4 
5 - name: Enable Apache modules.</code></pre>
</div>
</figure>

Soon after creating the role, though, I wanted to make the role work
with both Debian and RedHat hosts. I could've added two sets of tasks in
the `main.yml` file, resulting in twice the number of tasks and a bunch
of extra `when` statements:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 - name: Ensure Apache is installed (via apt).
 2   when: ansible_os_family == &#39;Debian&#39;
 3 
 4 - name: Ensure Apache is installed (via yum).
 5   when: ansible_os_family == &#39;RedHat&#39;
 6 
 7 - name: Configure Apache with lineinfile (Debian).
 8   when: ansible_os_family == &#39;Debian&#39;
 9 
10 - name: Configure Apache with lineinfile (Redhat).
11   when: ansible_os_family == &#39;RedHat&#39;
12 
13 - name: Enable Apache modules (Debian).
14   when: ansible_os_family == &#39;Debian&#39;
15 
16 - name: Other OS-agnostic tasks...</code></pre>
</div>
</figure>

If I had gone this route, and continued with the rest of the playbook
tasks in one file, I would've quickly surpassed my informal 100-line
limit. So I chose to use includes in my main tasks file:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 - name: Include OS-specific variables.
 2   include_vars: &quot;{{ ansible_os_family }}.yml&quot;
 3 
 4 - include: setup-RedHat.yml
 5   when: ansible_os_family == &#39;RedHat&#39;
 6 
 7 - include: setup-Debian.yml
 8   when: ansible_os_family == &#39;Debian&#39;
 9 
10 - name: Other OS-agnostic tasks...</code></pre>
</div>
</figure>

Two important things to notice about this style of distribution-specific
inclusion:

1.  When including vars files (with `include_vars`), you can actually
    *use variables in the name of the file*. This is handy in many
    situtations, and here we're including a vars file in the format
    `distribution_name.yml`. For our purposes, since the role will be
    used on Debian and RedHat-based hosts, we can create `Debian.yml`
    and `RedHat.yml` files in our role's `defaults` and `vars` folders,
    and put distribution-specific variables there.
2.  When including playbook files (with `include`), you can't use
    variables in the name of the file, but you can do the next best
    thing: include the files by name explicitly, and use a condition to
    tell Ansible whether to run the tasks inside (the `when` condition
    will be applied to every task inside the included playbook).

After setting things up this way, I put RedHat and CentOS-specific tasks
(like `yum` tasks) into `tasks/setup-RedHat.yml`, and Debian and
Ubuntu-specific tasks (like `apt` tasks) into `tasks/setup-Debian.yml`.
There are other ways of making roles work cross-platform, but using
distribution-specific variables files and included playbooks is one of
the simplest.

Now this Apache role can be used across different distributions, and
with clever usage of variables in tasks and in configuration templates,
it can be used in a wide variety of infrastructure that needs Apache
installed.

### Ansible Galaxy {#chap08.xhtml_leanpub-auto-ansible-galaxy}

Ansible roles are powerful and flexible; they allow you to encapsulate
sets of configuration and deployable units of playbooks, variables,
templates, and other files, so you can easily reuse them across
different servers.

It's annoying to have to start from scratch every time, though; wouldn't
it be better if people could share roles for commonly-installed
applications and services?

Enter [Ansible Galaxy](https://galaxy.ansible.com/).

Ansible Galaxy, or just 'Galaxy', is a repository of
community-contributed roles for common Ansible content. There are
already hundreds of roles available which can configure and deploy
common applications, and they're all available through the
`ansible-galaxy` command, introduced in Ansible 1.4.2.

Galaxy offers the ability to add, download, and rate roles. With an
account, you can contribute your own roles or rate others' roles (though
you don't need an account to use roles).

#### Getting roles from Galaxy {#chap08.xhtml_leanpub-auto-getting-roles-from-galaxy}

One of the primary functions of the `ansible-galaxy` command is
retrieving roles from Galaxy. Roles must be downloaded before they can
be used in playbooks.

Remember the basic LAMP (Linux, Apache, MySQL and PHP) server we
installed earlier in the book? Let's create it again, but this time,
using a few roles from Galaxy:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-galaxy install geerlingguy.apache geerlingguy.mysql geerlingguy.php</code></pre>
</div>
</figure>

<aside class="tip blurb">

The latest version or a role will be downloaded if no version is
specified. To specify a version, add the version after the role name,
for example: `$ ansible-galaxy install geerlingguy.apache,1.0.0`.

</aside>
<aside class="information blurb">

Ansible Galaxy is still evolving rapidly, and has seen many
improvements. There are a few areas where Galaxy could use some
improvement (like browsing for roles by Operating System in the online
interface, or automatically downloading roles that are included in
playbooks), but most of these little bugs or rough spots will be fixed
in time. Please check Ansible Galaxy's
[About](https://galaxy.ansible.com/intro) page and stay tuned to
Ansible's blog for the latest updates.

</aside>

##### Using role requirements files to manage dependencies {#chap08.xhtml_leanpub-auto-using-role-requirements-files-to-manage-dependencies}

If your infrastructure configuration requires five, ten, fifteen or more
Ansible roles, installing them all via `ansible-galaxy install` commands
can be exhausting. Additionally, if you host roles internally (e.g. via
an internal Git or Mercurial repository), you can't install the roles
through Ansible Galaxy. You can, however, pass the `ansible-galaxy`
command a "requirements" file with the `-r` option to automatically
download all dependencies.

Ansible allows a simple `.txt` format that is very basic (though this
format is deprecated and may be removed), but you should generally use
the more standard and expressive YAML format, which allows you to
install roles from Ansible Galaxy, GitHub, an HTTP download, BitBucket,
or your own repository. It also allows you to specify the path into
which the roles should be downloaded. An example `requirements.yml` file
looks like this:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code> 1 ---
 2 # From Ansible Galaxy, latest version.
 3 - src: geerlingguy.firewall
 4 
 5 # From Ansible Galaxy, specifying the version.
 6 - src: geerlingguy.php
 7   version: 1.4.1
 8 
 9 # From GitHub, into a particular path, with a custom name and version.
10 - src: https://github.com/geerlingguy/ansible-role-passenger
11   path: /etc/ansible/roles/
12   name: passenger
13   version: 1.0.2
14 
15 # From a web server, with a custom name.
16 - src: https://www.example.com/ansible/roles/my-role-name.tar.gz
17   name: my-role</code></pre>
</div>
</figure>

To install the roles defined in a requirements file, use the command
`ansible-galaxy install -r requirements.yml`. For more documentation on
Ansible requirements files, see the official documentation: [Advanced
Control over Role Requirements
Files](http://docs.ansible.com/ansible/galaxy.html#advanced-control-over-role-requirements-files).

#### A LAMP server in eight lines of YAML {#chap08.xhtml_leanpub-auto-a-lamp-server-in-eight-lines-of-yaml}

With the Apache, MySQL, and PHP roles installed, we can quickly create a
LAMP server. This example assumes you already have a CentOS-based linux
VM or server booted and can connect to it or run Ansible as a
provisioner via Vagrant on it, and that you've run the
`ansible-galaxy install` command above to download the required roles.

First, create an Ansible playbook named `lamp.yml` with the following
contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3   sudo: true
4 
5   roles:
6     - geerlingguy.mysql
7     - geerlingguy.apache
8     - geerlingguy.php</code></pre>
</div>
</figure>

Now, run the playbook against a host:

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-playbook -i path/to/custom-inventory lamp.yml</code></pre>
</div>
</figure>

After a few minutes, an entire LAMP server should be set up and running.
If you add in a few variables, you can configure virtualhosts, PHP
configuration options, MySQL server settings, etc.

We've effectively reduced about thirty lines of YAML (from previous
examples dealing with LAMP or LAMP-like servers) down to three.
Obviously, the roles have extra code in them, but the power here is in
abstraction. Since most companies have many servers using similar
software, but with slightly different configurations, having
centralized, flexible roles saves a lot of repetition.

You could think of Galaxy roles as glorified packages; they not only
install software, but they configure it *exactly* how you want it, every
time, with minimal adjustment. Additionally, many of these roles work
across different flavors of Linux and UNIX, so you have better
configuration portability!

#### A Solr server in eight lines of YAML {#chap08.xhtml_leanpub-auto-a-solr-server-in-eight-lines-of-yaml}

Let's grab a few more roles and build an Apache Solr search server,
which requires Java and Apache Tomcat to be installed and configured.

<figure class="code">
<div class="highlight">
<pre><code>$ ansible-galaxy install geerlingguy.java geerlingguy.tomcat6 geerlingguy.solr</code></pre>
</div>
</figure>

Then create a playbook named `solr.yml` with the following contents:

<figure class="code">
<div class="highlight">
<pre class="lineno"><code>1 ---
2 - hosts: all
3   sudo: yes
4 
5   roles:
6     - geerlingguy.java
7     - geerlingguy.tomcat6
8     - geerlingguy.solr</code></pre>
</div>
</figure>

Now we have a fully-functional Solr server, and we could add some
variables to configure it exactly how we want, by using a non-default
port, or changing the memory allocation for Tomcat6.

I could've also left out the `java` and `tomcat6` roles, since they'll
be automatically picked up during installation of the `geerlingguy.solr`
role (they're listed in the `solr` role's dependencies).

A role's page on the Ansible Galaxy website highlights available
variables for setting things like what version of Solr to install, where
to install it, etc. For an example, view the [geerlingguy.solr Galaxy
page](https://galaxy.ansible.com/list#/roles/445).

You can build a wide variety of servers with minimal effort with
existing contributed roles on Galaxy. Instead of having to maintain
lengthy playbooks and roles unique to each server, Galaxy lets you build
a list of the required roles, and a few variables that set up the
servers with the proper versions and paths. Configuration management
with Ansible Galaxy becomes *true* configuration management---you get to
spend more time managing your server's configuration, and less time on
packaging and building individual services!

#### Helpful Galaxy commands {#chap08.xhtml_leanpub-auto-helpful-galaxy-commands}

Some other helpful `ansible-galaxy` commands you might use from time to
time:

- `ansible-galaxy list` displays a list of installed roles, with version
  numbers
- `ansible-galaxy remove [role]` removes an installed role
- `ansible-galaxy init` can be used to create a role template suitable
  for submission to Ansible Galaxy

You can configure the default path where Ansible roles will be
downloaded by editing your `ansible.cfg` configuration file (normally
located in `/etc/ansible/ansible.cfg`), and setting a `roles_path` in
the `[defaults]` section.

#### Contributing to Ansible Galaxy {#chap08.xhtml_leanpub-auto-contributing-to-ansible-galaxy}

If you've been working on some useful Ansible roles, and you'd like to
share them with others, all you need to do is make sure they follow
Ansible Galaxy's basic template (especially within the `meta/main.yml`
and `README.md` files). To get started, use `ansible-galaxy init` to
generate a basic Galaxy template, and make your own role match the
Galaxy template's structure.

Then push your role up to a new project on GitHub (I usually name my
Galaxy roles like `ansible-role-[rolename]`, so I can easily see them
when browsing my repos on GitHub), and add a new role while logged into
galaxy.ansible.com.

### Summary {#chap08.xhtml_leanpub-auto-summary-5}

Using includes and Ansible roles organizes Playbooks and makes them
maintainable. This chapter introduced different ways of using `include`,
the power and flexible structure of roles, and how you can utilize
Ansible Galaxy, the community repository of configurable Ansible roles
that do just about anything.

<figure class="code">
<div class="highlight">
<pre><code> _________________________________________
/ When the only tool you own is a hammer, \
| every problem begins to resemble a      |
\ nail. (Abraham Maslow)                  /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||</code></pre>
</div>
</figure>
:::

[]{#chap09.xhtml}

::: {}