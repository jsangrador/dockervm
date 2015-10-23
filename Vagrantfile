# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.define "dockervm"
  config.vm.provider :virtualbox do |vb|
    vb.name = "dockervm"
  end
  config.vm.box = "ubuntu/vivid64"
  config.vm.network "private_network", ip: "192.168.33.2"
  config.vm.synced_folder ".", "/vagrant", disabled:true
  config.vm.provision "shell", inline: <<-SHELL
    #sudo apt-get install -y docker.io
    # Use instructions at https://docs.docker.com/installation/ubuntulinux/
    # to install the Docker.com supported latest docker.
    sudo apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    echo "deb https://apt.dockerproject.org/repo ubuntu-vivid main" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get purge -y lxc-docker*
    sudo apt-get install -y docker-engine

    # Create docker certs.
    # as per https://docs.docker.com/articles/https/
    openssl genrsa -out ca-key.pem 4096 > /dev/null 2>&1
    openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -subj "/C=US/ST=Nowhere/L=Nowhere/O=NoOrg/OU=ITDept/CN=192.168.33.2" -out ca.pem
    openssl genrsa -out server-key.pem 4096 > /dev/null 2>&1
    openssl req -subj "/CN=192.168.33.2" -sha256 -new -key server-key.pem -out server.csr
    echo subjectAltName = IP:192.168.33.2,IP:127.0.0.1 > extfile.cnf
    openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
    openssl genrsa -out key.pem 4096 > /dev/null 2>&1
    openssl req -subj '/CN=client' -new -key key.pem -out client.csr
    echo extendedKeyUsage = clientAuth > extfile.cnf
    openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
    rm -v client.csr server.csr
    chmod -v 0400 ca-key.pem key.pem server-key.pem
    chmod -v 0444 ca.pem server-cert.pem cert.pem
    sudo mv ca-key.pem server-key.pem server-cert.pem /
    sudo cp ca.pem /
    sudo chown vagrant:vagrant key.pem cert.pem ca.pem # so we can scp them easily from the host

    # Configure docker daemon to listen on TCP, as per
    # https://docs.docker.com/articles/systemd/
    sudo mkdir /etc/systemd/system/docker.service.d
    echo "[Service]" | sudo tee /etc/systemd/system/docker.service.d/override-command.conf
    echo "ExecStart=" | sudo tee -a /etc/systemd/system/docker.service.d/override-command.conf
    echo "ExecStart=/usr/bin/docker daemon -H unix:///var/run/docker.sock -H tcp://192.168.33.2:2376 --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem" | sudo tee -a /etc/systemd/system/docker.service.d/override-command.conf

    sudo systemctl daemon-reload
    sudo service docker restart
  SHELL
end
