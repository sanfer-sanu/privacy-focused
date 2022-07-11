# Wireguard

### Server Side Setup

- Log into server and make sure system is up to date
`apt-get update && apt-get upgrade`
Reboot if there are update that requires reboot.

#### Enable IP Forwarding
- We need to enable IP Forwarding. IP forwarding is the ability for an operating system to accept incoming network packets on one interface, recognize that it is not meant for the system itself, but that it should be passed on to another network. Edit the file `/etc/sysctl.conf` and change and uncomment to the line that says `net.ipv4.ip_forward=1`
- Now reboot or run `sysctl -p` to activate the changes.

#### Install wireguard
- Run below command to install wireguard
    ```sh
    apt-get install wireguard
    ```
#### Configure wireguard server
- Go to to the Wireguard config `cd /etc/wiregaurd` and then run the following command to generate the public and private keys for the server.
    ```sh
    umask 077; wg genkey | tee privatekey | wg pubkey > publickey
    ```

- Then run `cat privatekey` and copy it so we can put it in to the server config file.

- Create the `/etc/wireguard/wg0.conf`
    ```sh
    vim /etc/wireguard/wg0.conf
    ```
- This will get the basic server side setup and we will be coming back to this file to add the peers later.
    ```sh
    [Interface]
    PrivateKey = <Your Private Key Goes Here>
    Address = 10.100.10.100/24 # set-ip-address-for-the-wireguard-server
    ListenPort = 51820 # listening port 
    SaveConfig = true
    PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A FORWARD -o %i -j ACCEPT
    PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D FORWARD -o %i -j ACCEPT
    ```
#### Start & Stop the server
- To test that the server works run `wg-quick up wg0` to bring up the interface. Running `wg-quick down` will bring the interface down.
- If you want the `wg0` interface to be active on boot you need to run `systemctl enable wg-quick@wg0`
- Then you can use to `systemctl start wg-quick@wg0` start the server, `systemctl stop wg-quick@wg0` stop the server and `systemctl status wg-quick@wg0` to check the status.

### Linux Client Side Setup

#### Install wireguard
- Make sure the system is up to date.
`apt-get update && apt-get upgrade`
Reboot if there are update that requires reboot.
    ```sh
    apt-get install wireguard
    ```

#### Configure client side
- Go to to the Wireguard config `cd /etc/wiregaurd` and then run the following command to generate the public and private keys for the server.
    ```sh
    umask 077; wg genkey | tee privatekey | wg pubkey > publickey
    ```

- The run `cat privatekey` and copy it so we can put it in to the server config file.

- Create the /etc/wireguard/wg-client.conf
    `vim /etc/wireguard/wg-client.conf`
    ```sh
    [Interface]
    PrivateKey = <Your Private Key Goes Here>
    Address = 10.100.10.150/24
    DNS = 1.1.1.1 #set-your-dns-address
    
    [Peer]
    PublicKey = <Server Public Key>
    AllowedIPs = 0.0.0.0/0
    Endpoint = <Server IP Address>:51820
    ```
    Run `wg-quick up wg-client` to make sure the system comes up and run `wg-quick down wg-client` to take down the interface.
    
#### Adding clients to server
- To add client as a peer on server, run
    ```sh
    wg set wg0 peer <Client Public Key> allowed-ips <Client IP Address>
    ```
    eg: `wg set wg0 peer cVU13uIpVWxCPE4RYWawViI= allowed-ips 10.100.10.150`

- To remove a client peer from server, run
    ```sh
    wg set wg0 peer <Client Public Key> remove
    ```
    Eg: `wg set wg0 peer c0PKOPgKlrla+c9SwU= remove`
    
- Once both sides have been complete and Wireguard restarted on both side the system should be able to communicate.

#### Test client and server connectivity
- Run `wg show` to make sure the wireguard is running and IP address, port number is matching.
- Ping the server from client by running `ping 10.100.10.100`
- Then try getting out the internet by running `ping 1.1.1.1`

#### Turning on the UFW firewall on the server
- It is easy to enable the UFW firewall there are a few ports we need to open first, port 22 TCP for ssh management and 51820 UDP for Wireguard. To do this simply:

    ```sh
    ufw allow 22/tcp
    ```
    ```sh
    ufw allow 51820/udp
    ```
    ```sh
    ufw enable
    ```

#### Enjoy!