UniFi Network
===============

In 2022, we upgraded our WiFi network to a UniFi system made by Ubiquiti. This documents some of the setup and some of the problems we faced.


Running in Docker
-------------------

We have a docker-compose file that we used here (note this may not be up to date):

.. code-block::

    version: '2.3'
    services:
      mongo:
        image: mongo:3.6
        container_name: unifi_mongo
        networks:
          - default
        restart: unless-stopped
        volumes:
          - ./mongo-data/db:/data/db
          - ./mongo-data/dbcfg:/data/configdb
      controller:
        #image: "jacobalberty/unifi:${TAG:-latest}"
        image: "jacobalberty/unifi:v7.1.66"
        container_name: unifi_controller
        depends_on:
          - mongo
        init: true
        networks:
          - default
        restart: unless-stopped
        volumes:
          - ./unifi-data-dir/dir:/unifi
          - ./unifi-data-dir/data:/unifi/data
          - ./unifi-data-dir/log:/unifi/log
          - ./unifi-data-dir/cert:/unifi/cert
          - ./unifi-data-dir/init:/unifi/init.d
          - ./unifi-data-var/run:/var/run/unifi
          # Mount local folder for backups and autobackups
          - ./unifi-backup:/unifi/data/backup
        user: unifi
        sysctls:
          net.ipv4.ip_unprivileged_port_start: 0
        environment:
          DB_URI: mongodb://mongo/unifi
          STATDB_URI: mongodb://mongo/unifi_stat
          DB_NAME: unifi
        ports:
          - "3478:3478/udp" # STUN
          - "6789:6789/tcp" # Speed test
          - "8080:8080/tcp" # Device/ controller comm.
          - "8443:8443/tcp" # Controller GUI/API as seen in a web browser
          - "8880:8880/tcp" # HTTP portal redirection
          - "8843:8843/tcp" # HTTPS portal redirection
          - "10001:10001/udp" # AP discovery
      logs:
        image: bash
        container_name: unifi_logs
        depends_on:
          - controller
        command: bash -c 'tail -F /unifi/log/*.log'
        restart: unless-stopped
        volumes:
          - ./unifi-data-dir/log:/unifi/log


    networks:
      default:
        name: $DOCKER_MY_NETWORK

Notice that we do not override the network mode.
This is important because a lot of our troubles could have been solved if we decided to use the ``host`` network mode.
If we did that, then the controller software would recognize that its IP Address is ``192.168.X.X``.
However, when using the default network mode (`bridge <https://docs.docker.com/network/bridge/>`_),
the controller software will recognize its IP Address as ``172.X.X.X``, which will correctly refer to the container
when you are ON the machine running the controller software. However, when an AP attempts to connect to that IP Address,
it will not be able to find it because it tries to refer to the 172 address.

To fix the above, you just have to "Override Inform Host" on the `settings <https://unifi.wildmountainfarms.com/manage/default/settings/system>`_ page.
You set this value to the IP Address of the machine the controller software is running on (a 192 address).

.. note::

    When launching from https://network.unifi.ui.com, you must make sure that you have "Launch Using Hostname" selected.
    If you do not, it will attempt to launch using the 172 address, which will not work.



Setting up with Caddy
-----------------------

Since we are running this in docker, setting it up with caddy is a breeze. 
We are able to access the UniFi web interface by going to https://unifi.wildmountainfarms.com.

.. note::

    Once caddy is set up, you don't need many ports open like we specified in ``docker-compose.yml``. 
    However, we keep those ports open so SSO works.


Node Wireless Uplink Over Wired
-----------------------------------

We spent a while trying to figure out why a node wouldn't be wired.
If a node is plugged into a switch, it will think it has Ethernet even if that switch isn't connected to your main netork...

API
-----

There seems to be an undocumented API that we could use later.

The POST request goes to 
