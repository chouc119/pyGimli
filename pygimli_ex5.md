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
from pygimli.physics import ert 
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

It shows high resistivities in the dune area and low resistivity (salt or brackish water) towards the beach. From the dune, freshwater is moving towards the sea. We want to have a look at the measured and modelled data.

There is some topography present in the data set which affects the geometric factor.

We need a geometric factor to convert them into apparent resistivities. We use ```ert.createGeometricFactors``` which is calculated numerically, 

i.e. by a refined mesh with quadratic shape functions.

Note that repeated calls do not cost runtime as the result is cached.

```python
data['k'] = ert.createGeometricFactors(data, numerical=True)
```
Additionally we compute an analytical geometric factor based on a flat-earth assumption.

We are going to show the geometric effect, i.e. the ratio of both geometric factors (Rücker et al. 2006).
```python
k0 = ert.createGeometricFactors(data)
ert.show(data, vals=k0/data['k'], label='Topography effect',
        cMap="bwr", cMin=0.8, cMax=1.25, logScale=True);
```
![image](https://user-images.githubusercontent.com/101647060/181484535-2dc23179-364c-447e-ae49-3b207067299a.png)

For the inversion, we need an error estimate, i.e. retrieved from reciprocal analysis. 
Errors usually consist of a relative error and an absolute error (voltage gain). Here we assume typical values of 3% and 50µV.

```python
data['err'] = ert.estimateError(data, 
                                absoluteUError=0.00005, # 50µV
                                relativeError=0.03)  # 3%
ert.show(data, data['err']*100, label="error [%]");
```
![image](https://user-images.githubusercontent.com/101647060/181484901-ccff259f-0bbb-47b7-b13a-00940e5d53c2.png)

```python
mgr = ert.ERTManager(data)
mgr.invert(verbose=True,
           #paraDX=0.3, paraMaxCellSize=10, paraDepth=20, quality=34,
           lam=50, 
           zWeight=0.3,
          )
```

![image](https://user-images.githubusercontent.com/101647060/181485067-a6aca0ac-4d02-4920-bb58-b07411410f6c.png)

```python
mgr.showResult(cMin=1, cMax=500, xlabel="x (m)", ylabel="z (m)");
```
![image](https://user-images.githubusercontent.com/101647060/181485195-e986f5e4-68b9-41a6-be89-e9ea12bdfbd3.png)


It shows high resistivities in the dune area and low resistivity (salt or brackish water) towards the beach. From the dune, freshwater is moving towards the sea. We want to have a look at the measured and modelled data.

```python
mgr.showFit();
```
![image](https://user-images.githubusercontent.com/101647060/181485427-fb27ffb6-a8ad-4d8d-9a05-7a7b7a22fa29.png)

Alternatively, we might look at the misfit distribution to see whether it is uncorrelated Gaussian.

```python
misfit = pg.log(mgr.inv.response / mgr.data["rhoa"]) / data["err"]
pg.show(data, misfit, cMap="bwr", cMin=-3, cMax=3);
```

![image](https://user-images.githubusercontent.com/101647060/181485532-7425be5b-0566-4ec7-aaf5-b4b055173743.png)


A good and simple resolution measure is the coverage, i.e. sum of sensitivities.

```python
pg.show(mgr.paraDomain, mgr.coverage());
```
![image](https://user-images.githubusercontent.com/101647060/181485706-853014d9-656a-4e97-a839-0e20371fa00c.png)

```python
# 查看反演下的實際網格
ax, _ = pg.show(mgr.mesh, markers=True, showMesh=True, clipBoundaryMarkers=True);
# ax.set_xlim(-20, 180);
# ax.set_ylim(-40, 5);
```python
![image](https://user-images.githubusercontent.com/101647060/181486027-95e93926-fa0d-4b50-b11b-0466f83495e0.png)


Now we want to create our own mesh and include a known structural boundary as prior information.

```python
geo = pg.meshtools.createParaMeshPLC(data, paraMaxCellSize=100)
line = pg.meshtools.createLine(start=[60, 0], end=[160, 0], marker=1)
geo += line
ax, _ = pg.show(geo);
ax.set_xlim(-20, 180);
ax.set_ylim(-30, 5);
```
![image](https://user-images.githubusercontent.com/101647060/181486167-c250f19c-a4b1-4cc9-8921-73848984dcd9.png)

```python
mesh = pg.meshtools.createMesh(geo, quality=34)
ax, _ = pg.show(mesh)
ax.set_ylim(-30, 10)
ax.set_xlim(-20, 180);
```
![image](https://user-images.githubusercontent.com/101647060/181486262-350760b1-6aa9-43a8-b318-a324759170c9.png)

In this example we use the ```ert.ERTManager()``` from the ```ert``` physics module.

```python
mgrConstrained = ert.ERTManager()
mgrConstrained.invert(data=data, verbose=True, lam=50, mesh=mesh, maxIter=5)
mgrConstrained.showResult(cMin=1, cMax=500, xlabel="x (m)", ylabel="z (m)");
```
![image](https://user-images.githubusercontent.com/101647060/181486417-4ee9c6b9-abed-480a-8018-7a43b754b87b.png)
