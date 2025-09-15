# autossh-tunnel
This setup is based on this gist: https://gist.github.com/ttimasdf/ef739670ac5d627981c5695adf4c8f98

Systemd service to create an (auto) ssh tunnel from our dev vm to our kubernetes clusters.

## example usage

~~~yaml
---
- name: Install the autossh tunnel
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - autossh_tunnel
  vars:
    autossh_tunnel_hosts:
      - name: "example.com"
        local_host: "localhost"
        local_port: "1234"
        remote_host: "ssh.example.com"
        remote_port: "1234"
      - name: "example2.com"
        local_host: "localhost"
        local_port: "1234"
        remote_host: "ssh.example.com"
        remote_port: "1234"
    autossh_tunnel_user: "tunnel"
~~~

## manual install
~~~bash
# Create a new user, that runs the tunnel.
export TUNNEL_USER="tunnel"
sudo useradd -g nogroup -s /bin/false -m ${TUNNEL_USER}

# create the .ssh directory for the ${TUNNEL_USER}
# place your ssh key in this directory
sudo -u ${TUNNEL_USER} mkdir -p ~${TUNNEL_USER}/.ssh 

# configure your ssh connection (see section ssh config)
sudo -u ${TUNNEL_USER} nano ~${TUNNEL_USER}/.ssh/config

# replace example.com with your desired ssh host like in your `.ssh/config`
export SSH_HOST="example.com"

# test the connection to ${SSH_HOST} with the ${TUNNEL_USER}
sudo -u ${TUNNEL_USER} ssh ${SSH_HOST}

# clone this repository
git clone git@github.com:dBildungsplattform/infra-ansible-collections.git
cd infra-ansible-collections

# copy the system.d file
cp "roles/autossh_tunnel/templates/autossh@.service" "/etc/systemd/system/autossh@${SSH_HOST}.service"

# enable and start the service
systemctl enable autossh@${SSH_HOST}.service --now

# check the status of the service
systemctl status autossh@${SSH_HOST}.service
~~~

## ssh config

The ssh-config should look something like this:

~~~
Host example.com
  LocalForward local-ip:local-port example.com:remote-port
  ExitOnForwardFailure yes
  ServerAliveInterval 10
  ServerAliveCountMax 3
~~~
