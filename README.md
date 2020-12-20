This is an Ansible playbook to maintain TXT records on CloudFlare for SPF on G-Suite. You have to supply it with your CloudFlare API token as well as the CloudFlare zone (example.com) to act on. You then give the records you want to add by putting them in list format under the 'spf' variable. Each list element must have 'record', which is the name of the entry to add (record of 'test' would make the TXT entry for test.zone). Each list item must then have a value(s) for 'ipv4' and/or 'ipv6' which can be either a DNS entry to resolve to the A (IPv4) or AAAA (IPv6) record, or an IP address.

---
### Why?
Servers that I run can send internal emails to myself if I have their IP added to the SPF record of the domain they're trying to send as. This allows me to have a script to automatically update the IP's of those TXT records if they have changed since my servers run a dynamic DNS that would update their A/AAAA record when they notice a Public IP change.


## Usage
- Fill 'vars.yml' in this style:
```yml
  # Generate the API Token at this address
  cf_api_token: 'https://dash.cloudflare.com/profile/api-tokens'
  # CloudFlare zone to create the records at
  cf_zone: 'example.com'
  # List of the SPF records and the IP's/Domain's to resolve and add to the record.
  # (Can contain IPv4 and/or IPv6 entries)
  spf:
    # Create the SPF for a.example.com with the IPv4 address of 'test.com' and '192.168.0.1'
    - record: a
      ipv4:
        - 'test.com'
        - '192.168.0.1'
    # Create the SPF for foo.bar.example.com with the IPv4 address of 'test.com' and '192.168.0.1'
    # and the IPv6 address of AAAA.to.resolve.com as well as 'aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa'
    - record: foo.bar
      ipv4:
        - 'test.org'
        - '192.168.0.1'
      ipv6:
        - 'AAAA.to.resolve'
        - 'bbbb:bbbb:bbbb:bbbb:bbbb:bbbb:bbbb:bbbb'
```
- Run the playbook
```bash
ansible-playbook cloudflare-dns-update-spf.yml
```

This would create the following TXT records:  
a.example.com:  
```v=spf1 ipv4:1.1.1.1 ipv4:192.168.0.1 include:_spf.google.com ~all``` (assuming test.com resolves to 1.1.1.1)  
foo.bar.example.com:  
```v=spf1 ipv4:8.8.8.8 ipv4:192.168.0.1 ipv6:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa ipv6:bbbb:bbbb:bbbb:bbbb:bbbb:bbbb:bbbb:bbbb include:_spf.google.com ~all``` (assuming test.org resolves to 8.8.8.8 and AAAA.to.resolve resolves to 'aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa:aaaa')
