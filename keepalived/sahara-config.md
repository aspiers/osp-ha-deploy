Introduction
------------

**Important:** this configuration assumes that the controller nodes can access the floating IP network (10.10.10.0/24 in the example configuration). This was not the case on [the controller node configuration](controller-node.md), because the NIC used for the provider network did not have an IP address. You can accomplish this by setting up routes in the default gateway (192.168.1.1), or creating a separate route on the controller nodes.

The following commands will be executed on all controller nodes, unless otherwise stated.

You can find a phd scenario file [here](phd-setup/sahara.scenario).

Install software
----------------

    yum install -y openstack-sahara-api openstack-sahara-engine openstack-sahara-common openstack-sahara python-saharaclient

Configure Sahara
----------------

    crudini --set /etc/sahara/sahara.conf DEFAULT host 192.168.1.22X
    crudini --set /etc/sahara/sahara.conf DEFAULT use_floating_ips True
    crudini --set /etc/sahara/sahara.conf DEFAULT use_neutron True
    crudini --set /etc/sahara/sahara.conf DEFAULT rpc_backend rabbit
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_hosts hacontroller1,hacontroller2,hacontroller3
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_port 5672
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_use_ssl False
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_userid guest
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_password guest
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_login_method AMQPLAIN
    crudini --set /etc/sahara/sahara.conf oslo_messaging_rabbit rabbit_ha_queues true
    crudini --set /etc/sahara/sahara.conf DEFAULT notification_topics notifications
    crudini --set /etc/sahara/sahara.conf database connection mysql://sahara:saharatest@controller-vip.example.com/sahara
    crudini --set /etc/sahara/sahara.conf keystone_authtoken auth_uri http://controller-vip.example.com:5000/v2.0
    crudini --set /etc/sahara/sahara.conf keystone_authtoken identity_uri http://controller-vip.example.com:35357/
    crudini --set /etc/sahara/sahara.conf keystone_authtoken admin_user sahara
    crudini --set /etc/sahara/sahara.conf keystone_authtoken admin_password saharatest
    crudini --set /etc/sahara/sahara.conf keystone_authtoken admin_tenant_name services
    crudini --set /etc/sahara/sahara.conf DEFAULT log_file /var/log/sahara/sahara.log

Manage DB
---------

On node 1:

    sahara-db-manage --config-file /etc/sahara/sahara.conf upgrade head


Start services, open firewall ports
-----------------------------------
    firewall-cmd --add-port=8386/tcp
    firewall-cmd --add-port=8386/tcp --permanent
    systemctl enable openstack-sahara-api
    systemctl enable openstack-sahara-engine
    systemctl start openstack-sahara-api
    systemctl start openstack-sahara-engine

Testing
-------

On node 1, run the following commands to test the Sahara API:

    . /root/keystonerc_admin
    sahara plugin-list

Further Sahara testing requires creating a specific virtual machine image, which is outside the scope of this document. You can find instructions on [the Sahara wiki](http://docs.openstack.org/developer/sahara/devref/quickstart.html#upload-an-image-to-the-image-service).
