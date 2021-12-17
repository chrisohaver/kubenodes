# kubenodes

## Name

*kubenodes* - creates records for Kubernetes nodes.

## Description

*kubenodes* watches the Kubernetes API and synthesizes A, AAAA, and PTR records for Node addresses.
This plugin requires list/watch permission to the Nodes API.

This plugin can only be used once per Server Block.

## Syntax

```
kubenodes [ZONES...] {
    external
    endpoint URL
    tls CERT KEY CACERT
    kubeconfig KUBECONFIG [CONTEXT]
    ttl TTL
    fallthrough [ZONES...]
}
```
* `external` will build records using Nodes' external addresses.  If omitted, *kubenodes* will build records using
  Nodes' internal addresses.
* `endpoint` specifies the **URL** for a remote k8s API endpoint.
  If omitted, it will connect to k8s in-cluster using the cluster service account.
* `tls` **CERT** **KEY** **CACERT** are the TLS cert, key and the CA cert file names for remote k8s connection.
  This option is ignored if connecting in-cluster (i.e. endpoint is not specified).
* `kubeconfig` **KUBECONFIG [CONTEXT]** authenticates the connection to a remote k8s cluster using a kubeconfig file.
  **[CONTEXT]** is optional, if not set, then the current context specified in kubeconfig will be used.
  It supports TLS, username and password, or token-based authentication.
  This option is ignored if connecting in-cluster (i.e., the endpoint is not specified).
* `ttl` allows you to set a custom TTL for responses. The default is 5 seconds.  The minimum TTL allowed is
  0 seconds, and the maximum is capped at 3600 seconds. Setting TTL to 0 will prevent records from being cached.
  All endpoint queries and headless service queries will result in an NXDOMAIN.
* `fallthrough` **[ZONES...]** If a query for a record in the zones for which the plugin is authoritative
  results in NXDOMAIN, normally that is what the response will be. However, if you specify this option,
  the query will instead be passed on down the plugin chain, which can include another plugin to handle
  the query. If **[ZONES...]** is omitted, then fallthrough happens for all zones for which the plugin
  is authoritative. If specific zones are listed (for example `in-addr.arpa` and `ip6.arpa`), then only
  queries for those zones will be subject to fallthrough.

## External Plugin

To use this plugin, compile CoreDNS with this plugin added to the `plugin.cfg`.  It should be positioned before
the _kubernetes_ plugin if _kubenode_ is using the same zone or a superzone of _kubernetes_.

## Ready

This plugin reports that it is ready to the _ready_ plugin once it has synced to the Kubernetes API.

## Examples

Use Nodes' internal addresses to answer forward and reverse lookups in the zone `node.cluster.local.`.
Fallthrough to the next plugin for reverse lookups that don't match any Nodes' internal IP addresses.

```
kubenodes node.cluster.local in-addr.arpa ip6.arpa {
  fallthrough in-addr.arpa ip6.arpa
}
```

Use Nodes' external addresses to answer forward and reverse lookups in the zone `example.`. Fallthrough
to the next plugin for reverse lookups that don't match any Nodes' external IP addresses.

```
kubenodes example in-addr.arpa ip6.arpa {
  external
  fallthrough in-addr.arpa ip6.arpa
}
```