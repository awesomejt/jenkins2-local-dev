# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "jenkins-centos7"
  config.vbguest.auto_update = true
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
  end

  config.vm.provision "shell", inline: <<-SHELL
     yum update -y
     yum install -y nano wget curl java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel

     if [ ! -d /usr/local/git/bin ]; then
       yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils perl-ExtUtils-MakeMaker
       cd /usr/src
       wget https://www.kernel.org/pub/software/scm/git/git-2.12.1.tar.gz
       tar xzf git-2.12.1.tar.gz
       cd git-2.12.1
       make prefix=/usr/local/git all
       make prefix=/usr/local/git install
       echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
       source /etc/bashrc
     fi

     if [ ! -d /var/lib/jenkins ]; then
       wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
       rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
       yum install -y jenkins

       echo 'JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Djenkins.install.runSetupWizard=false"' >> /etc/sysconfig/jenkins

       chkconfig jenkins on
       systemctl start jenkins

       echo "Waiting on Jenkins to Start"
       sleep 30s

       if [ -f /vagrant/plugins.xml ]; then
         echo "Telling Jenkins to install intial set of plugins"
         curl -X POST -d '@/vagrant/plugins.xml' --header 'Content-Type: text/xml' http://localhost:8080/pluginManager/installNecessaryPlugins
       else
         echo "Unable to POST plugins file"
       fi

       if [ -f /vagrant/jobs/local-seed-job.xml ]; then
         echo "Creating local seed job"
         curl -s -X POST 'http://localhost:8080/createItem?name=SeedJob-Local' --data-binary '@/vagrant/jobs/local-seed-job.xml' -H "Content-Type: text/xml"
       else
         echo "Unable to create local seed job"
       fi

       if [ -f /vagrant/jobs/git-seed-job.xml ]; then
         echo "Creating SCM based seed job"
         curl -s -X POST 'http://localhost:8080/createItem?name=SeedJob-Develop' --data-binary '@/vagrant/jobs/git-seed-job.xml' -H "Content-Type: text/xml"
       else
         echo "Unable to create SCM based seed job"
       fi

       sleep 30s
       systemctl restart jenkins
     fi
  SHELL
end
