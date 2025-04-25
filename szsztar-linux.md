# Linux configurations

## Bind9

### named.conf.options

```conf
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

```conf
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

```sh
tsig keygen ddns > /etc/bind/ddns.key 
```

```conf
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

```conf
...

INTERFACES="ens33"

...
```

### /etc/dhcp/dhcpd.conf

> [!NOTE]
> You have plenty of examples. Pick one that fits most of your needs, end add these lines for DDNS:

```conf
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

### /etc/default/haproxy

```conf
ENABLED=1
```
### Load balancing modes

- roundrobin > select server in turns
- leastconn > select the servers with least connections
- source > selects server based on hash of the clients IP

### Layer 4 loadbalancing

`/etc/haproxy/haproxy.cfg`

```config
...

defaults
    mode tcp
    option tcplog

...

frontend www
    bind :::80
    deafult_backend wordpress-backend

backend wordpress-backend
    balance roundrobin
    mode tcp
    server wordpress-1  10.10.10.10:80 check
    server wordpress-2  10.10.10.20:80 check
```

### Layer 7 loadbalancing

`/etc/haproxy/haproxy.cfg`

```config
...

defaults
    mode http
    option httplog

...

frontend www
    bind :::80
    deafult_backend wordpress-backend

    option http-server-close
    acl url_wordpress path_beg -i /wordpress
    use_backend wordpress-backend if url_wordpress

    default_backend web-backend

backend wordpress-backend
    balance roundrobin
    http-request replace-path ^/type1(/)?(.*) /\2
    server wordpress-1 10.10.10.10:80 check
    server wordpress-2 10.10.10.20:80 check


backend wordpress-backend
    server web-1  10.10.10.30:80 check

listen stats
    bind :1936
    stats enable
    stats scope www
    stats scope web-backend
    stats scope wordpress-backend
    stats uri /
    stats realm Haproxy\ Statistics
    stats auth user:password
```

#### HTTPS setup

`/etc/haproxy/haproxy.cfg`

```config
...

defaults
    stats enable
    stats uri /stats
    stats realm Haproxy\ Statistics
    stats auth user:password

...

frontend www-http
   bind haproxy_www_public_IP:80
   default_backend www-backend

frontend www-https
   bind haproxy_www_public_IP:443 ssl crt /etc/ssl/private/example.com.pem
   default_backend www-backend

backend www-backend
   redirect scheme https if !{ ssl_fc }
   server www-1 www_1_private_IP:80 check
   server www-2 www_2_private_IP:80 check
```

## Apache2

> [!NOTE]
> Example config

```cfg
<VirtualHost *:80>
    ServerName web.company.com

    DocumentRoot /var/www/html
    DirectoryIndex index.html index.php index.html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks # Enable indexing for site
        AllowOverride None
        Require all granted
    </Directory>

    ErrorDocument 404 error.html # Error documents
    Alias /whoami /opt/wwwroot/whoami/main.html # Alias for anything
    
    # Logging
    ErrorLog /var/log/apache2/company/error.log
    CustomLog /var/log/apache2/company/access.log combined
    
    
    # Redirect to https
    Redirect permanent / https://web.company.com/
</VirtualHost>
```

## NFTables

> [!NOTE]
> How to filter?

- `ip saddr <IPv4 address / prefix>`
  - Example: `ip saddr {10.1.0.0/24, 10.2.0.0/24}`
- `ip daddr <IPv4 address / prefix>`
- `ip6 saddr <IPv6>`
- `ip6 daddr <IPv6>`
- `tcp sport <PORT>`
- `tcp dport <PORT>`
- `udp sport <PORT>`
- `udp dport <PORT>`
- `iif <INTERFACE_NAME>`
- `oif <INTERFACE_NAME>`

> [!NOTE]
> Example config

```bash
#!/usr/sbin/nft -f

flush ruleset

define INT4 = { 10.1.10.0/24 }

define INT_DMZ4 = { $INT4, 10.1.20.0/24 }
define INT_DMZ6 = { 2001:db8:1001:10::/64, 2001:db8:1001:20::/64 }

table inet filter {
    chain forward {
        type filter hook forward priority forward;
        # Apply default drop policy
        policy drop;
    
        # Accept return traffic
        ct state { established, related } accept;
    
        # Allow traffic from INT and DMZ to internet
        ip saddr $INT_DMZ4 oif ens18 accept;
        ip6 saddr $INT_DMZ6 oif ens18 accept;
    }
    
    chain srcnat {
        type nat hook postrouting priority srcnat;
    
        # Configure PAT for INT and DMZ networks
        ip saddr { 10.1.10.0/24, 10.1.20.0/24 } oif ens18 masquerade;
    }
  
    chain dstnat {
        type nat hook prerouting priority dstnat;
        
        # Create port forwarding rules for external HTTP(S) and DNS traffic
        ip daddr 1.1.1.10 tcp dport { 53,80,443 } dnat to 10.1.20.20;
        ip daddr 1.1.1.10 udp dport 53 dnat to 10.1.20.20;
    
        # Route INT and VPN networks to transparent HTTP proxy running on this host
        ip saddr { 10.1.10.0/24, 10.1.30.0/24 } tcp dport 80 redirect to 3128;
        ip6 saddr { 2001:db8:1001:10::/64, 2001:db8:1001:30::/64 } tcp dport 80 redirect to 3128;
    }
}
```

## PKI

### Root Certification Authority

> [!NOTE]
> Create a root CA to sign all your certificates.
> Create the CA:

```bash
mkdir /ca && cd /ca

# Generate CA key
openssl genrsa -aes256 -out CA.key 4096
# Generate CA certificate
openssl req -x509 -new -nodes \
  -key CA.key -sha256 \
  -days 365 \
  -out CA.crt \
  -subj '/C=HU/O=My Org/CN=My Org Root CA'
```

> [!NOTE]
> Add CA certificate to trusted CA certificates on a machine. This is needed for the machine to validate certificates created by this CA.

```sh
cp CA.crt /usr/local/share/ca-certificates/
update-ca-certificates
```

> The `update-ca-certificates` command only reads files with `.crt` extension.
{.is-warning}

### Subordinate Certification Authority

In order to make a sub CA, you need to generate a signing request and a private key.

```bash
openssl req -new -nodes \
  -out subca.csr \
  -newkey rsa:4096 \
  -keyout subca.key \
  -subj '/C=HU/O=My Org/CN=My Org SubCA'
```

Create a file for the v3 extensions.

<kbd>subca.v3.ext</kbd>

```bash
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints=CA:TRUE
```

Sign the certificate with the root CA certificate. Make sure to include the v3 extension file.

```bash
openssl x509 -req \
  -in subca.csr \
  -CA CA.crt \
  -CAkey CA.key \
  -CAcreateserial \
  -out subca.crt \
  -days 365 \
  -sha256 \
  -extfile subca.v3.ext
```

### Generate certificates

To generate a certificate, first create a signing request (CSR). This command will also create the key alongside the request.

```bash
openssl req -new -nodes \
  -out CERT.csr \
  -newkey rsa:4096 \
  -keyout CERT.key \
  -subj '/C=HU/O=My Org/CN=<FQDN>'
```

Create and edit the file `CERT.v3.ext`. This will be used to supply the x509v3 extensions for signing the certificate.

```bash
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage=digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName=@alt_names

# Add any alternative DNS names or IP addresses for the certificate
[alt_names]
DNS.1 = <Domain Address>
IP.1 = <IP Address>
```

> [!NOTE]
> Sign the certificate. This will output the certificate file which can be used with the key file.

```bash
openssl x509 -req \
  -in CERT.csr \
  -CA CA.crt \
  -CAkey CA.key \
  -CAcreateserial \
  -out CERT.crt \
  -days 365 \
  -sha256 \
  -extfile CERT.v3.ext
```

> [!NOTE]
> If needed, bundle a certificate and its key into a pkcs12 pack:

```bash
openssl pkcs12 -export \
  -inkey CERT.key \
  -in CERT.crt \
  -out CERT.p12
```

### Test a service running SSL/TLS

Connect to a service using openssl

```bash
openssl s_client -connect <host>:<port>
```
