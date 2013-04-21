Vagrant.require_plugin("berkshelf-vagrant")

Vagrant.configure("2") do |config|
  config.vm.hostname   = "billingstack"
  # It seems the Ubuntu provided boxes don't work -_-
  config.vm.box        = "ubuntu-1204-chef"
  config.vm.box_url    = "https://s3.amazonaws.com/gsc-vagrant-boxes/ubuntu-12.04-omnibus-chef.box"
  config.vm.network    :private_network, ip: "192.168.50.1"
  config.vm.network    :forwarded_port, guest: 80, host: 9080
  config.vm.network    :forwarded_port, guest: 9091, host: 9091
  config.ssh.max_tries = 40
  config.ssh.timeout   = 120
  config.berkshelf.enabled = true

  config.vm.provider :virtualbox do |vbox|
    # Enable the VBox GUI
    vbox.gui = true if ENV['VAGRANT_GUI']
  end

  # Install the correct version of chef
  config.vm.provision :shell do |shell|
    shell.inline = %Q{
      if [ ! -f "/usr/bin/chef-solo" ]; then
        sudo echo "#!/bin/bash" > /usr/sbin/policy-rc.d &&
        sudo echo "exit 101" >> /usr/sbin/policy-rc.d &&
        sudo chmod +x /usr/sbin/policy-rc.d &&
        echo "deb http://apt.opscode.com/ `lsb_release -cs`-0.10 main" | sudo tee /etc/apt/sources.list.d/opscode.list &&
        wget -O - http://apt.opscode.com/packages@opscode.com.gpg.key | sudo apt-key add - &&
        sudo apt-get update &&
        sudo apt-get install chef --yes
      fi
      sudo rm /usr/sbin/policy-rc.d || true
    }
  end

  # Provision the VM with chef-solo
  config.vm.provision :chef_solo do |chef|
    chef.log_level       = :debug

    # Set some chef paths
    #chef.data_bags_path  = "data_bags"
    #chef.roles_path      = "roles"
    chef.cookbooks_path   = "cookbooks"

    # Set the chef run-list
    chef.add_recipe("apt")
    chef.add_recipe("rabbitmq")
    chef.add_recipe("mysql::server")
    chef.add_recipe("billingstack::repository")
    chef.add_recipe("billingstack::central")
    chef.add_recipe("billingstack::api")
    chef.add_recipe("billingstack::ui")
    
    # Provide some chef attributes
    chef.json = {
      :billingstack => {
        :DEFAULT => {
          :rabbit_hosts => ['127.0.0.1:5672']
        }
      },
      :rabbitmq => {
        :use_distro_version => true
      },
      :mysql => {
        :server_root_password => "server_root_password",
        :server_repl_password => "server_repl_password",
        :server_debian_password => "server_debian_password"
      }
    }
  end
end
