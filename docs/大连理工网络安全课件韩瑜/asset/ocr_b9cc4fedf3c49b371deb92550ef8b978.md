# 改进后的假想的对话：

用户登录的每次对话：
(1) C → AS：ID_C || ID_TGS
(2) AS → C：E_{K_C}[Ticket_{tgs}]

每种服务类型一次：
(3) C → TGS：ID_C || ID_V || Ticket_{tgs}
(4) TGS → C：Ticket_V

每种服务会话一次：
(5) C → V：ID_C || Ticket_V

Ticket_{tgs} = E_{K_{tgs}}[ID_C||AD_C||ID_{tgs}||TS_1||Lifetime_1]
Ticket_V = E_{K_V}[ID_C||AD_C||ID_V||TS_2||Lifetime_2]

![](bbox=[0.65, 0.07, 0.97, 0.53])