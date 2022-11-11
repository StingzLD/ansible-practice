Working with Ansible Roles
This course is not approved or sponsored by Red Hat.

Introduction
Working with Ansible roles is a key concept covered on the Red Hat Certified Ansible Specialist Exam (EX407). This should not be a surprise, considering how much functionality roles provide. This exercise covers how to create a role and how to use roles within a playbook. In order to complete this exercise, you will need to have basic proficiency with several common Ansible modules and using Ansible playbooks. After completing this learning activity, you will better understand how to work with Ansible roles.

Solution
Log in to the Ansible control node as ansible using the credentials provided:

ssh ansible@<PUBLIC_IP_ADDRESS>
Note: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes.

Create a Role Called baseline in /etc/ansible/roles
Create the structure needed for the role:

cd /etc/ansible/roles/
sudo mkdir baseline && sudo chown ansible.ansible /etc/ansible/roles/baseline
mkdir /etc/ansible/roles/baseline/{templates,tasks,files}
echo "---" > baseline/tasks/main.yml
Configure the Role to Deploy the /etc/motd Template
Copy the file:

cp /home/ansible/resources/motd.j2 baseline/templates
Create a file called deploy_motd.yml:

vim baseline/tasks/deploy_motd.yml
Add the following content:

---
- template:
    src: motd.j2
    dest: /etc/motd
Save and exit with Escape followed by :wq.

Open main.yml:

vim baseline/tasks/main.yml
Add the following lines to the file:

- name: configure motd
  import_tasks: deploy_motd.yml
Save and exit with Escape followed by :wq.

Configure the Role to Install the Latest Nagios Client
Find the package we need to install by reading a text file in our home directory:

cat /home/ansible/resources/nagios_info.txt
That file tells us the package we need to install is nrpe.x86_64.

Copy the IP of the Nagios server that's in the file and paste it into a text file, as we'll need it later.

Create a file, which will install the package, called deploy_nagios.yml:

vim baseline/tasks/deploy_nagios.yml
Add the following content:

---
- yum: name=nrpe state=latest
Save and exit with Escape followed by :wq.

Open main.yml:

vim baseline/tasks/main.yml
Add the following lines to the bottom of the file:

- name: deploy nagios client
  import_tasks: deploy_nagios.yml
Save and exit with Escape followed by :wq.

Configure the Role to Add an Entry to /etc/hosts for the Nagios Server
Create a file called edit_hosts.yml:

vim baseline/tasks/edit_hosts.yml
Add the following content, substituting <IP_ADDRESS> with the Nagios server IP you copied earlier:

---
- lineinfile:
    line: "<IP_ADDRESS> nagios.example.com"
    path: /etc/hosts
Save and exit with Escape followed by :wq.

Open main.yml:

vim  baseline/tasks/main.yml
Add the following lines to the bottom of the file:

- name: edit hosts file
  import_tasks: edit_hosts.yml
Save and exit with Escape followed by :wq.

Configure the Role to Create the noc User and Deploy the Provided Public Key for the noc User on Target Systems
Copy the provided authorized_keys file to our files directory:

cp /home/ansible/resources/authorized_keys /etc/ansible/roles/baseline/files/
Create a file called deploy_noc_user.yml:

vim baseline/tasks/deploy_noc_user.yml
Add the following content:

---
- user: name=noc
- file:
     state: directory
     path: /home/noc/.ssh
     mode: 0600
     owner: noc
     group: noc
- copy:
     src: authorized_keys
     dest: /home/noc/.ssh/authorized_keys
     mode: 0600
     owner: noc
     group: noc
Save and exit with Escape followed by :wq.

Open main.yml:

vim baseline/tasks/main.yml
Add the following lines to the bottom of the file:

- name: set up noc user and key
  import_tasks: deploy_noc_user.yml
Save and exit with Escape followed by :wq.

Edit web.yml to Deploy the baseline Role
Change back to the home directory:

cd /home/ansible/
Open web.yml:

vim resources/web.yml
Edit it to match the following:

---
- hosts: webservers
  become: yes
  roles:
    - baseline
  tasks:
    - name: install httpd
      yum: name=httpd state=latest
    - name: start and enable httpd
      service: name=httpd state=started enabled=yes
Save and exit with Escape followed by :wq.

Run Your Playbook Using the Default Inventory
Deploy the playbook:

ansible-playbook resources/web.yml
Check Our Work
Log in to one of the nodes (the IP addresses are on the hands-on lab overview page):

ssh node1
We should see a new MOTD, so we know that play worked.

See if the noc user was set up:

id noc
Check to see if the nrpe package was installed:

sudo yum list nrpe
Conclusion
All these plays ran, and now we've got a playbook that we can edit when we want to keep things consistent across our webservers. Congratulations!