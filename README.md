**K3S on Vagrant**

Installs:
  - Ubuntu Jammy 22.04
  - K3S
  - Kubernetes API 1.28.9

Installation consists of 3 Leaders and 4 Worker nodes.

Install Steps
1. Edit/remove the netmask setting I have in the Vagrantfile.  I use a /23, please edit that for your network.
2. Edit the node IP addresses to what is needed for your network as well
3. Install virtualbox 7.x
4. Install vagrant 2.4.x
5. run ```vagrant up``` in the directory containing the Vagrantfile
