# 表 14.1　Kerberos 版本 4 的消息交换

# (a) 服务认证交换：获得票据授权票据

(1) C → AS: `ID_C || ID_tgs || TS_1`

(2) AS → C: `E_{K_c}[K_{c,tgs} || ID_{tgs} || TS_2 || Lifetime_2 || Ticket_{tgs}]`

`Ticket_{tgs} = E_{K_{tgs}}[K_{c,tgs} || ID_C || AD_C || ID_{tgs} || TS_2 || Lifetime_2]`

# (b) 票据授权票据交换：获取服务授权票据

(3) C → TGS: `ID_v || Ticket_{tgs} || Authenticator_c`

(4) TGS → C: `E_{K_{c,tgs}}[K_{c,v} || ID_v || TS_4 || Ticket_v]`

`Ticket_{tgs} = E_{K_{tgs}}[K_{c,tgs} || ID_C || AD_C || ID_{tgs} || TS_2 || Lifetime_2]`

`Ticket_v = E_{K_v}[K_{c,v} || ID_C || AD_C || ID_v || TS_4 || Lifetime_4]`

`Authenticator_c = E_{K_{c,tgs}}[ID_C || AD_C || TS_3]`

# (c) 客户/服务确认交换：获取服务

(5) C → V: `Ticket_v || Authenticator_c`

(6) V → C: `E_{K_{c,v}}[TS_5 + 1]`　（相互认证）

`Ticket_v = E_{K_v}[K_{c,v} || ID_C || AD_C || ID_v || TS_4 || Lifetime_4]`

`Authenticator_c = E_{K_{c,v}}[ID_C || AD_C || TS_5]`