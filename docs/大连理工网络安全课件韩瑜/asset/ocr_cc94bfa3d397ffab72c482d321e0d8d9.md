# 安全认证

$ID_C \| E_{P_C} \| ID_C \| ID_V$

+   1. $C \rightarrow AS$: $ID_C \|\| P_C \|\| ID_V$
+   2. $AS \rightarrow C$: $Ticket$
+   3. $C \rightarrow V$: $ID_C \|\| Ticket$

$Ticket = E_{K_V}[ID_C \|\| AD_C \|\| ID_V]$