# autossh-tunnel
This setup is based on this gist: https://gist.github.com/ttimasdf/ef739670ac5d627981c5695adf4c8f98

Systemd service to create an (auto) ssh tunnel from our dev vm to our kubernetes clusters.

## example usage

This role creates a new user (`autossh_tunnel_user`) and configures it to forward one or more ports from remote hosts.
For each host multiple forwards can be configured.
Each forward needs the `remote_host` and `remote_port` and a `local_port`. The `bind_address` is optional.
As an alternative to TCP Ports, unix sockets can be used.
They are specified in the respective `_port` variable, while setting the `address` or `host` to an empty string.


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
        forwards:
          - bind_address: "localhost"
            local_port: "1234"
            remote_host: "ssh.example.com"
            remote_port: "1234"
          - local_port: "5678"
            remote_host: "ssh2.example.com"
            remote_port: "5678"
          - local_port: "/var/run/mysql.sock"
            remote_port: "/var/run/mysql.sock"
      - name: "example2.com"
        forwards:
          - bind_address: "localhost"
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

> [!Note]
> *Excerpt from the open ssh man page:* LocalForward
> Specifies that a TCP port or Unix-domain socket on the local machine be forwarded over the secure channel to the specified host and port (or Unix-domain socket) from the remote machine.
> For a TCP port, the **first argument** must be `[bind_address:]port` or a Unix domain `socket path`.
> The **second argument** is the destination and may be `host:hostport` or a Unix domain `socket path` if the remote host supports it.
> **IPv6** addresses can be specified by enclosing addresses in square brackets.
> If either argument contains a '/' in it, that argument will be interpreted as a Unix-domain socket (on the corresponding host) rather than a TCP port.
> Multiple forwardings may be specified, and additional forwardings can be given on the command line.
> Only the superuser can forward privileged ports. By default, the local port is bound in accordance with the `GatewayPorts` setting.
> However, an explicit `bind_address` may be used to bind the connection to a specific address.
> The `bind_address` of `localhost` indicates that the listening port be bound for local use only, while an empty address or `*` indicates that the port should be available from all interfaces.
> Unix domain socket paths may use the tokens described in the `TOKENS` section and environment variables as described in the `ENVIRONMENT VARIABLES` section.
> *Source: https://man.openbsd.org/ssh_config*
