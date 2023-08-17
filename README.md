# PHAC Alpha DNS

This repo contains config files for PHAC DNS services. It is built on top of [CDS's dns repo](https://github.com/cds-snc/dns) but with an IaD (Infrastructure as Data) approach.

Currently, we have three sub-domains on top of the CDS's _alpha.canada.ca_ domain:

- _\*.phac-aspc.alpha.canada.ca_
- _\*.phac.alpha.canada.ca_
- _\*.aspc.alpha.canada.ca_

> Note: See https://github.com/cds-snc/dns/blob/main/terraform/phac-aspc.alpha.canada.ca.tf for reference.

## Request a DNS

To request a DNS, you'll need to [create a Managed Zone](https://cloud.google.com/dns/docs/zones#create_managed_zones) resource in your GCP project.

The `DNS name` field for the zone should have one of the previously mentioned subdomains as a prefix, the general convention is `<zone-name>.<sub-domain-name>.`. For instance, if my `Zone name` is `example`, then the `DNS name` could be `example.phac-aspc.alpha.canada.ca.`

> Note: The period (`.`) at the end is required to make it a FQDN (Fully Qualified Domain Name). If you're curious, _why?_ - read [this](https://jvns.ca/blog/2022/09/12/why-do-domain-names-end-with-a-dot-/).

Once done, click `Registrar Setup` in the top right corner of the `Zone details` page for your newly created zone and note down list of NS (Name Servers).

Now, submit a PR into the repo with the following template in the `dns-records` directory.

```yaml
apiVersion: dns.cnrm.cloud.google.com/v1beta1
kind: DNSRecordSet
metadata:
  name: <zone-name>
  namespace: alpha-dns
  annotations:
    ownerName: <Your Name>
    ownerContact: <Your.Email@example.com>
    # If there isn't an associated GitHub repository, please comment out the next line 
    gitHubRepo: <GitHub-repository>
spec:
  name: "<DNS-name>"
  type: "NS"
  ttl: <your-desired-value>
  managedZoneRef:
    external: <zone-reference-name>
  rrdatas:
    - "<name-server-1>"
    .
    .
    - "<name-server-N>"
```

In the above template, fill out the values for placeholders(`<>`):

- `<zone-name>`: Name of the resource, could be same as the Zone name that you've created.
- `<DNS-name>`: The DNS name from the previously created resource in your project. Don't forget the `.` at the end.
- `<your-desired-value>`: Value to set for ttl (Time to Live). A good default for this is 300 but feel free to modify it. Units are in seconds.
- `<zone-reference-name>`: This should be one of the three sub-domains we have. That is, one of `phac-aspc-alpha-canada-ca`, `phac-alpha-canada-ca` or `aspc-alpha-canada-ca`.
- `<name-server1>...<name-server-N>`: Paste the noted NS values from the previous step here.

After the PR is reviewed and merged, the config connector will provision / link the resources.

Once done, you can add other types of DNS record sets to your zone with the registered DNS name.
