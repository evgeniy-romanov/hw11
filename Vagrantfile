# -*- mode: ruby -*-
# vim: set ft=ruby :

hosts = {
    :"nginx1" => {
        :box_name => "centos/7",
        :cpu => '1',
        :ram => "512",
        :ip_addr => '192.168.56.21',
        :port => '8081'
    },
    :"nginx2" => {
        :box_name => "centos/7",
        :cpu => '1',
        :ram => "512",
        :ip_addr => '192.168.56.22',
        :port => '8082'
    },
}
Vagrant.configure("2") do |config|

    hosts.each do |boxname, boxconfig|

        config.vm.define boxname do |box|
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
            box.vm.network "forwarded_port", guest: 8080, host: boxconfig[:port]

            box.vm.provider :virtualbox do |vb|
                vb.cpus = boxconfig[:cpu]
                vb.memory = boxconfig[:ram]
                vb.name = "hw11" + "%s" % boxname
            end
        end
    end
end
