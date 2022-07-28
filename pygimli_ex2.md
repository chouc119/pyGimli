# pyGIMLi Tutorial

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
Mesh:  Mesh: Nodes: 1386 Cells: 2627 Boundaries: 4012

Assembling time:  0.026

Solving time:  0.007
![image](https://user-images.githubusercontent.com/101647060/181428149-15a885c6-2d80-4e63-811d-e21839a57aa2.png)
