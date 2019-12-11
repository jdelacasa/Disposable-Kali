
#  8888888b.  d8b                                              888      888               888    d8P           888 d8b 
#  888  "Y88b Y8P                                              888      888               888   d8P            888 Y8P 
#  888    888                                                  888      888               888  d8P             888     
#  888    888 888 .d8888b  88888b.   .d88b.  .d8888b   8888b.  88888b.  888  .d88b.       888d88K      8888b.  888 888 
#  888    888 888 88K      888 "88b d88""88b 88K          "88b 888 "88b 888 d8P  Y8b      8888888b        "88b 888 888 
#  888    888 888 "Y8888b. 888  888 888  888 "Y8888b. .d888888 888  888 888 88888888      888  Y88b   .d888888 888 888 
#  888  .d88P 888      X88 888 d88P Y88..88P      X88 888  888 888 d88P 888 Y8b.          888   Y88b  888  888 888 888 
#  8888888P"  888  88888P' 88888P"   "Y88P"   88888P' "Y888888 88888P"  888  "Y8888       888    Y88b "Y888888 888 888 
#                          888                                                                                         
#                          888                                                                                         
#                          888                                                                                                                           
#
# Repo: https://github.com/gulfsteve/Disposable-Kali
# Version: 0.1.0

# -*- mode: ruby -*-
# vi: set ft=ruby :

# capture the vagrant command passed in as first argument
VAGRANT_COMMAND = ARGV[0]

# Configure with API version 2
Vagrant.configure("2") do |config|

  config.vm.define "kali", primary: true do |kali|

      #use the offsec base box
    #kali.vm.box = "offensive-security/kali-linux"
    kali.vm.box = "kalilinux/rolling"

    # [OPTIONAL] set to false to skip
    # gets the latest version of the base box 
    kali.vm.box_check_update = true

    # set the hostname of the box
    #kali.vm.hostname = "kali"

    # [OPTIONAL] comment out to skip
    # create a shared folder between host and vm 
    kali.vm.synced_folder "~/shared", "/root/shared", create: true, owner: "root", group: "root", automount: true
    
    # enable X11 tunnelling via SSH
    kali.ssh.forward_agent = true
    kali.ssh.forward_x11 = true

    # this allows "vagrant up" to work normally using the vagrant user
    # but if "vagrant ssh", then the root user will be used

    if VAGRANT_COMMAND == "ssh"
      kali.ssh.username = 'root'
    end

    # configure network

    config.vm.network "private_network", ip: "192.168.33.10"
    # configure virtualbox specific settings
    kali.vm.provider :virtualbox do |vbox|

      # run headless (can attach with vbox gui if needed)
      vbox.gui = false

      # set the name in virtualbox
      vbox.name = "kali"

      # configure memory settings according to your needs
      
      vbox.memory = "2048"
      #vbox.memory = "4096"
      #vbox.memory = "8192"
      
      # configure cpu settings according to your needs

      #vbox.cpus = "1"
      #vbox.cpus = "2"
      vbox.cpus = "4"

      # [OPTIONAL] set hidpi
      vbox.customize ['setextradata', :id, 'GUI/HiDPI/UnscaledOutput', '1']

      vbox.customize ["modifyvm", :id, "--accelerate3d", "off"]

      
    end

    # Execute Provisioning Scripts
    
    kali.vm.provision "shell", inline: $script_sshd_allow_root_login, privileged: true
    kali.vm.provision "shell", inline: $script_root_shell_aliases, privileged: true
    kali.vm.provision "shell", inline: $script_time_zone, privileged: true
    kali.vm.provision "shell", inline: $script_swap_add, privileged: true, args: 6
    kali.vm.provision "shell", inline: $script_network_resolvers, privileged: true
    kali.vm.provision "shell", inline: $script_network_ufw, privileged: true
    kali.vm.provision "shell", inline: $script_packages_extra, privileged: true
    kali.vm.provision "shell", inline: $script_tools_prep, privileged: true
    
    kali.vm.provision "shell", inline: $script_dir_prep, privileged: true
    kali.vm.provision "shell", inline: $script_00_doc, privileged: true
    kali.vm.provision "shell", inline: $script_01_information_gathering, privileged: true
    kali.vm.provision "shell", inline: $script_19_auto, privileged: true
    kali.vm.provision "shell", inline: $script_20_social_media, privileged: true
    kali.vm.provision "shell", inline: $script_21_anon, privileged: true
    # Uncomment this install all updates / upgrades - can be a long process...
    #kali.vm.provision "shell", inline: $script_packages_update, privileged: true

    kali.vm.post_up_message = $msg
    
  end

end

# ######################################## SCRIPTS

$script_sshd_allow_root_login = <<-SCRIPT
  echo "--- sshd_allow_root_login running... "

  echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
  systemctl restart sshd

  echo "--- sshd_allow_root_login completed"
SCRIPT

$script_root_shell_aliases = <<-SCRIPT
  echo "--- root_shell_aliases running... "

  if [[ -s /root/.bash_aliases ]]; then
    echo ".bash_aliases already exists..."
  else
    echo "alias ll='ls -lvhAF --file-type --group-directories-first'" >> /root/.bash_aliases
    echo "alias lr='ll -R'" >> /root/.bash_aliases
    source /root/.bash_aliases
  fi

  cat /root/.bash_aliases

  echo "--- root_shell_aliases completed. "
SCRIPT

$script_time_zone = <<-SCRIPT
  echo "--- time_zone running... "

  cp /usr/share/zoneinfo/America/Chicago /etc/localtime
  date

  echo "--- time_zone completed. "
SCRIPT

$script_swap_add = <<-SCRIPT
  echo "--- swap_add running... "

  if [[ -s /swapfile ]]; then
    echo "swapfile already exists..."
  else
    size=$1
    fallocate -l ${size}G /swapfile
    chown root:root /swapfile
    chmod 0600 /swapfile
    mkswap /swapfile
    swapon /swapfile
    cp /etc/fstab /etc/fstab.bak
    echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
  fi

  swapon

  echo "--- swap_add completed."
SCRIPT

# TODO:  add a check here for existing entry
$script_network_resolvers = <<-SCRIPT
  echo "--- network_resolvers running... "
  
  if [[ -s /tmp/network_resolvers ]]; then
    echo "resolvers alreadt added..."
  else
    echo "supersede domain-name-servers 1.1.1.1, 1.0.0.1;" >> /etc/dhcp/dhclient.conf
  fi
  
  cat /etc/dhcp/dhclient.conf | grep supersede

  echo "--- network_resolvers completed. "
SCRIPT

#customize to ports your liking... 
$script_network_ufw = <<-SCRIPT
  echo "--- network_ufw running... "
  apt-get install ufw -y
  apt-get install gufw -y
  ufw allow 22/tcp
  ufw allow 80/tcp
  ufw allow 9021/tcp    
  ufw allow 9022/tcp
  echo yes | ufw enable
  ufw status verbose
  echo "--- network_ufw completed. "
SCRIPT

$script_packages_extra = <<-SCRIPT
  echo "--- packages_extra running... "
  apt-get install seclists -y 
  apt-get install ftp -y  
  apt-get install php-curl -y 
  apt-get install python-smb -y
  apt-get install mingw-w64 -y
  dpkg --add-architecture i386
  apt-get update
  apt-get install wine32 -y
  apt-get install tor nyx -y
  apt-get install bpython -y
  apt-get install tmux htop -y 
  apt-get install gobuster -y
  echo "--- packages_extra completed... "
  echo "--- install nodejs running... "
  #mkdir  ~/local
  #mkdir  ~/tmp
  #echo 'export PATH=$HOME/local/bin:$PATH' >> ~/.bashrc
  #cd ~/tmp
  #git clone git://github.com/nodejs/node.git
  #cd node
  #./configure –-prefix=~/local
  #make install
  #cd ..
  #git clone git://github.com/npm/npm.git
  #cd npm
  #make install
  #cd ../..

  #apt-get install software-properties-common -y
  curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
  apt-get install nodejs npm -y


SCRIPT

$script_tools_prep = <<-SCRIPT
  echo "--- tools_prep running... "
  systemctl start postgresql
  systemctl enable postgresql
  msfdb init 
  searchsploit -u
  gunzip /usr/share/wordlists/rockyou.txt.gz
  systemctl start tor
  echo "--- tools_prep completed. "
SCRIPT

$script_packages_update = <<-SCRIPT
  echo "--- packages_update is running... "
  export DEBIAN_FRONTEND=noninteractive
  apt-get update 
  apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" dist-upgrade
  echo "--- packages_update completed."
SCRIPT

$script_dir_prep = <<-SCRIPT

  mkdir -p /root/workspace > /dev/null 2>&1
  mkdir -p /root/workspace/01-information-Gathering > /dev/null 2>&1
  mkdir -p /root/workspace/01-information-Gathering/WEB > /dev/null 2>&1
  mkdir -p /root/workspace/02-vulnerability_analisys > /dev/null 2>&1
  mkdir -p /root/workspace/03-web-application_analisys > /dev/null 2>&1
  mkdir -p /root/workspace/04-Database_assessment > /dev/null 2>&1
  mkdir -p /root/workspace/05-Password_attacks > /dev/null 2>&1
  mkdir -p /root/workspace/00-doc > /dev/null 2>&1
  mkdir -p /root/workspace/19-auto > /dev/null 2>&1
  mkdir -p /root/workspace/20-social-media > /dev/null 2>&1
  mkdir -p /root/workspace/21-anon > /dev/null 2>&1

SCRIPT

$script_00_doc = <<-SCRIPT

   cd /root/workspace/00-doc
  git clone https://github.com/mantvydasb/Offensive-Security-OSCP-Cheatsheets.git
  cd Offensive-Security-OSCP-Cheatsheets ; git pull
 
  cd /root/workspace/00-doc
  git clone https://github.com/CyDefUnicorn/OSCP-Archives.git
  cd OSCP-Archives ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/0x4D31/awesome-oscp.git
  cd awesome-oscp ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/OlivierLaflamme/Cheatsheet-God.git
  cd Cheatsheet-God ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/adon90/pentest_compilation.git
  cd pentest_compilation ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/swisskyrepo/PayloadsAllTheThings.git
  cd PayloadsAllTheThings ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/trimstray/the-book-of-secret-knowledge.git
  cd the-book-of-secret-knowledge ; git pull

  cd /root/workspace/00-doc
  git clone https://github.com/Ignitetechnologies/Privilege-Escalation.git
  cd Privilege-Escalation ; git pull
  
SCRIPT

$script_01_information_gathering = <<-SCRIPT

  cd /root/workspace/01-information-Gathering 
  git clone https://github.com/diego-treitos/linux-smart-enumeration.git
  cd linux-smart-enumeration ; git pull

  echo "Install Sherlock - Find Usernames Across Social Networks"
  cd /root/workspace/01-information-Gathering
  git clone https://github.com/sherlock-project/sherlock.git
  cd sherlock ; git pull
  pip3 install -r requirements.txt
 
  cd /root/workspace/01-information-Gathering
  git clone https://github.com/blackploit/hash-identifier.git
  cd hash-identifier; git pull

  cd /root/workspace/01-information-Gathering/WEB
  git clone https://github.com/al0ne/Vxscan.git
  cd Vxscan ; pip3 install -r requirements.txt ; git pull

  cd /root/workspace/01-information-Gathering
  pip3 install git+https://github.com/decoxviii/userrecon-py.git
  cd userrecon-py
  pip3 install git+https://github.com/decoxviii/userrecon-py.git --upgrade

  echo "" > /root/workspace/01-information-Gathering/userrecon-py

  mkdir /usr/share/nmap/scripts/vulnscan
  cd /usr/share/nmap/scripts/vulscan
  wget http://www.computec.ch/projekte/vulscan/download/nmap_nse_vulscan-2.0.tar.gz && tar xzf nmap_nse_vulscan-2.0.tar.gz



SCRIPT

$script_19_auto = <<-SCRIPT
  
  cd /root/workspace/19-auto
  git clone https://github.com/jm33-m0/mec.git
  cd mec ; git pull
  ./install.py




SCRIPT



$script_20_social_media = <<-SCRIPT
  
  cd /root/workspace/20-social-media


SCRIPT

$script_21_anon = <<-SCRIPT
  
  cd /root/workspace/21-anon 
  wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip -O /root/workspace/21-anon/ngrok-stable-linux-amd64.zip
  unzip ngrok-stable-linux-amd64.zip

  npm install -g beame-insta-ssl

  cd /root/workspace/21-anon
  echo "https://serveo.net/" > services.txt
  echo "    ssh -R 80:localhost:3000 serveo.net" >> services.txt
  echo "https://ngrok.com/" >> services.txt
  echo "    /root/workspace/21-anon/ngrok help" >> services.txt
  echo "    /root/workspace/21-anon/ngrok http 80" >> services.txt
  echo "    visit http://localhost:4040" >> services.txt


  git clone https://github.com/D4Vinci/Cuteit.git
  cd Cuteit ; git pull

SCRIPT

$msg = <<MSG
------------------------------------------------------
Your Kali VM is ready!
Login with "vagrant ssh" and change the root password.
------------------------------------------------------
MSG
