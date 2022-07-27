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
Creating a subsurface model. We create a geometry definition of the domain which is a simple rectangle. The inputs are start and end points and you can also set a specific marker. The default marker start is 1.

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

