# # Euclid 算法

# ## 又叫 “辗转相除法”

简单的描述就是，记gcd(a,b)表示非负整数a,b的最大公因数，那么： gcd(a,b) =max[k,其中k|a,且k|b] = gcd(b,a%b) ,假设a>b