
```python
plt.figure(figsize=(8, 8))
plt.imshow(maze_arr, cmap="gray", interpolation="nearest")
if path:
    path_y, path_x = zip(*path)
    plt.plot(path_x, path_y, color="red", linewidth=2)
plt.xticks([])
plt.yticks([])
plt.show()

def bfs(start, end, barrier):
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    for i in range(len(maze)):
        for j in range(len(maze[i])):
            if(maze[i][j] == start):
                start = (i, j)
            if(maze[i][j] == end):
                end = (i, j)
    queue = deque()
    queue.append((start, [start]))
    visited = set()
    visited.add(start)
    while queue:
        position, path = queue.popleft()
        if position == end:
            return path
        elif len(path)==0x7fffffff:
            return path
        for d in directions:
            next_position = (position[0] + d[0], position[1] + d[1])
            if 0  2:
            maze[row][col] = flag[c]
            c += 1
            if c == len(flag):
                c = 0

end = (77, 1)      # 起点
start = (1, 83)    # 终点
path = bfs(start, end, "#")

# 遍历路径，获取路径上的字符
f = ""
for i in path:
    if maze[i[0]][i[1]] != " ":
        f += maze[i[0]][i[1]]
print(f[1:len(f)-1])

# SHCTF{81f6ad65-9da6-41ae-bd61-88dea61332f1}

# 画出迷宫路径
maze = [[0 if cell == "#" else 1 for cell in row] for row in maze]
draw_maze(maze, path)
```

![](https://attach.52pojie.cn/forum/202410/18/123946b09t9vaiv1w09hhe.png)
