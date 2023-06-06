# Ansible
![Alt text](pics_for_mds/anisble%20diagram.png)


### Vagrant

Have a vagrant file configured, either import one or run ```vagrant init``` to make one for configuration.

We want 3 vms, controller(for ansible), web and db.

Once the vagrantfile is configured run ```vagrant up``` to launch the vms

Once they are running ssh into the controller from your local git bash terminal : ```vagrant ssh controller``` and ```sudo apt update -y``` and ```sudo apt upgrade -y```

For the next 2 steps it should ask you to authorise the fingerprint, type yes and enter the password.

Then ssh into the web vm through the controller terminal : ```ssh vagrant@192.168.33.10``` and ```sudo apt update -y``` and ```sudo apt upgrade -y```

```exit``` so you are back in controller 

Then ssh into db vm through the controller terminal vagrant@192.168.33.11``` and ```sudo apt update -y``` and ```sudo apt upgrade -y```

### Installing ansible

Next install common propeties ```sudo apt install software-properties-common```

Then add the repo and install ansible ```sudo apt-add-repository ppa:ansible/ansible``` abd ```sudo apt install ansible -y```

Once you see the config files downloaded ```cd etc```, ```cd ansible```, ```sudo apt install tree``` this one is to visualise things more clearly, . Then ```sudo nano hosts``` to configure the hosts file to allow our controller to connect to our web and db vms we add 2 lines of code:

```192.168.33.10```
```192.168.33.11```

```crtl + x``` 
```y```
```enter```

```sudo ansible web -m ping```
```sudo ansible db -m ping```

they will fail but it will add the fingerprint.

finally add this to the config file with the 2 ip addresse

```192.168.33.10 anisble_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant```
```192.168.33.11 anisble_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant```

save and exit

```sudo ansibe all -m ping``

will ping all vms and you should get response pong.












