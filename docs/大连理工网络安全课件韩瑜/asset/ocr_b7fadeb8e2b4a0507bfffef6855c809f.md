# # AH两种模式示意图IPv6

IPv6
orig IP
hdr
extension headers
(if present)
TCP
Data

authenticated except for mutable fields

IPv6
orig IP
hdr
hop-by-hop, dest,
routing, fragment
AH
dest
TCP
Data

(b) Transport Mode

authenticated except for mutable fields in
new IP header and its extension headers

IPv6
new IP
hdr
ext
headers
AH
orig IP
hdr
ext
headers
TCP
Data

(c) Tunnel Mode