Learning Objectives

Create the necessary template files on the Ansible control node.

Create the files exports.j2 and etc.hosts.j2 in /home/ansible with the noted content:

/home/ansible/exports.j2

{{ share_path }} *(rw)
/home/ansible/etc.hosts.j2

127.0.0.1	localhost {{ ansible_hostname}}
{{ nfs_ip }}	{{ nfs_hostname }}

Create a playbook for the server 'nfs' in the Ansible inventory.

Create a playbook in /home/ansible/nfs.yml that meets the provided specification for the server nfs in the ansible inventory.

Create a playbook in /home/ansible/nfs.yml with the following content: - hosts: nfs become: yes vars: share_path: /mnt/nfsroot tasks: - name: install nfs yum: name: nfs-utils state: latest - name: start and enable nfs-server service: name: nfs-server state: started enabled: yes - name: configure exports template: src: /home/ansible/exports.j2 dest: /etc/exports notify: update nfs handlers: - name: update nfs exports command: exportfs -a listen: update nfs


Add a play for the remote host group.

Edit /home/ansible/nfs.yml such that changes are executed on the remote host group to meet the provided requirements.

Edit /home/ansible/nfs.yml to contain the following content after the nfs play: - hosts: remote become: yes vars: nfs_ip: "{{ hostvars['nfs']['ansible_default_ipv4']['address'] }}" nfs_hostname: "{{ hostvars['nfs']['ansible_hostname'] }}" vars_files: - /home/ansible/user-list.txt tasks: - name: configure hostsfile template: src: /home/ansible/etc.hosts.j2 dest: /etc/hosts.nfslab - name: get file status stat: path: /opt/user-agreement.txt register: filestat - name: debug info debug: var: filestat - name: create users user: name: "{{ item }}" when: filestat.stat.exists loop: "{{ users }}"


Execute playbook to verify your playbook works correctly.

Execute playbook /home/ansible/nfs.yml to verify your playbook works correctly.

Run ansible-playbook /home/ansible/nfs.yml from the control node.
