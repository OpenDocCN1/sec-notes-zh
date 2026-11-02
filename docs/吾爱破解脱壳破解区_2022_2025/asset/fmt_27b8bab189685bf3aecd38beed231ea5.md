
```
def get_real_c(_index, enc_input):

    # 通过索引和enc_input得到原始input

    tmp = enc_input - 97 - key[_index] - _index

    return tmp

def treap_traverse(_root):

    if _root == 0:

        return

    node_c = get_real_c(treap_dict[_root][2] - 1, _root)

    idx = encs.index(_root)     # 得到后续遍历后的数组中_root的索引

    # 如果左节点不为空，更新左节点的数据

    left_root = treap_dict[_root][0]

    right_root = treap_dict[_root][1]

    if left_root != 0:

        left_xh = get_child_xuhao(node_c, gxs_left[idx])

        left_c = get_real_c(left_xh - 1, treap_dict[_root][0])      # 左孩子

        treap_dict[left_root][2] = left_xh

        treap_dict[left_root][3] = left_c

        treap_traverse(left_root)

    if right_root != 0:

        right_xh = get_child_xuhao(node_c, gxs_right[idx])

        right_c = get_real_c(right_xh - 1, treap_dict[_root][1])     # 右孩子

        treap_dict[right_root][2] = right_xh

        treap_dict[right_root][3] = right_c

        treap_traverse(right_root)

treap_traverse(223)

flag = []

for _key, value in treap_dict.items():

    flag.append([value[2], value[3]])

flag.sort()

flag = [x[1] for x in flag]

print("SUSCTF{" + "".join(map(chr, flag)) + "}")

# SUSCTF{8226d8a68d25d8f03be17c4d7027b29c}
```
`flag`为：SUSCTF{8226d8a68d25d8f03be17c4d7027b29c}
