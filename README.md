# docker-shutdown-patch
Mageia Bug 29096: https://bugs.mageia.org/show_bug.cgi?id=29096

The package contains the modified file `/etc/systemd/system/docker.service`. Replacing the file fixed the docker freezes timeout error when restarting or shutting down the computer.  
  
**Dependencies:** docker systemd
  
/etc/systemd/system/docker.service
```
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target
Wants=docker-storage-setup.service

[Service]
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash

# While Docker has support for socket activation (-H fd://), this is not
# enabled by default because enabling socket activation means that on boot your
# containers won't start until someone tries to administer the Docker daemon.
Type=notify
ExecStart=/usr/sbin/dockerd \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $INSECURE_REGISTRY

ExecReload=/bin/kill -s HUP $MAINPID

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this property.
TasksMax=infinity

# Set delegate yes so that systemd does not reset the cgroups of docker containers
# Only systemd 218 and above support this property.
Delegate=yes

# Kill only the docker process, not all processes in the cgroup.
KillMode=process

# Restart the docker process if it exits prematurely.
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

ExecStartPre=/usr/sbin/docker-network-cleanup
ExecStopPost=/usr/sbin/docker-network-cleanup

[Install]
WantedBy=multi-user.target
```
