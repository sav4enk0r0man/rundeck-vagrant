system("
    if [ #{ARGV[0]} = 'up' ]; then
        echo 'Generate ssh keys for vms'
        yes y | ssh-keygen -b 2048 -t rsa -f vms-keys -P \"\"
    fi
")

Vagrant.configure("2") do |config|

  PROJECT="anvils"
  RDIP="192.168.50.2"
  IP_DB1="192.168.50.3"
  IP_DB2="192.168.50.4"
  RUNDECK_YUM_REPO="https://bintray.com/rundeck/rundeck-rpm/rpm"

  # Configure host
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.ssh.insert_key = false
  config.vm.box_version = "201708.22.0"
  config.vm.box = "bento/centos-7.3"
  config.ssh.insert_key = true

  config.vm.provider "virtualbox" do |vb|
   vb.cpus = "2"
   vb.memory = "2048"
  end

  ssh_private_key = File.open("vms-keys", "rb").read.strip
  ssh_public_key = File.open("vms-keys.pub", "rb").read.strip

  # DB1 host
  config.vm.define :db1 do |db1|
    db1.vm.hostname = "db1.anvils.com"
    db1.vm.network :private_network, ip: "#{IP_DB1}"

    db1.vm.provision :shell, inline: <<-SHELL
      echo "#{ssh_private_key}" > /home/vagrant/.ssh/id_rsa
      chown vagrant /home/vagrant/.ssh/id_rsa
      chmod 400 /home/vagrant/.ssh/id_rsa
      echo "#{ssh_public_key}" >> /home/vagrant/.ssh/authorized_keys
    SHELL
    db1.vm.provision :shell, inline: "yum install epel-release -y"
  end

  # DB2 host
  config.vm.define :db2 do |db2|
    db2.vm.hostname = "db2.anvils.com"
    db2.vm.network :private_network, ip: "#{IP_DB2}"

    db2.vm.provision :shell, inline: <<-SHELL
      echo "#{ssh_private_key}" > /home/vagrant/.ssh/id_rsa
      chown vagrant /home/vagrant/.ssh/id_rsa
      chmod 400 /home/vagrant/.ssh/id_rsa
      echo "#{ssh_public_key}" >> /home/vagrant/.ssh/authorized_keys
    SHELL
    db2.vm.provision :shell, inline: "yum install epel-release -y"
  end

  # Rundeck host
  config.vm.define :rundeck do |rundeck|
    rundeck.vm.hostname = "rundeck.anvils.com"
    rundeck.vm.network :private_network, ip: "#{RDIP}"

    rundeck.vm.provision :shell, inline: <<-SHELL
      echo "#{ssh_private_key}" > /home/vagrant/.ssh/id_rsa
      chown vagrant /home/vagrant/.ssh/id_rsa
      chmod 400 /home/vagrant/.ssh/id_rsa
      echo "#{ssh_public_key}" >> /home/vagrant/.ssh/authorized_keys
    SHELL
    rundeck.vm.provision :shell, inline: "yum install epel-release -y"

    rundeck.vm.provision :shell, :path => "install-rundeck.sh", :args => "#{RDIP} #{RUNDECK_YUM_REPO}"
    rundeck.vm.provision :shell, :path => "install-httpd.sh"
    rundeck.vm.provision :shell, :path => "add-project.sh", :args => "#{PROJECT}"
  end
end
