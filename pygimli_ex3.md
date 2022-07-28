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
![image](https://user-images.githubusercontent.com/101647060/181432861-fb837270-832e-4a4d-90f8-dd67ebbbaa9c.png)

```python
for pos in sensors:
    geometry.createNode(pos)
    
mesh = mt.createMesh(geometry, quality=33.5, area=1)
pg.show(mesh);
```
![image](https://user-images.githubusercontent.com/101647060/181432964-6ed9cc82-5a36-431c-819e-2fe0264ce139.png)

```python
ax, _ = pg.show(mesh, markers=True, showMesh=True)
ax.plot(sensors[:,0], sensors[:,1], "mo");
```
![image](https://user-images.githubusercontent.com/101647060/181438654-bb57d09f-f7db-4631-acb3-410606032554.png)

### Create measurement schedule
```python
scheme = tt.createCrossholeData(sensors)
print(scheme)
print(scheme.sensor(0))
np.column_stack((pg.x(scheme), pg.y(scheme)))
```
![image](https://user-images.githubusercontent.com/101647060/181439181-ed3b0b70-7d40-455b-941c-a5c2bd705dc2.png)

### Parameterize the subsurface
Now we populate the subsurface with property values on a cell-by-cell basis.

We assign values of velocities to all of the cells using ```mesh.cellMarkers()```. 

Note that the markers in this case, start with 0 which works because numpy uses 0 based indexing.

```python
# vmap = [[0, 800], [1, 500], [2, 1000], [3, 2000]]           # 或者直接填充
v = np.array([800,500,1000,2000])[mesh.cellMarkers()]         # 4個標記的速度
```
```python
ax, _ = pg.show(mesh, v, label=pg.unit("vel"));

shots = scheme["s"]
geophones = scheme["g"]

for s, g in zip(shots, geophones):
    ray = np.array([s, g])
    ax.plot(sensors[ray, 0], sensors[ray, 1], "w-", lw=0.5)
```
![image](https://user-images.githubusercontent.com/101647060/181443059-c9903be9-f03e-4243-956c-014076a0a4ab.png)

### Simulate traveltime measurements

The ```TravelTimeManager()``` class and method manager for travel time tomography. Below we are using the ```mgr.simulate function```. 

We explain more about the TravelTime Manager in the next notebook

```python
mgr = tt.TravelTimeManager()
data = mgr.simulate(mesh=mesh, scheme=scheme, slowness=1/v,
                   secNodes=4, noiseLevel=0.001, noiseAbs=1e-5, seed=1337)
print(data)
data.save("traveltime.dat")
```
```python
np.column_stack((data["s"], data["g"], data["t"]))[:12]
```
![image](https://user-images.githubusercontent.com/101647060/181446318-150963c1-09a0-4d06-8645-475befbd3e01.png)
