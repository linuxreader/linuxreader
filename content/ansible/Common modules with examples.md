+++
title = 'Common modules with examples'
description = 'Common Ansible modules with examples'
+++

**uri:**
Interacts with basic http and https web services. (Verify connectivity to a web server
+9)

Test httpd accessibility:
```yaml
uri:
  url: http://ansible1
```

Show result of the command while running the playbook:
```yaml
uri:
  url: http://ansible1
  return_content: yes
```

Show the status code that signifies the success of the request:
```yaml
uri:
  url: http://ansible1
  status_code: 200
```

**debug:**
Prints statements during execution. Used for debugging variables or expressions without stopping a playbook. 

Print out the value of the ansible_facts variable:
```yaml
debug:
  var: ansible_facts
```