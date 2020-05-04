Build instructions for building VyOS on Debian Buster
=====================================================

If you use a cloud image of Debian Buster
-----------------------------------------

cloud-init networking may be broken on Debian cloud image

.. code-block:: shell 

    echo 'source /etc/network/interfaces.d/*' >/etc/network/interfaces
    echo "nameserver 1.1.1.1" > /etc/resolv.conf

linux-image-cloud-amd64 which is used in cloud image of Debian Buster does not contain the aufs kernel module required for building vyos images Therefor we need to install linux-image-amd64 and make it the default boot entry

.. code-block:: shell 

    apt-get update
    DEBIAN_FRONTEND=noninteractive apt-get -y --assume-yes --quiet -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" dist-upgrade 
    DEBIAN_FRONTEND=noninteractive apt-get -y --assume-yes --quiet -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" install linux-image-amd64 

    grep '^\s*GRUB_DISABLE_SUBMENU' /etc/default/grub && sed -i  's/GRUB_DISABLE_SUBMENU=.*$/GRUB_DISABLE_SUBMENU=y/g' /etc/default/grub || echo "GRUB_DISABLE_SUBMENU=y" >> /etc/default/grub
    sed -i  's/#GRUB_TERMINAL/GRUB_TERMINAL/g' /etc/default/grub
    update-grub

    BOOTENTRY="`awk '/menuentry/ && /class/ {count++; print count-1"****"$0 }' /boot/grub/grub.cfg |grep -v '(recovery mode)'|grep -v -- '-cloud-amd64' |awk -F '*' '{print $1}'`"
    sed -i  "s/GRUB_DEFAULT=.*$/GRUB_DEFAULT=$BOOTENTRY/g" /etc/default/grub
    update-grub

    reboot

    DEBIAN_FRONTEND=noninteractive apt-get -y --assume-yes --quiet -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" purge linux-image-cloud-amd64 linux-image-\*cloud-amd64

    BOOTENTRY="`awk '/menuentry/ && /class/ {count++; print count-1"****"$0 }' /boot/grub/grub.cfg |grep -v '(recovery mode)'|grep -v -- '-cloud-amd64' |awk -F '*' '{print $1}'`"
    sed -i  "s/GRUB_DEFAULT=.*$/GRUB_DEFAULT=$BOOTENTRY/g" /etc/default/grub
    update-grub

Install required packages for vyos-vm-images and a bit more
-----------------------------------------------------------

.. code-block:: shell 

    DEBIAN_FRONTEND=noninteractive apt-get -y --assume-yes --quiet -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" install mc bash-completion vim git ansible python

Clone vyos-vm-images
--------------------

.. code-block:: shell 
    
    mkdir /root/vyos
    cd /root/vyos
    git clone https://github.com/hoocloud/vyos-vm-images.git
    cd vyos-vm-images/
    git checkout hoocloud

Build VyOS image for hoocloud (dev/debug)
-----------------------------------------

.. code-block:: shell 
    

    ansible-playbook qemu.yml -e iso_local=/tmp/vyos.iso -e cloud_init=true -e cloud_init_ds=NoCloud,ConfigDrive,None -e keep_user=true -e empty_config=true -e grub_console=serial -e consistent_network_device_naming=true -e cloud_init_disable_network_config=true
