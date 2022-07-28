# pyGIMLi Tutorial

## Inversion settings: Inverting electrical resistivity field data

### Objectives:
1. 下載電阻率場數據集 "data/beach.ohm"
2. 顯示數據的地形效果
3. 使用 ERT Method Manager 對數據進行反演

```python
import matplotlib.pyplot as plt
plt.style.use("seaborn-notebook")
```
```python
import matplotlib.pyplot as plt
import pygimli as pg
from pygimli.physics import ert  # the module
```
We first load a sample data file, which was measured on the beach of Borkum island in the North sea.

```python
data = ert.load("data/beach.ohm")
print(data)
```
![image](https://user-images.githubusercontent.com/101647060/181452600-2d3a6d89-eaf5-4655-8200-931644101b9d.png)

The data set obviously has, besides current and potential indices, voltage and current.

```python
fig, ax = plt.subplots()
ax.plot(pg.x(data), pg.z(data), 'o-')
ax.set_aspect(1.0)
```
![image](https://user-images.githubusercontent.com/101647060/181452776-3809bcab-feb9-4676-84a3-0972ac932e24.png)

There is some topography present in the data set which affects the geometric factor.

We need a geometric factor to convert them into apparent resistivities. We use ```ert.createGeometricFactors``` which is calculated numerically, 

i.e. by a refined mesh with quadratic shape functions.

Note that repeated calls do not cost runtime as the result is cached.

```python
data['k'] = ert.createGeometricFactors(data, numerical=True)
```

