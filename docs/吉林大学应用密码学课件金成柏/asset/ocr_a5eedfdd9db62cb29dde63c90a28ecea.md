# 素性测试——Miller Rabin Algorithm

算法

+   TEST (n) is:
+   1. 找出整数 k 和 q，其中 k > 0，q 是奇数，使 (n-1)=2^k q
+   2. 随机选取整数 a，1<a<n-1
+   3. if a^q mod n = 1 then return ("可能是素数");
+   4. for j = 0 to k − 1 do
+   5. if (a^(2^j q) mod n = n-1) then return("可能是素数")
+   6. return ("合数")