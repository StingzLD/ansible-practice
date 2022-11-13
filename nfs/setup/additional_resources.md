Additional Resources

The team has been working on automating the builds for common services using Ansible. You have been tasked with creating the build for a common NFS server. Write a playbook that satisfies the following requirements:

On nfs:

Make sure nfs-utils is installed.
Configure /etc/exports via an Ansible template stored in /home/ansible/exports.j2. Deploy the template so that /mnt/nfsroot is exported with read and write to all hosts.
Note: Your template file should contain the following with the variable {{ share_path }} being defined within your playbook:

{{ share_path }} *(rw)
Note: The file /etc/exports on nfs should have the following content once deployed:

/mnt/nfsroot *(rw)
Create a handler that runs the command exportfs -a if the file /etc/exports is modified in a playbook task.
You can assume all necessary firewall rules have been deployed.
On remote:

Configure /etc/hosts from a template file stored on control at /home/ansible/etc.hosts.j2 with the following entries:

  127.0.0.1 localhost {{ ansible_hostname }}
  {{ nfs_ip }}  {{ nfs_hostname }}
Note: You should populate the variables {{ nfs_ip }} and {{ nfs_hostname }} using magic variables in your playbook.
Create users from file stored on control at /home/ansible/user-list.txt only if the remote host has the file /opt/user-agreement.txt.

The Ansible control node has been configured for you and all other servers have already been configured for use with Ansible. The default inventory has been configured to include the group remote and server nfs.
