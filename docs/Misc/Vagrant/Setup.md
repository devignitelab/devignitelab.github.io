---
title: Setup
---

## Installing Virtualization Driver (default Virtualbox)
- [Download Virtualbox](https://www.virtualbox.org/wiki/Linux_Downloads){: .btn .btn-outline-primary .btn-sm .py-1 }
- #### Download and Save key
```bash
curl -o vbox_key.asc https://www.virtualbox.org/download/oracle_vbox_2016.asc 	
```
- #### Add key to Apt Sources
```bash
sudo apt-key add oracle_vbox_2016.asc 	
```
- #### Add below line to `/etc/apt/sources.list` (below example is for Ubuntu 20 focal)
```bash
echo deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian focal contrib | sudo tee -a /etc/apt/sources.list
```
- #### Get Updates
```bash
sudo apt-get update
```


## Installing Vagrant
- ### Linux
  - #### [Download Vagrant bin package](https://www.vagrantup.com/downloads)
  - #### Extract on desktop or a folder location
	```bash
	unzip <archive name>
	```
  - #### Create a symbolic link to the executable
    - #### In command the vagrant is extracted on Desktop
	```bash
	sudo ln -s /home/nikx/Desktop/vagrant /usr/local/bin
	```  

### References
- #### [Vagrant Image List](https://app.vagrantup.com/boxes/search)
