Advanced Features in Ansible Playbooks
Introduction
There are a number of features unique to Ansible playbooks which provide robust functionality. This exercise explores many of these features in a practical scenario of deploying a web server. Most notably, this exercise deals with confidential data in an Ansible vault and working with tags in Ansible playbooks.

You must create a modular playbook used for webserver management. Create a playbook called /home/ansible/webserver.yml that meets the following requirements:

On the host group webservers:

Deploy httpd.
You can assume all necessary firewall rules have been deployed.
Start and enable httpd.
Configure virtual host using the provided templates in /home/ansible/vhost.conf.j2 and /home/ansible/htpasswd.j2.
Note: The template references a variable defined in /home/ansible/confidential which must be included as a variable file in your playbook.
Use ansible vault to secure /home/ansible/confidential with password "I love ansible".
Run the data job stored in /opt/data-job.sh on each node in webservers asynchronously, without polling for status.
Create tags for the following tasks:
base-install for httpd installation and service configuration.
vhost for virtual host deployment.
data-job to execute the asynchronous data job.
The Ansible control node has been configured for you and each webserver has already been configured for use with Ansible. The default inventory has been configured to include the webservers group and sample hosts.

Solution
Begin by logging in to the Ansible control node using the credentials on the hands-on lab page:

ssh cloud_user@PUBLIC_IP_ADDRESS
Become the ansible user:

sudo -i -u ansible	
Use ansible-vault to protect the confidential information
Use ansible-vault to encrypt /home/ansible/confidential to protect the confidential information stored within using the password "I love ansible".

Run ansible-vault encrypt /home/ansible/confidential and supply the password "I love ansible".
Create a playbook that deploys httpd on webservers
Create a playbook in /home/ansible/webserver.yml that deploys httpd on webservers. It should be tagged with base-install and contain a handler that restarts the httpd daemon that is flagged by both installation and service manipulation for httpd.

Create the file /home/ansible/webserver.yml and add the following content:
vim /home/ansible/webserver.yml
	
---
- hosts: webservers
  become: yes
  vars_files:
    - /home/ansible/confidential
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: latest
      notify: httpd service
      tags:
        - base-install
  handlers:
    - name: Restart and enable httpd
      service:
        name: httpd
        state: restarted
        enabled: yes
      listen: httpd service
Deploy the templates stored on the control node to the webservers group
Configure /home/ansible/webserver.yml to deploy the templates /home/ansible/vhost.conf.j2 and /home/ansible/htpasswd.j2 stored on the control node to the webservers group. httpd must restart on config change. The tasks should be tagged vhost.

Add the following text to /home/ansible/webserver.yml just before the handler section:
vim /home/ansible/webserver.yml

    - name: configure virtual host
      template:
        src: /home/ansible/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
      notify: httpd service
      tags:
        - vhost
    - name: configure site auth
      template:
        src: /home/ansible/htpasswd.j2
        dest: /etc/httpd/conf/htpasswd
      notify: httpd service
      tags:
        - vhost
Asynchronously execute data-job on webservers
Configure /home/ansible/webserver.yml to asynchronously execute /opt/data-job.sh located on webservers with a timeout of 600 seconds and no polling. The task should be tagged with data-job.

Add the following text to /home/ansible/webserver.yml just before the handler section:
vim /home/ansible/webserver.yml

    - name: run data job
      command: /opt/data-job.sh
      async: 600
      poll: 0
      tags:
        - data-job
Your complete file should look similar to the below:
---
- hosts: webservers
  become: yes
  vars_files:
    - /home/ansible/confidential
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: latest
      notify: httpd service
      tags:
        - base-install
    - name: configure virtual host
      template:
        src: /home/ansible/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
      notify: httpd service
      tags:
        - vhost
    - name: configure site auth
      template:
        src: /home/ansible/htpasswd.j2
        dest: /etc/httpd/conf/htpasswd
      notify: httpd service
      tags:
        - vhost
    - name: run data job
      command: /opt/data-job.sh
      async: 600
      poll: 0
      tags:
        - data-job
  handlers:
    - name: Restart and enable httpd
      service:
        name: httpd
        state: restarted
        enabled: yes
      listen: httpd service
Execute playbook to verify your playbook works correctly
Execute playbook /home/ansible/webserver.yml to verify your playbook works correctly.

Run ansible-playbook --ask-vault-pass /home/ansible/webserver.yml from the control node and provide the vault password "I love ansible".
Conclusion
Congratulations, you've completed this hands-on lab!