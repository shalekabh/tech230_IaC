# Ansible app setup

We are using an ansible playbook (which is a configuration file) to install nginx and run the app:

Make sure to copy your app folder to the vm using:

```scp -r :/users/shale/tech230_virtualisation_tech230_Jenkins/app vagrant@192.168.33.12:~/```

On the controller terminal cd /etc/ansible:

Then create a playbook using ansible: ```sudo nano config_nginx_web.yaml```

in this file you want these commands:

```
# create a playbook to install nginx in web-server/s

# three dashes are used ot start the YAML file

---

# Add the name of the host
- hosts: web

# gather facts about the steps
  gather_facts: yes

# add admins access to the file
  become: true

# add instructions/tasks to install nginx
  tasks:
  - name: Installing Nginx
    apt: pkg=nginx state=present

# install nginx and enable nginx - ensure status is running
```

Next you want to copy your app from the controller to the web machine:

```
---
- name: Copy app folder to web VM
  hosts: web
  tasks:
    - name: Ensure destination directory exists
      file:
        path: /home/vagrant/app
        state: directory
      become: true

    - name: Copy app folder from controller to web
      copy:
        src: /home/vagrant/app/
        dest: /home/vagrant/app/
        owner: vagrant
        group: vagrant
        mode: '0755'
      become: true
```

To start the app and reverse proxy:

```
---
- name: Setup Node.js environment and start app on web VM
  hosts: web
  become: yes

  tasks:
    - name: Gathering Facts
      setup:

    - name: Update the system
      apt:
        update_cache: yes

    - name: Install curl
      apt:
        name: curl
        state: present

    - name: Add Node.js 14.x repository
      shell: curl -sL https://deb.nodesource.com/setup_14.x | bash -
      args:
        warn: false

    - name: Install Node.js
      apt:
        name: nodejs
        state: present


    - name: Set up Nginx reverse proxy
      replace:
        path: /etc/nginx/sites-available/default
        regexp: 'try_files \$uri \$uri/ =404;'
        replace: 'proxy_pass http://localhost:3000/;'

    - name: Test Nginx configuration
      command: nginx -t
      ignore_errors: yes
      register: nginx_config

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
      become: true
      when: nginx_config.rc == 0

    - name: Installing NPM
      apt:
        name: npm
        state: present


```

To run the play book the command is ```sudo ansible-playbook config_nginx_web.yaml``` then you should see this:

![Alt text](pics_for_mds/playbook%20test.png)

The full script is as follows; this includes the environment variable change:

```
# Create a playbook to install nginx in webserver/s

# Lets add 3 dashes --- to start the YAML file (this is how the interperator will pick it up)
---
# Add the name of the host
- hosts: web

# Gather facts about host (logs)
  gather_facts: yes

# Add admins access to this file
  become: true

# Add instructions / Tasks to install dependencies

- name: Install Nginx and Node.js
  hosts: web
  become: true

  tasks:
    - name: Installing Nginx
      apt:
        name: nginx
        state: present

    - name: Update the system
      apt:
        update_cache: yes

    - name: Install curl
      apt:
        name: curl
        state: present

    - name: Add Node.js 14.x repository
      shell: curl -sL https://deb.nodesource.com/setup_14.x | bash -
      args:
        warn: false

    - name: Installing Node.js
      apt:
        name: nodejs
        state: present
- name: Copy app from controller to web and install npm
  hosts: web
  become: true
  tasks:
    - name: Create destination directory
      file:
        path: /home/vagrant/app
        state: directory
        owner: vagrant

    - name: Copy app folder from controller to web
      copy:
        src: /home/vagrant/app
        dest: /home/vagrant/app
        owner: vagrant
        group: vagrant
        mode: '0755'
      become: true

    - name: Append DB_HOST to .bashrc
      lineinfile:
        dest: ~/.bashrc
        line: 'export DB_HOST=mongodb://18.203.95.206:27017/posts'
        insertafter: EOF
        create: yes
      become: true

    - name: Reverse Proxy
      replace:
        path: /etc/nginx/sites-available/default
        regexp: 'try_files \$uri \$uri/ =404;'
        replace: 'proxy_pass http://localhost:3000;'
      become: true

    - name: Test Nginx configuration
      command: nginx -t
      ignore_errors: yes
      register: nginx_config

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
      become: true
      when: nginx_config.rc == 0

    - name: Installing NPM
      apt:
        name: npm
        state: present
```