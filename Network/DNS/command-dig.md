# Domain Information Groper (DIG)
`dig` is a flexible tool for interrogating DNS name servers. It performs DNS lookups and displays the answers that are returned from the name server(s) that were queried.

## Usage
```
dig @server name type
```
Where:
* **server** is the hostname or IP address of the name server to query.
* **name** is the hostname of the record to look for.
* **type** is the resource record to query.

Unless a name server is specified, dig will try each of the servers listed in /etc/resolv.conf. For the most accurate data, it's wise to specify the authoritative name server of a domain otherwise cached data is typically returned.

## Querying Simple DNS Types

| DNS Type | Description | Example usage |
|---------:|------------:|--------------:|
| A | IPv4 Address | dig 1and1.com a |
| AAAA | IPv6 Address | dig 1and1.com aaaa |
| MX | Mail Exchanger | dig 1and1.com mx |
| TXT | Text/SPF Record | dig 1and1.com txt |
| CNAME | Canonical Name | dig 1and1.com cname |
| SOA | Start Of Authority | dig 1and1.com soa |

## Querying Advanced DNS Types
### Reverse DNS or Pointer records for IP addresses
While `A` records translate names into IP addresses, IP addresses can be translated into names through `PTR` or `rDNS` records. Somewhat unlike an almost endless number of domains that can be registered, the assigning of finite IP addresses are managed through the organization IANA and the `.arpa` DNS zone. To query a `PTR` record, we first have to reverse the octets in an IP address and append `.in-addr.arpa.` to the result. For example, `74.208.255.134` would be converted into `134.255.208.74.in-addr.arpa.` and can be queried using the `dig ptr 134.255.208.74.in-addr.arpa.` command. However, rather than manually reversing the IP address, the `-x` option can be used to automatically flip and append to the result.
| DNS Type | Description | Example usage |
|---------:|------------:|--------------:|
| PTR | Pointer/rDNS Record | dig -x 74.208.255.134  |

### Service Records
Service records are used to publish connection information for specific services of a domain. Unlike other types of DNS records, more than just a hostname is required when querying `SRV` records. Both the service name and protocol must be known before querying for an `SRV` record and must both start with an underscore.
```
dig _service._proto.domain srv
```
Where:
* **service** is the symbolic name of a service.
* **proto** is the transport protocol of a service.
* **domain** is the zone hostname of the Service to look for.

#### Example:
```
dig _sip._tls.1and1.com srv
```

## Additional dig options
### Trace option (+trace)
To trace through the DNS hierarchy step-by-step the `+trace` option can be appended to any dig command.

#### Example:
```
dig www.1and1.com a +trace
```
