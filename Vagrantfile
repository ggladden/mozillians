require "yaml"

# Load up our vagrant config files -- vagrantconfig.yaml first
_config = YAML.load(File.open(File.join(File.dirname(__FILE__),
                    "vagrantconfig.yaml"), File::RDONLY).read)

# Local-specific/not-git-managed config -- vagrantconfig_local.yaml
begin
  _config.merge!(YAML.load(File.open(File.join(File.dirname(__FILE__),
                 "vagrantconfig_local.yaml"), File::RDONLY).read))
rescue Errno::ENOENT # No vagrantconfig_local.yaml found -- that's OK; just
                     # use the defaults.
end

CONF = _config
MOUNT_POINT = '/home/vagrant/mozillians'

Vagrant::Config.run do |config|

    config.vm.box = "ubuntu-lucid-32-openldapV2.box"
    config.vm.box_url = "http://people.mozilla.com/~ddash/mozillians-v4.box"

    config.vm.forward_port("web", 8001, 8001)
    config.vm.forward_port("ldap", 1389, 1389)

    # Increase vagrant's patience during hang-y CentOS bootup
    # see: https://github.com/jedi4ever/veewee/issues/14
    config.ssh.max_tries = 50
    config.ssh.timeout   = 300

    # nfs needs to be explicitly enabled to run.

    if CONF['nfs'] == false or RUBY_PLATFORM =~ /mswin(32|64)/
        config.vm.share_folder("v-root", MOUNT_POINT, ".")
    else
        config.vm.share_folder("v-root", MOUNT_POINT, ".", :nfs => true)
    end

    # Add to /etc/hosts: 33.33.33.24 dev.mozillians.org
    config.vm.network "33.33.33.24"

    config.vm.provision :puppet do |puppet|
        puppet.manifests_path = "puppet/manifests"
        puppet.manifest_file  = "dev-vagrant.pp"
    end

end
