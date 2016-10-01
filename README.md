# Using Vagrant and Docker for Hydra Development

## What is Vagrant

For the scope of this presentation, vagrant is a wrapper around virtual machines.  Vagrant makes managing a VM easier.  When someone talks about something running "on vagrant" or "using vagrant", they mean it's being run on a virtual machine.
For the official about, https://www.vagrantup.com/about.html.  
## What is Docker
Docker can be thought of as some machinery that has just enough to run an application in an isolated space.  
The isolated spaces are called containers, and the main difference is they don't have their own OS.

The official docker what is, https://www.docker.com/what-docker.

## Using Vagrant with a VM.
### Installation
Install VirtualBox.
  This is the underlying VM.  You could use it directly with Oracle's GUI and command line tools if you wanted.
Install Vagrant.
  Because I would just end up writing a script to set up the VM for me.

### Make a Debian Box
Common use cases:
- In order to have a fresh machine to try things on.
- As a stand in for a remote machine.
- To have a place to write code with impunity.

#### Fresh Machine to Test Something (Apache)
1. create a Vagrantfile.
    ```bash
    cp examples/fresh-vagrantfile ./Vagrantfile
    ```

This is the configuration instructions for vagrant.
    ```ruby
    Vagrant.configure(2) do |config|
      # https://docs.vagrantup.com.

      # The virtual machine specification
      config.vm.box = "box-cutter/debian85"
      config.vm.define "fresh"

      config.vm.boot_timeout = 60
      config.vm.hostname = "fresh"

      # Create a forwarded port mapping which allows access to a specific port
      # accessing "localhost:8888" will access port 80 on the guest machine.
      config.vm.network "forwarded_port", guest: 80,   host: 8888, auto_correct: true

      config.vm.provider "virtualbox" do |vb|
        vb.name = "fresh"
        vb.memory = "1024"
      end
    end
    ```
1. start the virtual machine
    ```bash
    vagrant up
    ```
    It will take a little while to download the image, copy the basebox, and boot.
1. Connect to the virtual machine via ssh.
  Vagrant provides a convenient way to do this.  If you want to do it more manually, see the docs for the vagrant-ssh command.
    ```bash
    vagrant ssh
    ```
1. Now you're in a fresh Debian install. Do whatever it is you're going to do and exit the ssh session.
    ```bash
    echo "foo" > bar.txt
    # Update the apt package listing
    sudo apt-get update
    # Install apache2. To allow us to try some things out.
    # Also install tree for looking at directory structures.
    sudo apt-get install -y apache2 tree

    # Copy files from directory shared between host and guest
    sudo mkdir -p /var/www/localhost
    sudo cp /vagrant/examples/hello-index.html /var/www/localhost/index.html
    sudo cp /vagrant/examples/hello-site.conf /etc/apache2/sites-available/site.conf

    # Enable apache site
    sudo a2ensite site.conf
    sudo service apache2 reload

    # Exit the VM
    exit
    ```
1. The VM is still running.  Vagrant took care of forwarding localhost:8888 on our machine to localhost:80 on the VM.
Check that apache is running and serving out our index html by pointing a browser to localhost:8888 or using curl.
    ```bash
    # Should return the content of hello-index.html
    curl localhost:8888
    ```

#### As a Stand In for a Remote Machine
Testing capistrano deployment processes is an example of a workflow that requires a remote machine.  The alternative is installing and configuring sshd on your local machine or using an actual remote machine.  

1. copy the next examples as our Vagrantfile.
    ```bash
    cp examples/remote-vagrantfile ./Vagrantfile
    ```
