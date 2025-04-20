# ROUTING:
```
ipv6 unicast-routing
```

## TUNNELING:
(R1) fa0/0{1.1.1.1}< -- ISP -- >{2.2.2.2}fa0/0 (R2)
### Neccessary config
```
ipv6 unicast-routing
```

### Tunnel configuration
R1
```
int tunnel0 
 tunnel source fa0/0
 tunnel destination 2.2.2.2
 tunnel mode ipv6ip
 ipv6 add 1111::1/64
```

## VRF:
### Creating VRF
```
vrf definition Customer1
```

```
ip vrf Customer1
```

If you want to enable different address families, you have to type address-family {ipv4/ipv6} in vrf configuration mode

### Adding VRF to an interface
```
int g0/1
 ip vrf forwarding Customer1
```

### Configure routing for VRFs
```
ip route vrf Customer1 0.0.0.0 0.0.0.0 1.1.1.1
```

```
router ospf x vrf Customer1
 address-family ipv4 unicast vrf CustomerX

router ospfv3 1 
 address-family ipv6 unicast vrf CustomerX
```

```
router eigrp IPCisco
 address-family ipv4/ipv6 unicast vrf Customer1 autonomous-system 100
```

```
router bgp 100
 address-family ipv4/ipv6 vrf Customer1
```



### Show commands
- sh ip vrf
- sh ip route vrf Customer1 
- ping vrf Customer1 1.1.1.1

## BGP:
(R1)[AS 1] fa0/0{1.1.1.1}< --- >{1.1.1.2}fa0/0 [AS 2](R2)
If it has a route to a subnet, you can add a network in bgp.
R1:
```
router bgp 1
 neighbor 1.1.1.2 remote-as 2
```

R2:
```
router bgp 2
 neighbor 1.1.1.1 remote-as 1
```

### iBGP
Configure connection between private subnet using remote with the same AS number. 

### Best path selection

#### Route refresh

```
clear ip bgp x.x.x.x soft in
```

#### Manipulate weight
The highest **weight** will be prefered. The deafult **weight** of the paths will be **0**
```
neighbor x.x.x.x weight 1
```

```
route-map RM_AS_PATH_PREPEND
 set as-path prepend 200 200
exit
router bgp 100
 neighbor 1.1.1.1 route-map RM_AS_PATH_PREPEND
end
clear ip bgp 1.1.1.1 soft in
```

## OSPFv3:
### Neccessary configurations
```
ipv6 unicast-routing
ipv6 cef
```
### Interface config
```
interface g0/1
 ipv6 add ..
 ipv6 add fe80::1 link-local
 ipv6 ospf 1 area 0
```
### Area config
```
ipv6 router ospf 1
 router-id 1.1.1.1
 area 2 stub (--> stub area megadása, ha használunk több areát)
```

## EIGRP:
### Default config
```
router eigrp x
 eigrp router-id 1.1.1.1 ???
 redistribute static
 no auto-summary
 ip summary-address eigrp 1 192.168.0.0 255.255.252.0 5
 redistribute dforg-prot metric 1000 0 1 255 255 
```

### Messeage authentication:
```
key chain CHAINNAME
 key x
  key-string password
  exit
 exit
```

```
interface g0/1 (link-where you want to authenticate EIGRP messages)
 ip authentication mode eigrp x md5
 ip authentication key-chain eigrp x CHAINNAME
```

### Show commands:
- sh ip eig nei
- sh ip eig top
- sh run | sec eigrp
- sh ip protocols
- sh ip route


# HSRP:
## Default configuration
```
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
```
ip sla x
icmp-echo dst-add source-interface link-to-dst
ip sla schedule x life forever start-time now
track y ip sla x reachability
```

### Set up HSRP to use SLA tracking
(You have a proper working HSRP using preemption and priorty setting)
```
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



# Radius for SSH:
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

