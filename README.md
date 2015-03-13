# Open Domain API

## Introduction

An OAuth-based RESTful API that allows a domain registrant to authorize a third party client to connect directly to the registry or registrar to make certain modifications to the domain:

1. modify name server records for a domain name at the registrar
2. modify glue records
3. modify DS records


## Terminologies

### Name servers

These are the name servers responsible for answering DNS queries for a domain name. To modify the set of name servers, one typically goes to the domain registrar through which the domain name is registered.

### Domain

For the purposes of this API specification, a domain is a registration record held at a registrar (which is then provisioned in the authoritative registry of the domain's TLD.)

### Roles

There are two sides to the API -- the provider who implements and exposes the API, clients who consume the API. There are two distinct classes of providers defined in this specification: _registrars_ and _DNS providers_.


#### Clients

Clients are DNS operators that wish to modify the DNS delegation records associated with a domain name. Use cases are:

- DNS operator's onboarding process involves telling the user to update the name servers authoritative for the domain at the registrar. This can automate the process.
- DNS operator may need to relocate a name server for various reasons (e.g. migration, rebalancing of load)
- DNS operator can manage the key lifecycle for a safe DNSSEC key rollover by managing the DS records that the registry publishes at the parent zone.


#### Resource Server

For the purpose of this specification, the OAuth resource server role is fulfilled by an entity who will make the modifications, possibly through intermediaries, to the domain.

In the most general sense, for a given domain, it could be:
* the registry
* the registrar of record
* the reseller or a sub-reseller ...


#### DNS Operators

A DNS operator is an entity that offers DNS hosting service. It maintains the zone database, and operates authoritative DNS servers to answer queries relating to the zone. These authoritative DNS servers are typically the NameServers provisioned at the registrar, although many configurations exist where this is not true.


## API

### Base endpoint

Each resource server should publish a base URL e.g. `https://example.org/api/`

Performing a GET request on the API should yield the following information:
* `registration` endpoint - this should be an endpoint that allows Clients to identify itself with the resource server.
* `authorization` endpoint - the OAuth authorization endpoint
* `domain-api-authority` endpoint - this allows the resource server to delegate the APIs to another server (perhaps a reseller). This endpoint does not require the resource owner to grant resource authorization as it is read-only. It only requires [client credentials](http://tools.ietf.org/html/rfc6749#section-4.4).
* API endpoints - to operate on the resources (domain NS, glue and DS records)
   * `domain` endpoint - a REST-based API that allows one to manage the NS and DS records.
   * `host` endpoint - a REST-based API that allows one to manage the glue records

An example response could look like:

```javascript
{
  "endpoints": {
    "registration": "https://example.org/api/register",
    "authorization": "https://example.org/api/authorize",
    "domain-api-authority": "https://example.org/api/domain/{domain-name}",
    "domain": "https://example.org/api/domain/{domain-name}",
    "host": "https://example.org/api/host/{host-name}/"
  }
}
```

Some of the endpoints are URI templates.



## Discovery

In order to determine the authoritative resource server for a given domain, the client must peform a few steps by querying several parties.

In general, for a given domain name _foo.com_:

1. find `SRV` record for `_oda.com` to obtain the base endpoint offered by the com registry
2. query the com resource server's `domain-api-authority` endpoint to see who is responsible for _foo.com_. It delegates to the registrar's resource server.
3. query the registrar resource server's `domain-api-authority` endpoint to find if it is responsible for _foo.com_. It further delegates it to the reseller's resource server.
4. query the reseller resource server's `domain-api-authority` endpoint to find if it is responsible for _foo.com_. It answers affirmatively.

Note that steps 2 to 4 are essentially generic, and the client does not need to distinguish between a registry, registrar or reseller. There could also be further steps, so the client needs some loop detection and a limit to how many hops it will follow.

Should step 1 above fail, the discovery flow could allow for a "look-aside mapping service":

1. Use the mapping service to find out of there is an base endpoint entry for the TLD. If so, proceed with step 2 above.
2. If there isn't, perform a WHOIS query to find out the registrar of record. In case of ICANN-accredited registrar, extract the Registrar ID.
3. Use the mapping service to find out if the Registrar ID has a base endpoint mapping. If so, continue with step 3 above.


## Registration

Due to the distributed nature of the domain registration ecosystem, as well as the potentially large number of resource servers, it is important to allow DNS operators to automatically register itself with a resource server (probably on first use.)

This is established by having DNS operators calling the registration endpoint of the resource server. The call may be signed using the DNS operators key, which allows an automatic registration of whitelisted operators (at the discretion of the resource server.)

DNS operators may publish their key in a well-known location, which is out of scope for this specification.


## Recommendations for Clients

### General

The following recommendations apply to all cases:

1. before replacing any records, tell the user what you are going to do and seek confirmation from the user.
2. after replacing the records, provide a summary of the changes made, including the old records that were deleted.


### To update the NameServers of a Domain

If the client has no existing authorization information for the given domain, it should:

1. performs discovery to find the resource server for the given domain
2. registers itself with the resource server
3. request OAuth authorization by redirecting the resource owner to the authorization endpoint
4. Once authorization is complete, obtain an access token
5. call API endpoint to manage name servers.



## Recommendations for Registrars

1. Should respond with an error if the domain has expired (but is perhaps yet to be purged from the registrar's database)
2. Should respond with a warning if the domain is inactive (e.g. if the registry has place a hold on it so any changes to the DNS will not be reflected)
3. Should offer single-use OAuth keys by default.
4. May offer long term keys
5. Should log all changes
5. Should email the 


a bootstrap service that tries to map the nameserver names to a known API endpoint, and also the registrar name to a known API

