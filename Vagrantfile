Vagrant.configure("2") do |config|
    # Ansible control server:
    config.vm.define "acs" do |acs|
        acs.vm.box = "generic/ubuntu2204"
        acs.vm.hostname = "acs"

        acs.vm.provision "shell" do |shell|
            shell.inline = <<-SHELL
                set -euo pipefail

                echo "Preventing apt-get from showing dialogs..."
                export DEBIAN_FRONTEND=noninteractive

                echo "Updating apt..."
                apt-get update

                echo "Installing pass..."
                apt-get install -y pass

                echo "Installing pip..."
                apt-get install -y python3-pip

                echo "Installing Ansible..."
                pip3 install ansible

                echo "Generating gpg key for user vagrant..."
                su -l vagrant -c "gpg --batch --gen-key /vagrant/gpg-batch.txt"

                echo "Retrieving gpg key ID..."
                gpg_key_id=`su -l vagrant -c "gpg --list-keys --with-colons | grep '^pub' | head -n 1 | awk -F: '{print $5}'"`

                echo "Initializing pass with the gpg key ID..."
                su -l vagrant -c "pass init $gpg_key_id"

                echo "Storing the vault password in pass..."
                su -l vagrant -c "echo 'Abcd1234!' | pass insert --echo 'ansible-vault'"
            SHELL
        end

        # Make sure all sensitive info is only readable by user.
        acs.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
    end
end
