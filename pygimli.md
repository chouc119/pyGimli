# pyGIMLi Tutorial

### 安裝pyGIMLi
- 開啟 Anaconda Prompt
- 輸入conda create -n pg -c gimli -c conda-forge pygimli=1.2.6
- (上述步驟為，創建一個叫做pg的環境，並在環境內載入套件 pygimli)
--------------------------------------------------------------------------------------------------
## Create a subsurface geometry & mesh

#### Import module
```python
import pygimli as pg                  # 匯入pygimli並命名為pg
from pygimli import meshtools as mt   # 從pygimli匯入meshtools並命名為mt
from pygimli.viewer import showMesh   # 從pygimli.viewer匯入showMesh
```
--------------------------------------------------------------------------------------------------

### Geometry creation
Creating a subsurface model. We create a geometry definition of the domain which is a simple rectangle. 

The inputs are start and end points and you can also set a specific marker. 

The default marker start is 1.

#### Create a "world" (i.e., a layered subsurface)
```python
# 創建模型的尺寸設定
left = -30
right = 30
depth = 25
```
```python
world = mt.createWorld(start=[left, 0],
                       end=[right, -depth],
                       layers=[-5])
pg.show(world); 
```
```pg.show()``` is a handy tool to show your world and how it is being built
![image](https://user-images.githubusercontent.com/101647060/181185065-da94d37c-0109-4015-b5e0-b4ecea8d525a.png)

#### Create a line (e.g., dipping interface)
```python
line = mt.createLine(start=[left, -20], end=[right, -15])
pg.show(line);
```
![image](https://user-images.githubusercontent.com/101647060/181186212-49a407d4-1faf-45f3-8aef-234793e3b5a4.png)

```python
geometry = world + line   #將剛剛創建的world加上剛剛創建的line
pg.show(geometry);
```
![image](https://user-images.githubusercontent.com/101647060/181186553-f4fed9c9-2758-4e0a-a26c-813481cd508e.png)

#### Create a polygon (e.g., geological body)
Next we create a polygon that is closed and contains three vertices.

You can add more nodes to the polygon with ```addNodes``` and define how to interpolate between these nodes with ```interpolate```

```python
body = mt.createPolygon([[-8, -12], [0, -7], [8, -13]],
                        isClosed=True, marker=3,
                        addNodes=5,
                        interpolate='spline', 
                        )
pg.show(body, showNodes=True);
```
![image](https://user-images.githubusercontent.com/101647060/181187686-130e696e-61d8-4da3-8ef9-68cdd523a18f.png)

```python
geometry += body    #+= 這種運算子具有相加同時指派的功能
pg.show(geometry, boundaryMarkers=True);
```
![image](https://user-images.githubusercontent.com/101647060/181189209-b084b27a-2534-47e1-b74a-b60506227cb3.png)



#### Mesh creation & quality
pyGIMLi has different ways to create meshes. 

```mt.createMesh``` creates a mesh using Triangle, a two-dimensional constrained Delaunay mesh generator.

The additional input parameters control the maximum triangle area and the mesh smoothness. 

The quality factor prescribes the minimum angles allowed in the final mesh. 

This can improve numerical accuracy, however, fine meshes lead to increased computational costs. 

Notice that we are now using ```showMesh``` which is convenient for 2D mesh visualization.

```python
from pygimli.viewer import showMesh
mesh = mt.createMesh(geometry, 
                     area=1.0,
                     quality=33,
                     smooth=[2, 4] # [0:no smoothing or 1:node center or 2:weighted node center, # of iter]
                    )
showMesh(mesh, markers=True, showMesh=True); 
```

![image](https://user-images.githubusercontent.com/101647060/181190199-08ed1316-7d56-4165-9be9-b12a274f02e8.png)

Save geometry and mesh for later re-use


