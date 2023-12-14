# Wireguard VPN

This is a reference document on how to install Wireguard on a server and clients and use it as VPN. 
This guide is for a Debian 11 server but should be mostly valid for other Linux distributions.

## Server configuration

Update you system `sudo apt update && apt upgrade`.

Install Wireguard as showed [on the Wireguard website](https://www.wireguard.com/install/).

`cd` into the directory you want to store your server keys and use the command `wg genkey | tee server.key | wg pubkey > server.pub` to generate them.

Create the [/etc/wireguard/wg0.conf](wg0.conf) file:

- The `PrivateKey` should be the private key you generated (content of the *server.key* file)
- The `Address` should be a local address you want your server to have in the VPN network.
- The iptables POSTROUTING rules output need to be the name of your server interface connected to the internet.

**Optional** - If you have a firewall, add a rule to allow UDP connection on the port 5182.
For UFW you can do that with `ufw allow 51820/udp`.

You can now start the Wireguard service with `wg-quick up wg0` and verify it's running with `wg show`.

**Optional** - You can enable your interface to automatically start on boot with `systemctl enable wg-quick@wg0`.

Enable IP forwarding by adding or uncommenting the following line of */etc/sysctl.conf*:

```
net.ipv4.ip_forward = 1
```

Then reload the config with `sysctl -p /etc/sysctl.conf`.

## Client configuration

Create a [client configuration](client.conf).

- The interface `Address` should be the one of your client
- The `AllowedIp` field is the range of IP that your client will try to send request to through the VPN. If you want all requests to go through it use the range `0.0.0.0/0`
- The `DNS` field forces your clients to send DNS requests through the VPN, thus avoiding DNS leaks

You can either generate the clients private and public keys on your server then import them in your clients or do the opposite.

## Add the client to your server

To add the client connection to your server you can use two methods. Either you can add the client configuration to
your server's *wg0.conf* or you can add it with a command.

The `AllowedIPs` field determines which IPs will be routed to your peer.

### Method 1

Add the following information to your *wg0.conf*:

```
[Peer]
PublicKey = <Client Public key>
Endpoint = <Client VPN network IP>:51820
AllowedIPs = 10.21.0.2/32
```

### Method 2

Use the command `wg set wg0 peer [client pub key] endpoint [client IP]:51820 allowed-ips [client IP]`

## Verification

You should now be able to connect to your VPN! You can check your IP with a What's my IP service,
and [check your connection for DNS leaks](https://www.dnsleaktest.com/).
