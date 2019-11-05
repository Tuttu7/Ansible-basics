
###  Ansible Installation step
```
amazon-linux-extras install ansible2 -y
```
### Checking ansible version
```
ansible --version
```
### Default Configuration File Of Ansible
```
/etc/ansible/ansible.cfg
```
### Default Inventory File 
```
/etc/ansible/hosts
```
### Creating an additional Inventory File
```
vi inventoryfile.txt

[amazon]

172.31.44.235  ansible_port=22 ansible_user='ec2-user' ansible_ssh_private_key_file='testvm.pem'
```
### Checking Connectivity to the client ec2 instance
```
[root@ip-172-31-43-140 ec2-user]# ansible -i inventory.txt amazon -m ping
172.31.44.235 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```
### Installing Httpd Using Ansible 
```
[root@ip-172-31-43-140 ec2-user]# ansible -i inventory.txt amazon -m yum -a 'name=httpd state=present' 
 [WARNING]: Platform linux on host 172.31.44.235 is using the discovered Python interpreter at /usr/bin/python, but future installation of another
Python interpreter could change this. See https://docs.ansible.com/ansible/2.8/reference_appendices/interpreter_discovery.html for more information.

172.31.44.235 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "changes": {
        "installed": [
            "httpd"
        ]
    }, 
    "msg": "You need to be root to perform this command.\n", 
    "rc": 1, 
    "results": [
        "Loaded plugins: extras_suggestions, langpacks, priorities, update-motd\n"
    ]
}
```
### We need to specify the -b (privilege escalation) to sort out this error :
```
ansible -i inventory.txt amazon  -b -m yum -a 'name=httpd state=present' 

172.31.44.235 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "changes": {
        "installed": [
            "httpd"
        ]
    }, 
    "msg": "", 
    "rc": 0, 
    "results": [
```
### Restating/Enabling httpd service

```
[root@ip-172-31-43-140 ec2-user]#ansible -i inventory.txt amazon -b -m service -a 'name=httpd state=restarted enabled=true'

172.31.44.235 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "enabled": true, 
    "name": "httpd", 
    "state": "started", 
    "status": {
```
### Copying website
```
cd /tmp ; wget https://www.tooplate.com/zip-templates/2116_blugoon.zip ; unzip 2116_blugoon.zip ; cd ; mkdir content ;  cp -r /tmp/2116_blugoon/*  content/ 

[root@ip-172-31-43-140 ec2-user]# ansible -i inventory.txt amazon -b -m copy -a 'src=/root/content/ dest=/var/www/html/'
172.31.44.235 | CHANGED => {
    "changed": true, 
    "dest": "/var/www/html/", 
    "src": "/root/content/"
}
```

### Removing httpd package
```
 [root@ip-172-31-43-140 ec2-user]#ansible -i inventory.txt amazon -b -m yum -a 'name=httpd state=absent' 
 
 172.31.44.235 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "changes": {
        "removed": [
            "httpd"
        ]
    }, 
```

### Removing document root
```
[root@ip-172-31-43-140 ec2-user]# ansible -i inventory.txt amazon -b -m shell -a 'rm -rf /var/www/html/* /etc/httpd/*'
 
 172.31.44.235 | CHANGED | rc=0 >>
 ```
 ### Creating Playbook to Install httpd and host site
 ```
 vi install-httpd.yml

---
- name: "Installing Httpd And Hosting Website"
  hosts: amazon
  become: yes
  tasks:

    - name: "Installing httpd"
      yum:
        name: httpd
        state: present
    
    - name: "Restarting/Enabling httpd"
      service:
        name: httpd
        state: restarted
        enabled: true
            
    - name: "Copying Website Content"
      copy:
        src: ./content/
        dest: /var/www/html/
        owner: apache
        group: apache
```
### Syntax Checking
```
[root@ip-172-31-43-140 ec2-user]# ansible-playbook -i inventory.txt install-httpd.yml --syntax-check
playbook: install-httpd.yml
```
### Running the command
```
[root@ip-172-31-43-140 ec2-user]# ansible-playbook -i inventory.txt install-httpd.yml

PLAY [Installing Httpd And Hosting Website] ******************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
ok: [172.31.44.235]

TASK [Installing httpd] **************************************************************************************************
ok: [172.31.44.235]

TASK [Restarting/Enabling httpd] *****************************************************************************************
changed: [172.31.44.235]

TASK [Copying Website Content] *******************************************************************************************
changed: [172.31.44.235]

PLAY RECAP ***************************************************************************************************************
172.31.44.235              : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

###  Editing the Above playbook to install php and creating index.php
```
vi install-httpd.yml

- name: "Installing Httpd And Hosting Website"
  hosts: amazon
  become: yes
  tasks:

    - name: "Installing httpd"
      yum:
        name: httpd,php
        state: present
    
    - name: "Creating index.php"
      copy:
        content: "<?php phpinfo(); ?>"
        dest: /var/www/html/index.php
            
            
    - name: "Restarting/Enabling httpd"
      service:
        name: httpd
        state: restarted
        enabled: true
            
    - name: "Copying Website Content"
      copy:
        src: ./content/
        dest: /var/www/html/
        owner: apache
        group: apache
```
### Running the command 

```
[root@ip-172-31-43-140 ec2-user]# ansible-playbook -i inventory.txt install-httpd.yml 

PLAY [Installing Httpd And Hosting Website] ******************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
ok: [172.31.44.235]

TASK [Installing httpd] **************************************************************************************************
ok: [172.31.44.235]

TASK [Creating index.php] ************************************************************************************************
ok: [172.31.44.235]

TASK [Restarting/Enabling httpd] *****************************************************************************************
changed: [172.31.44.235]

TASK [Copying Website Content] *******************************************************************************************
ok: [172.31.44.235]

PLAY RECAP ***************************************************************************************************************
172.31.44.235              : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```




