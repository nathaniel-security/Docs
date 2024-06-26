# SSL Attacks

* SSL Server Test
  * https://www.ssllabs.com/ssltest/

### Searching CT Logs

There are two popular options for searching CT logs:

| Tool                                | Key Features                                                                                                     | Use Cases                                                                                                 | Pros                                              | Cons                                         |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | -------------------------------------------- |
| [crt.sh](https://crt.sh/)           | User-friendly web interface, simple search by domain, displays certificate details, SAN entries.                 | Quick and easy searches, identifying subdomains, checking certificate issuance history.                   | Free, easy to use, no registration required.      | Limited filtering and analysis options.      |
| [Censys](https://search.censys.io/) | Powerful search engine for internet-connected devices, advanced filtering by domain, IP, certificate attributes. | In-depth analysis of certificates, identifying misconfigurations, finding related certificates and hosts. | Extensive data and filtering options, API access. | Requires registration (free tier available). |

#### Pull data from SSL

* data about which sites are protected can be in the SSL certificate

```shell-session
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

**Filter by subdomain**

```shell-session
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

**Get ip address for above subdomains**

```
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u >> subdomainlist
```

```shell-session
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

* can run shodan on the IP's
