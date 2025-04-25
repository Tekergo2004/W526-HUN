# ROUTING

```cisco
ipv6 unicast-routing
```

## TUNNELING

(R1) fa0/0{1.1.1.1}< -- ISP -- >{2.2.2.2}fa0/0 (R2)

### Neccessary config

```cisco
ipv6 unicast-routing
```

### Tunnel configuration

R1:

```cisco
int tunnel0 
 tunnel source fa0/0
 tunnel destination 2.2.2.2
 tunnel mode ipv6ip
 ipv6 add 1111::1/64
```

## VRF

### Creating VRF

```cisco
vrf definition Customer1
```

```cisco
ip vrf Customer1
```

If you want to enable different address families, you have to type address-family {ipv4/ipv6} in vrf configuration mode

### Adding VRF to an interface

```cisco
int g0/1
 ip vrf forwarding Customer1
```

### Configure routing for VRFs

```cisco
ip route vrf Customer1 0.0.0.0 0.0.0.0 1.1.1.1
```

```cisco
router ospf x vrf Customer1
 address-family ipv4 unicast vrf CustomerX

router ospfv3 1 
 address-family ipv6 unicast vrf CustomerX
```

```cisco
router eigrp IPCisco
 address-family ipv4/ipv6 unicast vrf Customer1 autonomous-system 100
```

```cisco
router bgp 100
 address-family ipv4/ipv6 vrf Customer1
```

### Show commands

- sh ip vrf
- sh ip route vrf Customer1 
- ping vrf Customer1 1.1.1.1

## BGP

(R1)[AS 1] fa0/0{1.1.1.1}< --- >{1.1.1.2}fa0/0 [AS 2](R2)
If it has a route to a subnet, you can add a network in bgp.
R1:

```cisco
router bgp 1
 neighbor 1.1.1.2 remote-as 2
```

R2:

```cisco
router bgp 2
 neighbor 1.1.1.1 remote-as 1
```

### iBGP

Configure connection between private subnet using remote with the same AS number.

### Best path selection

#### Route refresh

```cisco
clear ip bgp x.x.x.x soft in
```

#### Manipulate weight

The highest **_weight_** will be prefered. The deafult **_weight_** of the paths will be **_0_**.

```cisco
neighbor x.x.x.x weight 1
```

#### Manipulating LOCAL PREFERENCE

The highest _**local preference**_ is preferred. It does that, prefer the next hop address of x.x.x.x  for the z.z.z.z/24 prefix. The deafult is **_100_**.

```cisco
route-map RM_LOCAL_PREF permit 10
 set local-preference 101
 exit

router bgp 100
 neighbor x.x.x.x route-map RM_LOCAL_PREF in
 end

clear ip bgp x.x.x.x soft in
```

#### Manipulating AS PATH

The shorthest _**AS Path**_ is preferred. It's adding additional hops to the AS Path to the network. It can be in- or outbund rule. This configuration will append AS 200 twice to the end of the path.

```cisco
route-map RM_AS_PATH_PREPEND
 set as-path prepend 200 200
exit

router bgp 100
 neighbor x.x.x.x route-map RM_AS_PATH_PREPEND in
end

clear ip bgp x.x.x.x soft in
```

#### Manipulating MED

The lowest _**MED**_ is preferred. The default value is **_0_**.

```cisco
route-map RM_MED premit 10
 set metric 1
 exit

router bgp 100
 neighbor x.x.x.x route-map RM_MED out
 end

clear ip bgp 1.1.1.1 soft in
```

#### Instlaling Multiple Paths

You can set how much path can be learned for the subnet (deafult is to select one best path).

```cisco
router bgp 100
 maximum-paths 2
 end
```

## OSPFv3:

### Neccessary configurations

```cisco
ipv6 unicast-routing
ipv6 cef
```

### Interface config

```cisco
interface g0/1
 ipv6 add ..
 ipv6 add fe80::1 link-local
 ipv6 ospf 1 area 0
```

### Area config

```cisco
ipv6 router ospf 1
 router-id 1.1.1.1
 area 2 stub (--> stub area megadása, ha használunk több areát)
```

## EIGRP:

### Default config

```cisco
router eigrp x
 eigrp router-id 1.1.1.1 ???
 redistribute static
 no auto-summary
 ip summary-address eigrp 1 192.168.0.0 255.255.252.0 5
 redistribute dforg-prot metric 1000 0 1 255 255 
```

### Messeage authentication

```cisco
key chain CHAINNAME
 key x
  key-string password
  exit
 exit
```

```cisco
interface g0/1 (link-where you want to authenticate EIGRP messages)
 ip authentication mode eigrp x md5
 ip authentication key-chain eigrp x CHAINNAME
```

### Show commands

- sh ip eig nei
- sh ip eig top
- sh run | sec eigrp
- sh ip protocols
- sh ip route

# HSRP

## Default configuration

```cisco
interface g0/0 (link to client network)
 standby version 2
 standby [0-4095] 
  - ip gateway-address
  - priority [1-255] (higher is active)
  - preempt
  - authentication
    - password (max 8 char)
    - md5 
      - key-chain CHAINNAME
      - key-string password (max 64 char)
    - text password (max 8 char)
```

## Tracking using SLA

### Set up SLA

```cisco
ip sla x
icmp-echo dst-add source-interface link-to-dst
ip sla schedule x life forever start-time now
track y ip sla x reachability
```

### Set up HSRP to use SLA tracking

(You have a proper working HSRP using preemption and priorty setting)

```cisco
int link-to-dst
 standby z track y decrement 10
                 ^            ^
                 |            |
                 |            |
             track-number     |
                              |
           how much the priority will be decremented
```

### Show commands

- sh standby
- sh track

# Radius for SSH

```cisco
username backup priv 15 sec password
aaa new-model
aaa authentication login default group radius local
radius server RADIUS
  address ipv4 192.168.1.15 auth-port 1812 acct-port 1813
  key winRadius
  exit
line vty 0 15
  login authentication default 
  exit
```

Another configuration mode:
```cisco
aaa new-model

radius server HQ-SVR1
 address ipv4 10.1.21.12 auth-port 1812 acct-port 1813
 key Passw0rd

aaa group server radius HQ-RADIUS
 server name HQ-SVR1

aaa authentication login default group HQ-RADIUS local
aaa authorization exec default group HQ-RADIUS local
```
