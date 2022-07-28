# pyGIMLi Tutorial

## Simulating crosshole traveltime measurements

###  Read in the previously defined geometry

We start by reading in the geometry defined in the first notebook using the ```mt.readPLC``` functionhttps.

```python
import pygimli as pg
import pygimli.meshtools as mt
import pygimli.physics.traveltime as tt

geometry = mt.readPLC("data/geometry.poly")
geometry.scale([0.5, 1, 1])

ax, _ = pg.show(geometry)
```
![image](https://user-images.githubusercontent.com/101647060/181431949-d2ded8d1-db19-4f51-a713-c91183f75373.png)


### Define source and receiver positions
```python
import numpy as np

n = 10                                                            # 每個鑽孔的來源和接收器數量
borehole = np.ones((n, 2)) * 10                                   # 右側鑽孔在 x = 10 m 位置
borehole[:,1] = np.linspace(-.5, -23, n)                          # 降至23m深度

sensors = np.vstack([borehole] * 2)
sensors[n:,0] *= -1                                               # 左側鑽孔在 x = -10 m 位置

ax.plot(sensors[:,0], sensors[:,1], "ko")
ax.figure
```
