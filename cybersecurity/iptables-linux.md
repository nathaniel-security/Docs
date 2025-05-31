# iptables-linux

## Introduction

* used for managing the firewall rules and network traffic
* it can act as a router
* can also filter packets
* it can interact with all protocoals from the
  * `/etc/protocols`
* iptables rules are not persistent
  * to save ip tables

```
sudo apt install iptables-persistance
```

*
* The firewall matches packets with rules defined in these tables and then takes the specified action on a possible match.
  * Tables is the name for a set of chains.
    * filter table
      * input chain
        * everything coming into network
      * output chain
        * everything going out of network
      * forward chain
        * everything forward
    * nat table
      * output chain
      * pre-routing chain
        * for altering packets as soon as they come in
      * post routing chain
        * for altering packets as they are about to go out
    * mangle table
      * input chain
      * output chain
      * forward chain
      * pre routing chain
      * post routing chain
  * Chain is a collection of rules.
  * Rule is condition used to match packet.
  * Target is action taken when a possible rule matches. Examples of the target are ACCEPT, DROP, QUEUE.
  * Policy is the default action taken in case of no match with the inbuilt chains and can be ACCEPT or DROP.

```
iptables --table TABLE -A/-C/-D... CHAIN rule --jump Target
```

### Tables in Iptables

* There are five possible tables as follows:
  * filter:
    * USED filtering input and output traffic
      * simply deals with the firewall aspect
  * nat
    * used to redirect connection to other interfaces and networks
  * mangle
    * focuses on alternating packets
  * raw : Configures exemptions from connection tracking. Built-in chains are PREROUTING and OUTPUT.
  * security : Used for \[\[Mandatory Access Control (MAC)]]

## Commands

### Flush ip tables

```
iptables -F
```

### List tables

```
iptables -L
```

* default is filter chain

### Accept traffic

```
sudo iptables --policy INPUT ACCEPT
```

* allows all traffic

### Deny Traffic

```
sudo iptables --policy INPUT DROP
```

* denys all traffic

### Block a particular ip

```
iptables -I INPUT -d 1.1.1.1 -j DROP 
```

* in the above command you will block all input traffic with the destination ip `1.1.1.1`
*

    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
* you will get back loss since the packets are going to `1.1.1.1`hello world
*

```
iptables -I INPUT -s 1.1.1.1 -j DROP 
```

* in the above you will block all `INPUT` packet with the source `1.1.1.1`
* during ssh connection
  * if i drop packets and allow in couple of seconds my connection is not terminated
    * guessing a keep alive is on

### Nat

* **NAT** modifies the source or destination IP address of packets.
* useful for hiding private IP addresses
  * **SNAT** Source NAT
    * Change the **source** address of outgoing traffic
      * useful when many devices in your network share one public IP
  * **DNAT** Destination NAT
    * Change the **destination** address of incoming traffic
      * useful to redirect external traffic to internal servers
  * **Masquerade**
    * A type of SNAT for dynamic IP addresses.
* PRE Routing vs POST Routing
  *

      <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
  * pre routing applies to before the packet is redirected to a process

```bash
sudo su
```

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### Task

* convert linux box into router between two networks
  * connect 2 other computers
* block traffic to a particular source and destinations

```
iptables -t nat -A OUTPUT -o lo -p tcp --dport 80 -j REDIRECT --to-port 8080
```

* https://askubuntu.com/questions/444729/redirect-port-80-to-8080-and-make-it-work-on-local-machine

## References

* https://www.geeksforgeeks.org/iptables-command-in-linux-with-examples/
* https://www.youtube.com/watch?v=6Ra17Qpj68c
* https://www.youtube.com/watch?v=x0h\_CSmbyxg
* NAT
  * https://www.youtube.com/watch?v=NAdJojxENEU\&ab\_channel=HusseinNasser

### Question

#### What is the difference between netfilter and iptables?

* Iptables is just a command-line tool used to add or remove netfilter rules

#### what is the network stucture of the linux os
