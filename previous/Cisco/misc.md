```bash
    # https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipv6_fhsec/configuration/15-sy/ip6-nd-inspect.html

    ipv6 nd inspection policy P1
        drop-unsecure
        sec-level minimum 2
        device-role {}
        trusted-port {}

    int g0/1 
        ipv6 nd inspection 
        ipv6 nd inspection attach-policy {}

    # https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipv6_fhsec/configuration/15-s/ip6f-15-s-book/ip6-ra-guard.html

    ipv6 ra guard policy {}
        device-role host 
        
        device-role router
        managed-config-flag on
        other-config-flag on
        match ipv6 access-list list1
        trusted-port 


    int g0/1 
        ipv6 nd raguard attach-policy {}

    #

    ipv6 snooping policy policy1
        ipv6 snooping attach-policy policy1
        security-level glean

    int g0/1 
        ipv6 snooping attach-policy {}

    # 
    ipv6 source-guard policy {}
        permit link-local
        deny global-autoconfig
        trusted

    int g0/1
        ipv6 source-guard attach-policy my_source_guard_policy
    # DHCPv6 guard

    ipv6 dhcp guard policy pol1
        device-role server
        match server access-list acl1
        trusted-port
```
https://community.cisco.com/t5/networking-documents/ipv6-nat64-dynamic-overload-mapping-pat-configuration-example/ta-p/3137971
https://community.cisco.com/t5/networking-documents/ipv6-stateful-nat64-configuration-example/ta-p/3124475
https://community.cisco.com/t5/networking-documents/ipv6-6to4-tunneling-configuration-example/ta-p/3139227

```sh
# RIP

	route-map {route-map name}
		set interface {if}
		match ip address {acl-name}
		
	router rip
	
        offset-list {id} out {offset}

        default-information originate route-map {route-map name}

        distribute-list {acl-name} {in/out} # set filter

        distance {ad} {source-ip} {wildcard mask} (ACL-name)
        
# ipv6 FHS
	
	ipv6 snooping policy {name}
		security-level {glean/guard/inspect}
		
	int g0/1
		ipv6 snooping attach-policy {name}
	
	# dhcp guard
	
	ipv6 dhcp guard policy {name}
		device-role {server/client}
		
	int g0/1
		ipv6 dhcp guard attach-policy {name}
		
	show ipv6 dhcp guard policy 
```

