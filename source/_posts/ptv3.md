❯ find data/scannet/train -maxdepth 1 -mindepth 1 -type d | head
data/scannet/train/scene0224_00
data/scannet/train/scene0613_00
data/scannet/train/scene0047_00
data/scannet/train/scene0570_00
data/scannet/train/scene0470_01
data/scannet/train/scene0600_02
data/scannet/train/scene0537_00
data/scannet/train/scene0501_02
data/scannet/train/scene0472_01
data/scannet/train/scene0150_01
❯ ls -lah data/scannet/train/scene0224_00
总用量 3.8M
drwxrwxr-x    2 root01 root01 4.0K 5月  10  2024 .
drwxrwxr-x 1203 root01 root01  36K 5月  14  2024 ..
-rwxrwxr-x    1 root01 root01 223K 5月  10  2024 color.npy
-rwxrwxr-x    1 root01 root01 891K 5月  10  2024 coord.npy
-rwxrwxr-x    1 root01 root01 594K 5月  10  2024 instance.npy
-rwxrwxr-x    1 root01 root01 891K 5月  10  2024 normal.npy
-rwxrwxr-x    1 root01 root01 594K 5月  10  2024 segment200.npy
-rwxrwxr-x    1 root01 root01 594K 5月  10  2024 segment20.npy
❯ python - <<'PY'
import os
import numpy as np

root = "data/scannet/train/scene0224_00"

for name in sorted(os.listdir(root)):
    if not name.endswith(".npy"):
        continue

    path = os.path.join(root, name)
    x = np.load(path)
    
    print("=" * 70)
    print("file :", name)
    print("shape:", x.shape)
    print("dtype:", x.dtype)
    
    if x.ndim == 1:
        print("first 10:", x[:10])
        print("min/max :", x.min(), x.max())
        print("unique sample:", np.unique(x)[:30])
    else:
        print("first 5 rows:")
        print(x[:5])
        print("min:", x.min(axis=0))
        print("max:", x.max(axis=0))
PY
======================================================================
file : color.npy
shape: (75941, 3)
dtype: uint8
first 5 rows:
[[ 6  7  6]
 [17 18 16]
 [22 21 16]
 [66 56 47]
 [62 55 43]]
min: [2 2 0]
max: [255 255 255]
======================================================================
file : coord.npy
shape: (75941, 3)
dtype: float32
first 5 rows:
[[0.1983308  0.94450593 0.14272517]
 [0.1845233  0.9292399  0.12974097]
 [0.48927182 0.9495544  0.05585653]
 [0.26112506 1.4065274  0.34075788]
 [0.2643808  1.4222246  0.35346526]]
min: [ 0.03573645  0.09501085 -0.00215898]
max: [2.9687948 4.4702406 2.7748787]
======================================================================
file : instance.npy
shape: (75941,)
dtype: int64
first 10: [5 5 2 4 4 2 4 5 5 4]
min/max : -1 14
unique sample: [-1  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14]
======================================================================
file : normal.npy
shape: (75941, 3)
dtype: float32
first 5 rows:
[[ 0.4370696  -0.85735595  0.27183357]
 [ 0.6721261  -0.669318    0.31660056]
 [ 0.04504218  0.01025204  0.9989264 ]
 [ 0.96921414 -0.24550118  0.01836177]
 [ 0.97874224 -0.20497696  0.00561668]]
min: [-0.9999675 -0.9999648 -0.9999883]
max: [0.9999797  0.9999827  0.99999434]
======================================================================
file : segment20.npy
shape: (75941,)
dtype: int64
first 10: [ 0  0 -1  0  0 -1  0  0  0  0]
min/max : -1 19
unique sample: [-1  0  1  3  5  9 10 19]
======================================================================
file : segment200.npy
shape: (75941,)
dtype: int64
first 10: [ 0  0 -1  0  0 -1  0  0  0  0]
min/max : -1 172
unique sample: [ -1   0   2   5  10  11  13  16  61  83 172]

## 1. `coord.npy`

这个最重要：

```
coord.shape
```

一般是：

```
(N, 3)
```

内容：

```
x y z
```

例如：

```
[[ 1.231  2.103  0.114]
 [ 1.229  2.105  0.116]
 [ 1.228  2.107  0.117]
 ...
]
```

就是实际 3D 点的位置。

Pointcept 预处理脚本直接取原始 ScanNet mesh 顶点前三列：

```
coords = vertices[:, :3]
```

并存成 `float32`。

这和你自己的：

```
XYZRGB.ply
```

里的：

```
X Y Z
```

本质完全一样。

------

# 2. `color.npy`

形状：

```
(N, 3)
```

内容：

```
R G B
```

例如：

```
[[143 120 101]
 [141 119 100]
 [139 118  99]]
```

官方预处理保存为：

```
uint8
```

范围一般：

```
0 ~ 255
```



训练时配置里还会：

```
NormalizeColor
```

将：

```
0 ~ 255
```

转换到类似：

```
0 ~ 1
```

当前 Pointcept 的 `NormalizeColor` 就是：

```
color = color / 255
```



------

# 3. `normal.npy`

形状：

```
(N, 3)
```

表示：

```
nx ny nz
```

即点的表面法向量。

例如：

```
[[ 0.13 -0.22  0.96]
 [ 0.15 -0.20  0.96]
 ...
]
```

一般长度约：

$\sqrt{n_x^2+n_y^2+n_z^2}=1$

Pointcept 的 ScanNet 预处理程序是从 mesh 三角面计算 vertex normal 的。

这个以后对你的医学表面点云会很有价值，因为：

```
XYZ
```

只能告诉网络：

> 点在哪里。

而：

```
Normal
```

还能告诉网络：

> 这个地方表面朝哪个方向、局部曲面形态怎样。

------

# 4. `segment20.npy`

这是你目前最应该关注的标签。

形状：

```
(N,)
```

例如：

```
[1, 1, 1, 1, 5, 5, 5, 0, 0, ...]
```

每个点对应一个语义类别。

ScanNet20 一共：

```
20 classes
```

比如：

```
wall
floor
cabinet
bed
chair
sofa
table
door
window
bookshelf
...
```

外加：

```
-1
```

表示：

```
ignore
```

Pointcept 的 `ScanNetDataset` 加载后会把：

```
segment20.npy
```

改名成：

```
data_dict["segment"]
```

所以你在网络里看到的：

```
segment
```

实际上不是硬盘上的 `segment.npy`。

而是：

```
segment20.npy
        ↓
ScanNetDataset
        ↓
segment
```

这是一个很关键的区别。

------

# 5. `segment200.npy`

和 `segment20.npy` 一样：

```
(N,)
```

但它是 ScanNet200：

```
200 类
```

如果跑：

```
ScanNetDataset
```

通常用 20 类。

如果跑：

```
ScanNet200Dataset
```

就读取：

```
segment200.npy
```

官方两个 dataset class 就是这样区分的。

所以你之前跑的是普通：

```
scannet / ptv3
```

如果配置是 ScanNet20，那么实际监督标签就是：

```
segment20.npy
```

------

# 6. `instance.npy`

形状也是：

```
(N,)
```

但它不是“这是什么类别”，而是：

> **这个点属于哪一个具体物体。**

比如一间房里两把椅子：

```
semantic:

chair chair chair chair chair chair
```

但 instance：

```
3 3 3 8 8 8
```

表示：

```
前三个点 → chair #3
后三个点 → chair #8
```

所以：

```
segment
```

回答：

> 是什么？

而：

```
instance
```

回答：

> 是哪一个？

如果你现在只做 **semantic segmentation**：

```
instance.npy
```

暂时不重要。

那么第一版 dataset 直接：

```
your_scene/
├── coord.npy
├── color.npy
└── segment.npy
```

对应：

```
PLY XYZ
   ↓
coord.npy

PLY RGB
   ↓
color.npy

人工标注
   ↓
segment.npy
```

如果再算法向量：

```
your_scene/
├── coord.npy
├── color.npy
├── normal.npy
└── segment.npy
```

就已经和 ScanNet 的核心结构几乎一致。

直接看看这个 scene **每个语义类别各有多少点**：

`python - <<'PY'
import numpy as np

root = "data/scannet/train/scene0224_00"

seg = np.load(root + "/segment20.npy")

labels, counts = np.unique(seg, return_counts=True)

print("Total points:", len(seg))
print()

for label, count in zip(labels, counts):
    ratio = count / len(seg) * 100
    print(
        f"label={label:3d} "
        f"points={count:6d} "
        f"ratio={ratio:6.2f}%"
    )
PY`

