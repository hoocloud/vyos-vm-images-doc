#build opt for dev/debug
ansible-playbook qemu.yml -e iso_local=/tmp/vyos.iso -e cloud_init=true -e cloud_init_ds=NoCloud,ConfigDrive,None -e keep_user=true -e empty_config=true -e grub_console=serial -e consistent_network_device_naming=true -e cloud_init_disable_network_config=true

