Configure a bare EL7 host as a personal OpenVPN server.

I found this arrangement useful when travelling with my laptop.

Configuration has been persisted here in case I should need it again, and to
remind myself of what was necessary to properly secure my client.


# Requirements

- A laptop or whatever device on which you normally compute.  This local machine
  will act as your travelling OpenVPN client.

- A headless machine somewhere on the Internet.  This remote machine will act as
  your OpenVPN server.

- [Ansible](https://www.ansible.com/).  The Ansible playbook contained within
  this repository will be used to configure your remote machine as a personal
  (non-shared) OpenVPN server.


# Usage

## Remote set up

1. Rent a virtual machine from the provider and political entity of your choice.

2. Install an EL7-derived operating system, like CentOS, on the remote machine.

3. Ensure you are able to SSH to the remote machine as root.

4. Run the `openvpn.yml` Ansible playbook against the remote machine.  This step
   will generate an OpenVPN static key and write it to
   `/etc/openvpn/static.key`.  Using a secure transport, like SSH, take a copy
   of this key.

5. Reboot the remote machine.


## Local set up

1. Install an OpenVPN client on the local machine.

2. Configure the OpenVPN client.  Supply it with the copy of the OpenVPN static
   key taken previously.  A sample configuration may be found below.

3. Configure the local machine's DNS resolver to send its requests to some
   public recursor.  This traffic will be tunnelled.

4. Deconfigure IPv6 on the local machine.

5. Configure a packet filter on the local machine that will drop or reject all
   traffic that does not egress via the OpenVPN tunnel.  A sample `pf`
   configuration may be found below.

Sample OpenVPN client configuration:

    verb 3
    
    dev tun
    # Replace with the network address of the remote OpenVPN server.
    remote 1.2.3.4
    ifconfig 10.8.0.2 10.8.0.1
    redirect-gateway def1
    
    cipher AES-256-CBC
    secret static.key
    
    keepalive 10 60
    
    mute-replay-warnings

Sample `pf` configuration (for macOS):

    block return out on ! lo0
    # Replace with the network address of the remote OpenVPN server.
    pass out to 1.2.3.4
    pass out on utun1


# Caveats

The configuration as it stands will not tunnel IPv6 traffic.

When configured in static key mode, the OpenVPN server will not push DHCP
options and other configuration parameters to clients.
