#Ansible Playbook test for mdbook container

This playbook simply checks if mdbook is up to date and the mdbook service is running

```
- name: mdbook check
  hosts: myhosts
  tasks:
   - name: Ensure mdbook latest
     ansible.builtin.apt:
       name: mdbook
       state: latest
   - name: Ensure mdbook is running
     ansible.builtin.service:
       name: mdbook
       state: started
```
