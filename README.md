# ansible-role
ansible role to install and setup of a docker in dev server 
Task 3: This is to run the command locally to install the docker in dev,but  to create a pipeline we have to push the code to the Azure repo for that we need create a inventory file and group vars instaed of host.ini

In this we have to install the docker in dev server by running the ansible playbook.

	1. We have created a role. In this role under task folder we created a main.yml. In this file we wrote steps to install the docker.
	
	#installing the required docker packages
	- name: Install required packages
	  become: true
	  become_user: root
	  yum: pkg={{ item.pkg }} state=present
	  with_items:
	    - { pkg: 'docker-ce' }
	    - { pkg: 'docker-ce-cli' }
	    - { pkg: 'containerd.io' }
	    - { pkg: 'python3-pip' }
	  notify:
	    - reload systemctl
	    - restart docker
	
	- name: Check docker module for python
	  command: pip3 show docker
	  register: docker_module
	  changed_when: false
	  ignore_errors: true
	
	# Install the docker module for python for use with Ansible
	- name: Install docker module for python
	  become: true
	  become_user: root
	  command: pip3 install docker
	  when: "'Name: docker' not in docker_module.stdout"
	
	- name: Enable docker service and ensure it is not masked
	  systemd:
	    name: docker
	    enabled: true
	    masked: false
	
	# Start the Docker service if it's running
	- name: Make sure the docker service is started
	  systemd:
	    state: started
	    name: docker
	
	2. Under the handelers folder we created a main.yml file and declared the docker restart steps
	  
	# handlers file for install_docker
	- name: reload systemctl
	  command: systemctl daemon-reload
	
	- name: restart docker
	  systemd:
	    state: started
	    name: docker
	
 3. we have to create a host file outside of the role because to pass the user and password while running the playbook.
            
[DEV]
sycddigecom01 ansible_ssh_user=ironman45

Here we declared the dev variable and give the dev server name and ansible user.

	4. Now the final step is write the ansible-playbook to use the role and install the docker in dev server. We created a docker.yml playbook
	In that we declare hosts and role.
	  
	- name: Install docker on RHEL
	  hosts: "{{ var1 }}"
	  become: yes
	  roles:
	    - install_docker
	
	5. Now we have to run the ansible-playbook. The command to run the ansible playbook is
	  ansible-playbook -i host.ini docker.yml -e "var1=DEV" -k
	
	-I is to call the host and in the host we pass the ansible user and docker.yml is a playbook -e(to pass the extra variable)  and we give var1=DEV( we use var1 to call the dev) and _k is to pass the password).


Task 4:
In this task we have to push the code to the RBF ansible so for this  
	1. I have created a inventory file to declare the dev and Qa servers.
	   
	
	all:
	  children:
	    # All of backendconfig
	    backendconfig:
	      children:
	        # backendconfig Dev for internal
	        backendconfig_dev:
	          hosts:
	            sycddigecom01:
	        # backendconfig QA for internal
	        backendconfig_qa:
	          hosts:
	            sycqdigecom01: 
	        
	    # All Internal systems that share common variables
	    all_internal:
	      vars:
	      children:
	        all_internal_np:
	          vars:
	          hosts:
	            sycddigecom01:
	            sycqdigecom01:


	2. In the group vars I have created the 2 yml files backendconfig_dev and backendconfig_qa and declared env: "dev" and env: "qa"
	
	
	3. In the playbook I declared a variable host to call the hosts from the inventory.
	
	4. While creating the pipeline these are the extra varibale I pass
	
	--limit backendconfig_dev -e "variable_host=backendconfig_dev" --vault-password-file "~/.vault-env"

Playbook: 

Variable host:  we use this in the pipeline as additional parameter to call the inventory
variable_host=backendconfig_dev" - it will take dev from inventory

--limit backendconfig_dev - it will limit to dev.


default('backendconfig')
