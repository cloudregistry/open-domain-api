# Open Zone API or "Open Domain API"

An OAuth-based RESTful API that allows a third party client to:

1. modify name server records for a domain name at the registrar, or 
2. to modify DNS records at the DNS provider.


## Terminologies

### Name servers

These are the name servers responsible for answering DNS queries for a domain name. To modify the set of name servers, one typically goes to the domain registrar through which the domain name is registered.

### Domain

For the purposes of this API specification, a domain is a registration record held at a registrar (which is then provisioned in the authoritative registry of the domain's TLD.)

### Zone

For the purposes of this API specification, a zone is a collection of DNS resource records identified by a domain name. This is loosely equivalent to a zone file.

### Roles

There are two sides to the API -- the provider who implements and exposes the API, clients who consume the API. There are two distinct classes of providers defined in this specification: _registrars_ and _DNS providers_.


#### Clients

Clients are applications or services that wish to modify the name servers or DNS records of a particular domain name. The domain name is typically registered at a registrar, and its DNS could be hosted with the same registrar or yet a different party.

Examples of potential clients using this API are:
* Google Apps for Business - who may want to verify that the customer owns the domain, and provide an automated way of changing the MX records at the DNS provider so that mails are sent to the registrar.
* Tumblr could offer custom domains functionality by helping the user change the A records at the DNS provider.
* A DNS hosting provider could use the API to change the name servers of a domain to point to its own name servers. It could also use the API to provision DNSSEC DS records.


#### Registrars

Registrars are entities that provide domain name registration services. In many cases, the registrar is not the final nor authoritative database for the domain registration. In some cases, though, domains are registered directly at the registry. The concept of "registry" is not relevant to this API specification.


#### DNS Providers

A DNS provider is an entity that offers DNS hosting service. It maintains the zone database, and operates authoritative DNS servers to answer queries relating to the zone. These authoritative DNS servers are typically the NameServers provisioned at the registrar, although many configurations exist where this is not true.

It is worth noting that some DNS hosting services offer "secondary DNS servers" or "DNS slaves". These services hold secondary copies of the zones that are replicated from a master database. As such, these services are, by the definition of this specification, _not_ DNS Providers. They should not implement this API for the secondary zones that are configured in this manner.


## API

### Base

* /info - [open] queries the supported APIs and associated versions, and returns the OAuth endpoint URL
* /

### Registrar

### Zone

* endpoints for adding rr to an rrset
* an endpoint to do nsupdate-style atomic updates

## Recommendations for Clients

### General

The following recommendations apply to all cases:

1. before replacing any records, tell the user what you are going to do and seek confirmation from the user.
2. after replacing the records, provide a summary of the changes made, including the old records that were deleted.


### To update the NameServers of a Domain

1. Perform a WHOIS query to find out the registrar of record
2. Determine the ODA API endpoint of the registrar, using the bootstrap service or other means
3. Query the registrar ODA API endpoint to find out if it supports the API
4. Redirect the registrar 


api to ask the registrar if it has the authoritative zone data. It could be that the DNS 
func to list the supported API bundle


### Recommendations for Registrars

1. Should respond with an error if the domain has expired (but is perhaps yet to be purged from the registrar's database)
2. Should respond with a warning if the domain is inactive (e.g. if the registry has place a hold on it so any changes to the DNS will not be reflected)
3. Should offer two single-use OAuth keys by default.
4. May offer long
5. Should log all changes
5. Should email the 


a bootstrap service that tries to map the nameserver names to a known API endpoint, and also the registrar name to a known API


# Issues

1. Should registrars require clients to register themselves first, and provide base API only to 