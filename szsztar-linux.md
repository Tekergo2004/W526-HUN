# Linux configurations

## Bind9

### named.conf.options

```dns
    ...

    # Where to forward the queries (hierarchy). Both IPv4, IPv6 addresses can be used.
    forwarders {
        1.1.1.1
    };

    # Don't know
    dnssec-validation auto/no;
    
    # When recursion needed (don't ask what it is)
    recursion yes;
    allow-recursion { any; };
    # 

    # From which subnet can the DNS clients query this server
    allow-query { any; };

    # How does the server listen
    listen-on port 53 { any; };
    listen-on-v6 { any; };

    ...
```

### named.conf.local (zone configurations)

```dns
acl dmz-servers {
    10.10.10.10;
    2001:db8:1010:10::10;
};

# This is how master/primary zones/
# Forward zone
zone shanghai.net IN {
    type master;
    file "/var/bind/db.shanghai.net";

    # Configurations to transfer zones to a secondary/slave server(s)
    allow-transfer {
        dmz-servers;
    };

    also-notify {
        dmz-servers;
    };
}

# IPv4 Reverse zone
zone 10.10.10.in-addr.arpa IN {
    type master;
    file "/var/lib/bind/db.10.10.10";
};

# IPv6 reverse zone
zone 0.1.0.0.0.1.0.1.8.b.d.0.1.0.0.2.ip6.arpa IN {
    type master;
    file "/var/lib/bind/db.db8.1010";
};

# This is how a slave/secondary zone looks like.
zone lego.dk IN {
    type slave;
    file "/var/lib/bind/db.lego.dk";
    
    # Transfer from master/primary zone
    masters {
        10.20.20.1;
        2001:db8:1020:20::1;
    };
}

## Zone for DDNS
zone ddns.net IN {
    type master;
    file "/var/lib/bind/db.ddns.net";
    
    # Below you can read how to create a key like this and how to include it into a file.
    allow-update { key ddns; }; 
};
```

> [!NOTE]
> With Bind9 installation you get a tool named `tsig-keygen` you can use it to generate sha-1? keys.

```bash
tsig keygen ddns > /etc/bind/ddns.key 
```

```dns
include "/etc/bind/ddns.key"

zone x.z IN {
    ...

    allow-update {
        key ddns;
    };

    ...
}
```

### Forward lookup zone

> [!NOTE]
> Copy `db.empty` to `db.your.domain` and edit it like the following. Replace `localhost` with `your.domain`.

```dns
name    IN  TYPE    IP-ADDRESS
```

#### Types

- NS
- A
- AAAA
- CNAME
- MX
- SRV

### Reverse lookup zone

> [!NOTE]
> Copy `db.empty` to `db.your.domain` and edit it like the following. Replace `localhost` with `your.domain`.

```dns
remaining-reverse-IP    IN  PTR    FQDN
```

## DDNS (ISC-DHCP-SERVER)

> [!NOTE]
> I used `isc-dhcp-server` package for this configuration. Get the key what have been generated above in DNS config.

### /etc/deafult/isc-dhcp-server

> [!NOTE]
> Set your interface

```dhcp
INTERFACES="ens33"
```

### /etc/dhcp/dhcpd.conf

> [!NOTE]
> You have plenty of examples. Pick one that fits most of your needs, end add these lines for DDNS:

```dhcp
include "/etc/dhcp/ddns.key";

ddns-updates on;
ddns-update-style standard;

update-static-leases on;

subnet 10.10.10.0 netmask 255.255.255.0 {
    range 10.10.10.10 10.10.10.250;
    option routers 10.10.10.254;
    ddns-domainname "shanghai.net";
    zone shanghai.net. {
        primary 10.10.10.1; # DNS server ip address
        key ddns; # The name from ddns.key
    }
}
```

## HAproxy

## PKI

## Apache2
