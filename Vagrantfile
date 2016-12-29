VAGRANTFILE_API_VERSION = '2'

EPEL_REPO_6 = '''
[epel]
name     = EPEL 6 - \$basearch
baseurl  = http://mirror.globo.com/epel/6/\$basearch
enabled  = 1
gpgcheck = 0
'''

EPEL_REPO_7 = '''
[epel]
name     = EPEL 7 - \$basearch
baseurl  = http://mirror.globo.com/epel/7/\$basearch
enabled  = 1
gpgcheck = 0
'''

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
    vb.customize [ 'modifyvm', :id, '--memory', '512' ]
    vb.customize [ 'modifyvm', :id, '--nictype1', 'virtio' ]
    vb.customize [ 'modifyvm', :id, '--natdnshostresolver1', 'on' ]
    vb.customize [ 'modifyvm', :id, '--natdnsproxy1', 'on' ]
  end
      
  config.vm.define 'ubuntu-trusty' do |ubuntu_t|
    ubuntu_t.vm.box      = 'ubuntu/trusty64'
    ubuntu_t.vm.hostname = 'ubuntu-trusty'
    
    ubuntu_t.vm.provision 'shell', inline: 'add-apt-repository ppa:openjdk-r/ppa'
    ubuntu_t.vm.provision 'shell', inline: 'apt-get update'
    ubuntu_t.vm.provision 'shell', inline: 'apt-get install -y -qq  python-pip libffi-dev libssl-dev python-dev'
    ubuntu_t.vm.provision 'shell', inline: 'apt-get install -y -qq  openjdk-8-jre'
    ubuntu_t.vm.provision 'shell', inline: 'pip install ansible==2.2.0.0 ansible-lint jinja2'

    ubuntu_t.vm.provision 'ansible' do |ansible| 
      ansible.playbook = 'tests/test_vagrant.yml'
      ansible.extra_vars = {
        filebeat_user:  'root',
        filebeat_group: 'root',
        filebeat_create_user: false,
      }
    end
    
  end

  config.vm.define 'ubuntu-precise' do |ubuntu_p|
    ubuntu_p.vm.box      = 'ubuntu/precise64'
    ubuntu_p.vm.hostname = 'ubuntu-precise'
    
    ubuntu_p.vm.provision 'shell', inline: 'add-apt-repository ppa:openjdk-r/ppa'
    ubuntu_p.vm.provision 'shell', inline: 'apt-get update'
    ubuntu_p.vm.provision 'shell', inline: 'apt-get install -y -qq  python-pip libffi-dev libssl-dev python-dev'
    ubuntu_p.vm.provision 'shell', inline: 'apt-get install -y -qq  openjdk-8-jre'
    ubuntu_p.vm.provision 'shell', inline: 'pip install ansible==2.2.0.0 ansible-lint jinja2'

    ubuntu_p.vm.provision 'ansible' do |ansible| 
      ansible.playbook = 'tests/test_vagrant.yml'
      ansible.extra_vars = {
        filebeat_user:  'root',
        filebeat_group: 'root',
        filebeat_create_user: false,
      }
    end
    
  end

  config.vm.define 'centos-6' do |centos6|
    centos6.vm.box      = "puppetlabs/centos-6.6-64-nocm"
    centos6.vm.hostname = 'centos-6'

    centos6.vm.provision 'shell', inline: 'yum install -y ca-certificates'
    centos6.vm.provision 'shell', inline: "echo \"#{EPEL_REPO_6}\" > /etc/yum.repos.d/epel.repo"
    centos6.vm.provision 'shell', inline: 'yum install -y python-pip python-devel gcc libffi-devel openssl-devel'
    centos6.vm.provision 'shell', inline: 'yum install -y java-1.8.0-openjdk.x86_64'
    centos6.vm.provision 'shell', inline: 'pip install -q pip --upgrade'
    centos6.vm.provision 'shell', inline: 'pip install -q ansible==2.2.0.0 ansible-lint jinja2'

    centos6.vm.provision 'ansible' do |ansible| 
      ansible.playbook   = 'tests/test_vagrant.yml'
      ansible.extra_vars = {
      }
    end
  end

  config.vm.define 'centos-7' do |centos7|
    centos7.vm.box      = 'centos/7'
    centos7.vm.hostname = 'centos-7'

    centos7.vm.provision 'shell', inline: 'yum install -y ca-certificates'
    centos7.vm.provision 'shell', inline: "echo \"#{EPEL_REPO_7}\" > /etc/yum.repos.d/epel.repo"
    centos7.vm.provision 'shell', inline: 'yum install -y python-pip python-devel gcc libffi-devel openssl-devel'
    centos7.vm.provision 'shell', inline: 'yum install -y java-1.8.0-openjdk.x86_64'
    centos7.vm.provision 'shell', inline: 'pip install -q pip --upgrade'
    centos7.vm.provision 'shell', inline: 'pip install -q ansible==2.2.0.0 ansible-lint jinja2'

    centos7.vm.provision 'ansible' do |ansible| 
      ansible.playbook = 'tests/test_vagrant.yml'
      ansible.extra_vars = {
      }
    end
  end

end
  
# vi:ts=2:sw=2:et:ft=ruby:
