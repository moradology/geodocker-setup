# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6"

current_dir    = File.dirname(File.expand_path(__FILE__))
cluster_conf        = YAML.load_file("#{current_dir}/conf.yml")

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provision "docker",
    images: cluster_conf["images"]

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = false
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
  else
    $stderr.puts "\nERROR: Please install the vagrant-hostmanager plugin."
    exit(1)
  end

  config.vm.define "master", primary: true do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.11.100"

    # Disable because this will not get used.
    master.vm.synced_folder ".", "/vagrant", disabled: true

    master.vm.synced_folder "./master", "/opt/app", type: "nfs"

    master.vm.network "forwarded_port", guest: 12234, host: 10000
    master.vm.network "forwarded_port", guest: 2181, host: 20000
    master.vm.network "forwarded_port", guest: 3000, host: 30000
    master.vm.network "forwarded_port", guest: 50095, host: 40000
    master.vm.network "forwarded_port", guest: 9997, host: 50000
    master.vm.network "forwarded_port", guest: 9999, host: 60000

    master.vm.provision "docker" do |d|
      d.run "zookeeper",
        image: "daunnc/geodocker-zookeeper:latest",
        args: '--net=host '\
              '--volume=/data/gt/zookeeper:/data/zookeeper '\
              '--env="ZOOKEEPER_ID=1" '\
              '--env="ZOOKEEPER_SERVER_1=master" '\
              '--env="ZOOKEEPER_SERVER_2=slave-1" '\
              '--env="ZOOKEEPER_SERVER_3=slave-2" '\
              '--detach -p 2181:2181 '\
              '--restart=always '

      d.run "hdfs-name",
        image: "daunnc/geodocker-hadoop-name:latest",
        args: '--net=host '\
              '--volume=/data/gt/hdfs:/data/hdfs '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--detach '\
              '--restart=always '

      d.run "hdfs-sname",
        image: "daunnc/geodocker-hadoop-sname:latest",
        args: '--net=host '\
              '--volume=/data/gt/hdfs:/data/hdfs '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--detach '\
              '--restart=always '

      d.run "hdfs-data",
        image: "daunnc/geodocker-hadoop-data:latest",
        args: '--net=host '\
              '--volume=/data/gt/hdfs:/data/hdfs '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--detach '\
              '--restart=always '

      d.run "accumulo-init",
        image: "daunnc/geodocker-accumulo-master:latest",
        args: '--net=host '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" ',
        cmd:  'bash -c "hadoop fs -mkdir -p /accumulo-classpath && accumulo init --instance-name gis --password GisPwd"'

      d.run "accumulo-master",
        image: "daunnc/geodocker-accumulo-master:latest",
        args: '--net=host '\
              '--detach '\
              '--restart=always '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" '

      d.run "accumulo-tracer",
        image: "daunnc/geodocker-accumulo-tracer:latest",
        args: '--net=host '\
              '--detach '\
              '--restart=always '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" '

      d.run "accumulo-gc",
        image: "daunnc/geodocker-accumulo-gc:latest",
        args: '--net=host '\
              '--detach '\
              '--restart=always '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" '

      d.run "accumulo-monitor",
        image: "daunnc/geodocker-accumulo-monitor:latest",
        args: '--net=host '\
              '--detach '\
              '--restart=always '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" '

      d.run "accumulo-tserver",
        image: "daunnc/geodocker-accumulo-tserver:latest",
        args: '--net=host '\
              '--detach '\
              '--restart=always '\
              '--env="HADOOP_MASTER_ADDRESS=master" '\
              '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
              '--env="ACCUMULO_SECRET=secret" '\
              '--env="ACCUMULO_PASSWORD=GisPwd" '

    end

    master.ssh.forward_x11 = true

    master.vm.provider :virtualbox do |v|
      v.memory = 4096
      v.cpus = 2
    end
  end

  (1..cluster_conf["slaves"]).each do |i|
    config.vm.define "slave-#{i}" do |slave|
      slave.vm.hostname = "slave-#{i}"
      slave.vm.network "private_network", ip: "192.168.11.10#{i}"

      # Disable because this will not get used.
      slave.vm.synced_folder ".", "/vagrant", disabled: true

      slave.vm.synced_folder "./slave", "/opt/app", type: "nfs"

      slave.vm.network "forwarded_port", guest: 1000, host: 10000 + i
      slave.vm.network "forwarded_port", guest: 2000, host: 20000 + i
      slave.vm.network "forwarded_port", guest: 3000, host: 30000 + i
      slave.vm.network "forwarded_port", guest: 4000, host: 40000 + i
      slave.vm.network "forwarded_port", guest: 5000, host: 50000 + i
      slave.vm.network "forwarded_port", guest: 6000, host: 60000 + i


      slave.vm.provision "docker" do |d|
        d.run "zookeeper",
          image: "daunnc/geodocker-zookeeper:latest",
          args: '--net=host '\
                '--volume=/data/gt/zookeeper:/data/zookeeper '\
                "--env=\"ZOOKEEPER_ID=#{i}\" "\
                '--env="ZOOKEEPER_SERVER_1=master" '\
                '--env="ZOOKEEPER_SERVER_2=slave-1" '\
                '--env="ZOOKEEPER_SERVER_3=slave-2" '\
                '--detach '\
                '--restart=always '

        d.run "hdfs-data",
          image: "daunnc/geodocker-hadoop-data:latest",
          args: '--net=host '\
                '--volume=/data/gt/hdfs:/data/hdfs '\
                '--env="HADOOP_MASTER_ADDRESS=master" '\
                '--detach '\
                '--restart=always '

        d.run "accumulo-tserver",
          image: "daunnc/geodocker-accumulo-tserver:latest",
          args: '--net=host '\
                '--detach '\
                '--restart=always '\
                '--env="HADOOP_MASTER_ADDRESS=master" '\
                '--env="ACCUMULO_ZOOKEEPERS=master,slave-1,slave-2" '\
                '--env="ACCUMULO_SECRET=secret" '\
                '--env="ACCUMULO_PASSWORD=GisPwd" '


      end

      slave.ssh.forward_x11 = true

      slave.vm.provider :virtualbox do |v|
        v.memory = 2048
        v.cpus = 1
      end
    end
  end
end
