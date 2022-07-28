# [pyGIMLi Tutorial](https://github.com/gimli-org/transform2021)

## The equation level - Modeling the steady-state heat equation

### Loading the mesh
We load the mesh that has been previously generated in notebook 1 (mesh generation).
```python
mesh = pg.load("data/mesh.bms")
showMesh(mesh, markers=True, showMesh=True);
```
![image](https://user-images.githubusercontent.com/101647060/181425700-c6a37818-fd87-4a46-baa6-e590cdb8240e.png)

### The equation level

Now we solve the heat equation
![image](https://user-images.githubusercontent.com/101647060/181426043-7c4d1cee-0105-4b7d-9ac0-1681fbc88974.png)

for the unknown temperature **T** in dependence of a thermal diffusivity $\alpha$ and a heat source **f**. 

The parameter $\alpha$ needs to be known on the whole mesh. 

Therefore we create a map that associates an $\alpha$ value for every model region 0-3 and have a look at it:

```python
amap = [[0, 1.0], [1, 5.0], [2, 10.0], [3, 20.0]]      #marker, value
showMesh(mesh, amap, logScale=True);
```
![image](https://user-images.githubusercontent.com/101647060/181427647-a3962842-232e-40e8-b938-6dcc96e4551e.png)

### The finite-element solver

```pg.solve``` solves a partial differential equation using the function ```solveFiniteElement```.

Instead of a heat source, we fix temperatures at the boundaries (bc) using Dirichlet boundary conditions.

```python
bc={'Dirichlet':{-1:5.0, -2:12.0}}
#bc={'Dirichlet':{-1:20.0}, 'Neumann': {-2:-0.1}}
T = pg.solve(mesh, a=amap, bc=bc, verbose=True)
showMesh(mesh, data=T, label='Temperature $T$', cMap="hot",
         nLevs=11, showBoundary=True);
```
![image](https://user-images.githubusercontent.com/101647060/181428357-6d732575-033f-48ac-99a5-a6de8ea56e3c.png)


![image](https://user-images.githubusercontent.com/101647060/181428149-15a885c6-2d80-4e63-811d-e21839a57aa2.png)

We are now interested in the stream lines of the heat flow. 

Therefore we return the temperature axis and draw the streams on top of it with ```drawStreams```. 

Any other MPL plots could go here.

```python
from pygimli.viewer.mpl import drawStreams
ax, cb = showMesh(mesh, data=T, label='Temperature $T$', cMap="hot",
                  nLevs=11, showBoundary=True);
drawStreams(ax, mesh, T)
```
![image](https://user-images.githubusercontent.com/101647060/181429250-9741ef5d-cba5-4fd8-9607-8363b873fb52.png)


### Interpolation
Often, results are needed on another mesh, e.g. in another software. 

pyGIMLi brings along interpolation that uses the Finite Element properties in the function ```interpolate```

```python
grid = pg.createGrid(x=np.arange(-25, 25.1), y=np.arange(-25, 0.1))
Tgrid = pg.interpolate(mesh, T, grid.cellCenters())
ax, cb = showMesh(grid, data=Tgrid, label= 'Temperature $T$', cMap="hot",
                  nLevs=11, showBoundary=True, showMesh=True);
```
![image](https://user-images.githubusercontent.com/101647060/181429787-918ac04c-3f7a-413b-a9b4-850db6cb2c94.png)

```python
grid = pg.createGrid(x=np.arange(-25, 25.1), y=np.arange(-25, 0.1))
Tgrid = pg.interpolate(mesh, T, grid.positions())
ax, cb = showMesh(grid, data=Tgrid, label='Temperature $T$', cMap="hot",
                  nLevs=11, showBoundary=True, showMesh=True);
```
![image](https://user-images.githubusercontent.com/101647060/181430020-57c9ab7c-f6d2-45d3-9b8e-be92ee592010.png)

```python
# produce a temperature depth log (copy)
depth = np.arange(0, 25, 0.1)
pos = [[0, -d] for d in depth]
Tlog = pg.interpolate(mesh, T, pos)
fig, ax = plt.subplots()
ax.plot(Tlog, depth)
ax.set_ylim([max(depth), 0])
ax.set_title('Temperature Depth Log')
ax.set_xlabel('Temperature')
ax.set_ylabel('Depth')
```
![image](https://user-images.githubusercontent.com/101647060/181430134-46893bc8-0bcf-4a7d-858e-2d240e53ae5a.png)

------------------------------------------------------------------------------------------------------------------------------

### Appendix: Numerical accuracy

We could improve the solution by using a finer mesh

```python
meshH = mesh.createH2()
Th2 = pg.solver.solve(meshH, a=amap, bc=bc, verbose=True)
showMesh(meshH, data=Th2, label='Temperature $T$', cMap="hot",
         nLevs=11, showBoundary=True);
```
![image](https://user-images.githubusercontent.com/101647060/181430568-576a1272-5976-4051-8622-2b3de8d6d4c6.png)

![image](https://user-images.githubusercontent.com/101647060/181430495-aba97732-7965-4e78-a5cf-8e4f2b7f7680.png)

```python
meshP = mesh.createP2()
Tp2 = pg.solver.solve(meshP, a=amap, bc=bc, verbose=True)
showMesh(meshP, data=Tp2, label='Temperature $T$', cMap="hot",
         nLevs=11, showBoundary=True);
```
![image](https://user-images.githubusercontent.com/101647060/181430673-d460d383-1232-4907-b123-33b6f44c1ce6.png)

```python
pg.show(meshH, Th2-Tp2);
```
![image](https://user-images.githubusercontent.com/101647060/181430718-cfb820b5-7a07-4366-9d26-d0640c93d9c1.png)

