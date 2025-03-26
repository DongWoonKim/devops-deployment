VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config| 
  config.vm.define :gitlab do |cfg|
    cfg.vm.box = "bento/ubuntu-22.04"
    cfg.vm.hostname = "gitlab.local"

    if Vagrant.has_plugin?("vagrant-vbguest")
      cfg.vbguest.auto_update = false
    end

    cfg.vm.network "forwarded_port", guest: 80, host: 8080
    cfg.vm.network "forwarded_port", guest: 22, host: 8022
    cfg.vm.network "private_network", ip: "192.168.33.44"
    cfg.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/"]

    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "gitlab.local"
      vb.memory = "4096"
    end

    cfg.vm.provision "shell", inline: <<-'SHELL'
      sudo apt-get update 
      sudo apt-get install -y curl openssh-server ca-certificates libatomic1

      debconf-set-selections <<< "postfix postfix/mailname string $HOSTNAME"
      debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
      DEBIAN_FRONTEND=noninteractive sudo apt-get install -y postfix
 
      if [ ! -e /vagrant/ubuntu-jammy-gitlab-ce_15.11.2-ce.0_arm64.deb ]; then
          wget --content-disposition -O /vagrant/ubuntu-jammy-gitlab-ce_15.11.2-ce.0_arm64.deb https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/jammy/gitlab-ce_15.11.2-ce.0_arm64.deb/download.deb
      fi
 
      sudo dpkg -i /vagrant/ubuntu-jammy-gitlab-ce_15.11.2-ce.0_arm64.deb || sudo apt-get -f install -y 
      sudo gitlab-ctl reconfigure || echo "gitlab-ctl command not found. 설치가 정상적으로 완료되었는지 확인하세요."
    SHELL
  end

  # Runner VM 정의
  config.vm.define :runner do |cfg|
    cfg.vm.box = "bento/ubuntu-22.04"
    cfg.vm.hostname = "runner.local"
 
    cfg.vm.network "private_network", ip: "192.168.33.45"

    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "runner.local"
      vb.memory = "2048"
    end

    cfg.vm.provision "shell", inline: <<-'SHELL'
      sudo apt-get update
      sudo apt-get install -y curl 
    SHELL
  end
end