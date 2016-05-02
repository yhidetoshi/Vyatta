VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
 
  config.vm.define :vyatta_client1 do |vyatta_client1|
    vyatta_client1.vm.box = "Base-OS-Cent"
    vyatta_client1.vm.hostname = "vm1"
    vyatta_client1.vm.network "private_network", ip: "192.168.1.2"
    #vyatta_client1.vm.network "private_network", ip: "10.0.2.20"

#    vyatta_client1.vm.provider "virtualbox" do |v1|
#	v1.cpus = 1
#	v1.memory = 512
#    end
  end

  config.vm.define :vyatta_client2 do |vyatta_client2|
    vyatta_client2.vm.box = "higebu/vyos-1.1.7-amd64"
    vyatta_client2.vm.hostname = "vyatta-client2"
    vyatta_client2.vm.network "private_network", ip: "192.168.1.11"
    vyatta_client2.vm.network "private_network", ip: "10.0.1.11"
    vyatta_client2.vm.network "private_network", ip: "10.0.2.11"


#    vyatta_client2.vm.provider "virtualbox" do |v2|
#	v2.cpus = 1
#	v2.memory = 512
#    end
  end

  config.vm.define :vyatta_client3 do |vyatta_client3|
    vyatta_client3.vm.box = "higebu/vyos-1.1.7-amd64"
    vyatta_client3.vm.hostname = "vyatta-client3"
    vyatta_client3.vm.network "private_network", ip: "10.0.1.9"
    vyatta_client3.vm.network "private_network", ip: "10.0.2.9"

#    vyatta_client3.vm.provider "virtualbox" do |v3|
#	v3.cpus = 1
#	v3.memory = 512
#    end
  end

  config.vm.define :vyatta_client4 do |vyatta_client4|
    vyatta_client4.vm.box = "Base-OS-Cent"
    vyatta_client4.vm.hostname = "vm4" 
    #vyatta_client4.vm.network "private_network", ip: "10.0.0.8"
    vyatta_client4.vm.network "private_network", ip: "10.0.1.8"


    #vyatta_client4.provider "virtualbox" do |v4|
    #	v4.cpus = 1
    #	v4.memory = 512
    #end
  end

#  config.vm.define :vyatta_client5 do |vyatta_client5|
#    vyatta_client5.vm.box = "higebu/vyos-1.1.7-amd64"
#    vyatta_client5.vm.hostname = "vyatta-client5"
#    vyatta_client5.vm.network "private_network", ip: "192.168.2.2"

    #vyatta_client5.provider "virtualbox" do |v5|
    #	v5.cpus = 1
    #	v5.memoory = 512
    #end
 # end

end
