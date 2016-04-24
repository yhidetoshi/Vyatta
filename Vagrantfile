VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
 
  config.vm.define :vyatta_client1 do |vyatta_client1|
    vyatta_client1.vm.box = "higebu/vyos-1.1.7-amd64"
    vyatta_client1.vm.hostname = "vyatta-client1"
    vyatta_client1.vm.network "private_network", ip: "192.168.33.30"

    vyatta_client1.vm.provider "virtualbox" do |v1|
	v1.cpus = 1
	v1.memory = 512
    end
  end
 

  config.vm.define :vyatta_client2 do |vyatta_client2|
    vyatta_client2.vm.box = "higebu/vyos-1.1.7-amd64"
    vyatta_client2.vm.hostname = "vyatta-client2"
    vyatta_client2.vm.network "private_network", ip: "192.168.33.31"

    vyatta_client2.vm.provider "virtualbox" do |v2|
	v2.cpus = 1
	v2.memory = 512
    end
  end


  config.vm.define :vyatta_client3 do |vyatta_client3|
    vyatta_client3.vm.box = "higebu/vyos-1.1.7-amd64"
    vyatta_client3.vm.hostname = "vyatta-client3"
    vyatta_client3.vm.network "private_network", ip: "192.168.33.32"

    vyatta_client3.vm.provider "virtualbox" do |v3|
	v3.cpus = 1
	v3.memory = 512
    end
  end

end
