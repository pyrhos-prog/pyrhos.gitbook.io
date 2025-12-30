# Dnsrecon

## DNSRECON

Su función principal es obtener información útil sobre la infraestructura de DNS de un objetivo, lo que puede ser valioso para identificar posibles vulnerabilidades o configuraciones incorrectas que puedan ser explotadas:

```
pyrhos@debian-laptop:~$ dnsrecon -d x.com
[*] std: Performing General Enumeration against: x.com...
[-] DNSSEC is not configured for x.com
[*]      SOA a.u10.twtrdns.net 204.74.111.101
[*]      NS c.u10.twtrdns.net 205.251.194.151
[*]      NS d.r10.twtrdns.net 205.251.199.195
[*]      NS d.u10.twtrdns.net 205.251.199.195
[*]      NS a.r10.twtrdns.net 205.251.192.179
[*]      NS a.u10.twtrdns.net 204.74.111.101
[*]      Bind Version for 204.74.111.101 Nameserver"
[*]      NS b.r10.twtrdns.net 205.251.196.198
[*]      NS b.u10.twtrdns.net 205.251.196.198
[*]      NS c.r10.twtrdns.net 205.251.194.151
[*]      MX alt3.aspmx.l.google.com 172.253.130.26
[*]      MX alt4.aspmx.l.google.com 142.250.4.27
[*]      MX alt1.aspmx.l.google.com 192.178.213.26
[*]      MX alt2.aspmx.l.google.com 142.250.147.27
[*]      MX aspmx.l.google.com 173.194.76.26
[*]      MX alt3.aspmx.l.google.com 2a00:1450:4010:c20::1b
[*]      MX alt4.aspmx.l.google.com 2404:6800:4003:c06::1a
[*]      MX alt1.aspmx.l.google.com 2a00:1450:4013:c1e::1b
[*]      MX alt2.aspmx.l.google.com 2a00:1450:4025:c01::1a
[*]      MX aspmx.l.google.com 2a00:1450:400c:c00::1a
[*]      A x.com 172.66.0.227
[*]      A x.com 162.159.140.229
[*]      TXT x.com atlassian-domain-verification=j6u0o1PTkobCXC84uEF/sWpIPtaZURBVYqKzmTvT8wugLcHT1vvrzzA63iP1qSLN
[*]      TXT x.com atlassian-sending-domain-verification=bd424180-8645-4de5-bd6a-285479c7577a
[*]      TXT x.com figma-domain-verification=ee8420edd01965ba297f3438c907cfc6fbbaa1ee90a07b28f28bcfca8e6017bb-1729630998
[*]      TXT x.com google-site-verification=8yQmoVhQedzlt36RPeQP41ytrEFk9aHEnde_xm0626g
[*]      TXT x.com google-site-verification=F6u9mGL--d2lbLljvH3b1UUgXtevQPdcamKr9c8914A
[*]      TXT x.com google-site-verification=lEZNYWieV7-UbDJafAm0u_RvNFb7GGqIYWAP4JmG5qs
[*]      TXT x.com google-site-verification=rbRGYlOADDbtUYJTGd8GEDm0PwPZExviDSaSH4JLR8Q
[*]      TXT x.com google-site-verification=reUF-TgZq93ZGtzImw42sfYglI2hY0QiGRmfc4jeKbs
[*]      TXT x.com kkdl3qb3tcrmdhfsm803p67r0my0svs8
[*]      TXT x.com shopify-verification-code=cUZazKrqCWgcshrcGvgfFR1lieuhRF
[*]      TXT x.com slack-domain-verification=Csk4bjCPFnJaDLLaKFUwCTFuUpCVvnYlAm2Tba0i
[*]      TXT x.com stripe-verification=46F7B88485621DC18923B43D12E90E6CDBCE232F2FEBCF084E6EFA91F6BA707D
[*]      TXT x.com v=spf1 ip4:199.16.156.0/22 ip4:199.59.148.0/22 include:_spf.google.com include:_spf.salesforce.com include:_oerp.x.com include:phx1.rp.oracleemaildelivery.com include:iad1.rp.oracleemaildelivery.com -all
[*]      TXT x.com 3089463
[*]      TXT x.com _w548xs1kfxtlqk3jyx19bzwk34c473i
[*]      TXT x.com adobe-idp-site-verification=ab4d9ce3473a73e81f46238da34ea4967fd5ac80e5c43fbfa8dff46d06a5321c
[*]      TXT x.com adobe-sign-verification=c693a744ee2d282a36a43e6e724c5ea
[*]      TXT x.com apple-domain-verification=sEij6tJOW11fVNrG
[*]      TXT _dmarc.x.com v=DMARC1; p=reject; rua=mailto:caf935f12c8645b2921b0749d1fcd49e@dmarc-reports.cloudflare.net
[*] Enumerating SRV Records
[-] No SRV Records Found for x.com

```
