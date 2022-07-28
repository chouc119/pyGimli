# [pyGIMLi Tutorial](https://github.com/gimli-org/transform2021)

## Introduction to method managers: Traveltime inversion

### Load Data
```python
data = tt.load("data/traveltime.dat")
print(data)
```
![image](https://user-images.githubusercontent.com/101647060/181446971-0cbec52f-af4a-4b4a-9d91-17f433fad9c8.png)

### Define function to display data
```python
# 為了顯示數據，我們定義了一個函數
from pygimli.viewer.mpl import showVecMatrix
def showCrossholeData(data, vals, **kwargs):
    d = -pg.y(data)                         # 感測深度
    ds = d[data["s"]]                       # 射擊深度
    dg = d[data["g"]]                       # 檢波器深度
    ax, cb = showVecMatrix(ds, dg, vals, label="apparent velocity (m/s)", **kwargs);
    ax.set_xlabel("shot depth");
    ax.set_ylabel("geophone depth");

showCrossholeData(data, data["t"])
```

### Create 2D mesh
```python
x = np.linspace(min(pg.x(data)), max(pg.x(data)), 15)
y = np.linspace(min(pg.y(data)), max(pg.y(data)), 19)
grid = pg.meshtools.createMesh2D(x, y)
ax, cb = pg.show(grid)
ax.plot(pg.x(data), pg.y(data), "ro");
```
![image](https://user-images.githubusercontent.com/101647060/181448568-be6a29e7-ddbd-450d-aa7e-73bbe1d6df11.png)

### Inversion
We create an instance of the ```TravelTimeManager``` class with the loaded data. 

Method Managers work as an interface for end-user interaction and can be seen as simple but complete application classes which manage all tasks of geophysical data processing. 

In this example we are using the TravelTimeManager from the TravelTime physics module where we use the ```invert``` method.

```python
mgr = tt.TravelTimeManager(data)
```

```python
mgr.invert(data, mesh=grid, 
           startModel=0.001,
           useGradient=False,
           #zWeight=1,
           secNodes=3,
           lam=50,
           verbose=True)
mgr.showResult();
```
![image](https://user-images.githubusercontent.com/101647060/181449110-435599b2-baa8-47de-881c-555e18b445b1.png)
![image](https://user-images.githubusercontent.com/101647060/181449139-5275c70d-3988-49b8-9294-82808a70baa3.png)

The Method Manager also has ```showResult()``` and ```drawRayPaths()``` methods.

```python
ax, cb = mgr.showResult();
mgr.drawRayPaths(ax);
```
![image](https://user-images.githubusercontent.com/101647060/181449395-e2843113-b5a4-4eac-9bb6-3acda3e5867d.png)

Now we look at the simulated data (left) and inverted response (right)

```python
fig, ax = plt.subplots(ncols=2)
showCrossholeData(data, data["t"], ax=ax[0])
showCrossholeData(data, mgr.inv.response, ax=ax[1])
```

```python
# 查看錯位
misfit = data["t"] - mgr.inv.response
showCrossholeData(data, misfit, cMap="bwr", cMin=-0.001, cMax=0.001)
```
