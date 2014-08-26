# -*- mode: ruby -*-
# vi: set ft=ruby :

PROJECT_NAME = "neos.demo.typo3.org"
SUBCONTEXT = PROJECT_NAME.gsub(".", "").capitalize + "vm"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 1.5.4"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "chef/debian-7.4"

  config.vm.hostname = PROJECT_NAME + ".vm"

  # Create a private network, which allows host-only access to the machine
  config.vm.network "private_network", ip: "10.1.0.2"

  # adjust VirtualBox specific settings
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    # this allows use of our own 10.0.2.x network
    vb.customize ["modifyvm", :id, "--natnet1", "10.0.100.0/24"]
  end

  # disable default shared folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # use your local SSH keys / agent even within the virtual machine
  config.ssh.forward_agent = true

  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box
  end

  # share site folder into releases folder
  config.vm.synced_folder ".", "/var/www/" + config.vm.hostname + "/releases/vagrant",
    type: "rsync",
    group: "web",
    rsync__args: ["--verbose", "--archive", "--delete", "-z", "--perms", "--chmod=Dg+s,Dg+rwx"],
    rsync__exclude: [
      ".git/",
      ".idea/",
      ".DS_Store",
      "Configuration/PackageStates.php",
      "Configuration/Production/" + SUBCONTEXT,
      "Configuration/Development/" + SUBCONTEXT,
      "Data/Sessions/**",
      "Data/Temporary/**",
      "Data/Persistent/**",
      "Data/Surf/**",
      "Data/Logs/**",
      "Web/_Resources/**"
    ]

  # Install chef client
  config.vm.provision "shell", inline: "dpkg -s chef > /dev/null 2>&1 || (wget -O - http://opscode.com/chef/install.sh | sudo /bin/sh)"

  # Provision with Chef Solo
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = [ "Build/Chef/cookbooks", "Build/Chef/site-cookbooks" ]
    chef.roles_path = "Build/Chef/roles"
    chef.data_bags_path = "Build/Chef/data_bags"

#    chef.log_level = :debug

    chef.add_role "base"
    chef.add_role "vagrant"
    chef.add_role "databaseserver"
    chef.add_role "webserver"

    chef.json = {
    }
  end
end
