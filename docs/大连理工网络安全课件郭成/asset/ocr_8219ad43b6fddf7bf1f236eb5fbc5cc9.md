# # Table 14.4 Kerberos Version 5 Flags

| INITIAL | This ticket was issued using the AS protocol and not issued based on a ticket-granting ticket. |
|---|---|
| PRE-AUTHENT | During initial authentication, the client was authenticated by the KDC before a ticket was issued. |
| HW-AUTHENT | The protocol employed for initial authentication required the use of hardware expected to be possessed solely by the named client. |
| RENEWABLE | Tells TGS that this ticket can be used to obtain a replacement ticket that expires at a later date. |
| MAY-POSTDATE | Tells TGS that a postdated ticket may be issued based on this ticket-granting ticket. |
| POSTDATED | Indicates that this ticket has been postdated; the end server can check the authtime field to see when the original authentication occurred. |
| INVALID | This ticket is invalid and must be validated by the KDC before use. |
| PROXIABLE | Tells TGS that a new service-granting ticket with a different network address may be issued based on the presented ticket. |
| PROXY | Indicates that this ticket is a proxy. |
| FORWARDABLE | Tells TGS that a new ticket-granting ticket with a different network address may be issued based on this ticket-granting ticket. |
| FORWARDED | Indicates that this ticket has either been forwarded or was issued based on authentication involving a forwarded ticket-granting ticket. |