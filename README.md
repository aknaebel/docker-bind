# docker-bind

## Description:

This docker image provide a [bind](https://www.isc.org/downloads/bind/) service based on [Alpine Linux](https://hub.docker.com/_/alpine/)

## Usage:
```
docker run --name bind -d -p 53:53/udp -p 53:53 \
-v /bindconfig/named.conf:/etc/bind/named.conf \
-v /bindconfig/zones:/etc/bind/zones \
--restart=always aknaebel/bind
```

## Docker-compose:
```
version: '2'
services:
    bind:
        image: aknaebel/bind
        ports:
            - "53:53/udp"
        volumes:
            - /bindconfig/named.conf:/etc/bind/named.conf
            - /bindconfig/zones:/etc/bind/zones
        restart: always
```

```
docker-compose up -d
```

## Bind stuff:

### Bind configuration:

Put your configuration in the ``named.conf`` file and mount it into the docker container with the option:
```
-v <bindconfig>/named.conf:/etc/bind/named/conf
```

### Minimal authoritative named.conf:
```
options {
        directory "/var/bind";

        // Configure the IPs to listen on here.
        listen-on { 127.0.0.1; };
        listen-on-v6 { none; };

        // If you want to allow only specific hosts to use the DNS server:
        //allow-query {
        //      127.0.0.1;
        //};

        // Specify a list of IPs/masks to allow zone transfers to here.
        //
        // You can override this on a per-zone basis by specifying this inside a zone
        // block.
        //
        // Warning: Removing this block will cause BIND to revert to its default
        //          behaviour of allowing zone transfers to any host (!).
        allow-transfer {
                none;
        };

        // If you have problems and are behind a firewall:
        //query-source address * port 53;

        pid-file "/var/run/named/named.pid";

        // Changing this is NOT RECOMMENDED; see the notes above and in
        // named.conf.recursive.
        allow-recursion { none; };
        recursion no;
};

// Example of how to configure a zone for which this server is the master:
//zone "example.com" IN {
//      type master;
//      file "/etc/bind/master/example.com";
//};

// You can include files:
//include "/etc/bind/example.conf";
```

### Minimal recursive named.conf:
```
options {
        directory "/var/bind";

        // Specify a list of CIDR masks which should be allowed to issue recursive
        // queries to the DNS server. Do NOT specify 0.0.0.0/0 here; see above.
        allow-recursion {
                127.0.0.1/32;
        };

        // If you want this resolver to itself resolve via means of another recursive
        // resolver, uncomment this block and specify the IP addresses of the desired
        // upstream resolvers.
        //forwarders {
        //      123.123.123.123;
        //      123.123.123.123;
        //};

        // By default the resolver will attempt to perform recursive resolution itself
        // if the forwarders are unavailable. If you want this resolver to fail outright
        // if the upstream resolvers are unavailable, uncomment this directive.
        //forward only;

        // Configure the IPs to listen on here.
        listen-on { 127.0.0.1; };
        listen-on-v6 { none; };

        // If you have problems and are behind a firewall:
        //query-source address * port 53;

        pid-file "/var/run/named/named.pid";

        // Removing this block will cause BIND to revert to its default behaviour
        // of allowing zone transfers to any host (!). There is no need to allow zone
        // transfers when operating as a recursive resolver.
        allow-transfer { none; };
};

// Briefly, a zone which has been declared delegation-only will be effectively
// limited to containing NS RRs for subdomains, but no actual data beyond its
// own apex (for example, its SOA RR and apex NS RRset). This can be used to
// filter out "wildcard" or "synthesized" data from NAT boxes or from
// authoritative name servers whose undelegated (in-zone) data is of no
// interest.
// See http://www.isc.org/products/BIND/delegation-only.html for more info

//zone "COM" { type delegation-only; };
//zone "NET" { type delegation-only; };

zone "." IN {
        type hint;
        file "named.ca";
};

zone "localhost" IN {
        type master;
        file "pri/localhost.zone";
        allow-update { none; };
        notify no;
};

zone "127.in-addr.arpa" IN {
        type master;
        file "pri/127.zone";
        allow-update { none; };
        notify no;
};
```

### Zones files config:

The main idea is to put the zones files in a ``zones`` subdir and mount it into the docker container with the docker option:
```
-v <bindconfig>/zones:/etc/bind/zones
```
