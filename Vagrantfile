# -*- mode: ruby -*-
# # vi: set ft=ruby :

DEPS = [
    'build-essential',
    'curl',
    'rabbitmq-server',
    'vim',
].join(' ')


Vagrant::Config.run do |config|
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.network :hostonly, "33.33.33.18"
  config.vm.forward_port 15672, 15672
  config.vm.forward_port 15673, 15673
  config.vm.forward_port 5672, 5672
  config.vm.forward_port 5673, 5673
  config.ssh.forward_agent = true

  config.vm.provision :shell do |shell|
    shell.inline = <<-eos
        #!/bin/sh

        # throw this out if you're not in iceland
        if !(grep ftp.rhnet.is /etc/apt/sources.list) then
            sed -i "s/us.archive.ubuntu.com/ftp.rhnet.is/g" /etc/apt/sources.list
        fi

        apt-get update -y -q
        apt-get install python-software-properties -y -q
        
        # rabbitmq 
        if !(grep www.rabbitmq.com /etc/apt/sources.list) then
            echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
            wget -qO - http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | apt-key add -
        fi

        apt-get update -y -q
        apt-get install #{DEPS} -y -q

        rabbitmq-plugins enable rabbitmq_management
        rabbitmq-plugins enable rabbitmq_management_visualiser
        
        service rabbitmq-server stop

        RABBITMQ_NODE_PORT=5672 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" RABBITMQ_NODENAME=r1 rabbitmq-server -detached 
        RABBITMQ_NODE_PORT=5673 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=r2 rabbitmq-server -detached
        sleep 5
        rabbitmqctl -n r2 stop_app 
        rabbitmqctl -n r2 join_cluster r1@`hostname -s`
        rabbitmqctl -n r2 start_app
        rabbitmqctl -n r1 set_policy ha-all "^ha\." '{"ha-mode":"all"}'

    eos
  end

end
