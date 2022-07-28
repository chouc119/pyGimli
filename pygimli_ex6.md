# [pyGIMLi Tutorial](https://github.com/gimli-org/transform2021)

## Inversion with custom forward operators

### Objectives

1. Show how to use an own forward operator along with the pyGIMLi inversion
2. Generate a synthetic model for drone-base CSEM
3. Invert the synthetic data with pyGIMLi

We use a modelling tool called ```empymod``` from Dieter Werthmüller. The repository can be found on https://emsig.github.io/

```empymod``` provides electromagnetic modelling for 1D VTI media with arbitrary transmitters and sensors in 3D space. For installation we call:

    conda install -c conda-forge empymod

```python
import matplotlib.pyplot as plt
plt.style.use("seaborn-notebook")
```
```python
import pygimli as pg
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.colors as clr
import empymod
```
First, we define some settings that are to be used. We assume an electric dipole transmitter perpendicular to the x-z place of length ```txLen```
and magnetic receivers in the air (e.g. on a drone) along a profile line. Date are obtained in frequency domain for a number of frequencies.

```python
# receiver setting
rx = np.arange(50., 501., 50)
ry = np.zeros(rx.size)
# empymod arguments including transmitter
txLen = 400
inpdat = {'src': [0, 0, txLen/2, -txLen/2., 0.1, 0.1], 'strength': 1,  # ground
           'mrec': True, 'rec': [rx, ry, -20, 0, 90],                  # Hz air
                                                                       # 'mrec': False, 'rec': [rx, ry, 0.1, 90, 0],  # Ey ground
          'srcpts': 11, 'htarg': {'pts_per_dec': -1}, 'verb': 1}
                                                                       # frequencies
freqs = [10., 20, 50, 100, 200, 500, 1000, 2000., 5000.]               # Hz
```

We define a function for calling empymod with vectors of resistivity and depth.

```python
def fwd(res, dep):
    """Call empymods function bipole with the above arguments."""
    assert len(res) == len(dep)
    OUT = np.zeros((len(freqs), len(rx)), dtype=np.complex)
    for i, f in enumerate(freqs):
        OUT[i, :] = empymod.bipole(res=np.concatenate(([2e14], res)),
                                   depth=dep, freqtime=f, **inpdat)

    return OUT
```

Next we create a functon to show the real and imaginary part of the data as image plots.

```python
def showData(A, ax=None, sym=True):
    """Show data as two subplots for real and imaginary part."""
    if ax is None:
        fig, ax = plt.subplots(1, 2)

    if A.ndim == 1: # long vector
        reim = np.reshape(A, (2, -1))
        A = np.reshape(reim[0], (len(freqs), -1)) + \
            np.reshape(reim[1], (len(freqs), -1)) * 1j
 
    norm = clr.NoNorm()
    cmap = "Spectral_r"
    if sym:
        mi = np.min(np.abs(A))
        ma = np.max(np.abs(A))
        norm = clr.SymLogNorm(mi, vmin=-ma, vmax=ma, base=10)#, linscale=mi)
        cmap = 'RdBu_r'

    imr = ax[0].imshow(A.real, norm=norm, cmap=cmap)
    plt.colorbar(imr, ax=ax[0], orientation="horizontal")
    imi = ax[1].imshow(A.imag, norm=norm, cmap=cmap)
    plt.colorbar(imi, ax=ax[1], orientation="horizontal")
    if A.shape[1] == len(rx):
        for a in ax:
            a.xaxis.set_ticks(range(len(rx))[::3])
            a.xaxis.set_ticklabels([str(int(r)) for r in rx][::3])
            a.set_xlabel('x (m)')
            a.yaxis.set_ticks([y for y in range(len(freqs))])
            a.yaxis.set_ticklabels([str(int(f)) for f in freqs])
            a.set_ylabel('f (Hz)')

    ax[0].xaxis.set_ticks([])
    ax[0].set_xlabel('')
    ax[0].set_title("real part")
    ax[1].set_title("imaginary part")
    plt.tight_layout()
    return ax
```
    
We now create a synthetic model of a good conductor embedded in a resistive halfspace.
    
```python
depth = [0, 100, 200]   # m
res = [100, 10, 100]
A0 = fwd(res, depth)
showData(A0);
```

To use the forward modelling in pyGIMLi, we create a class, derived from the pg modelling base class, 
and set the response and createStartModel functions, and init a 1D mesh for an Occam-type inversion.

```python
class myFwd(pg.Modelling):
    def __init__(self, depth):
        """Initialize the model."""
        self.dep = depth
        self.mesh1d = pg.meshtools.createMesh1D(len(self.dep))
        super().__init__()
        self.setMesh(self.mesh1d)
    
    def response(self, model):
        """Forward response."""
        A = fwd(model, self.dep)
        Avec = A.ravel()
        return np.hstack((Avec.real, Avec.imag))
        
    def createStartModel(self):
        return pg.Vector(len(self.dep)) * 100
```


We organize the real and imaginary part and add relative error to the data

```python
A0vec = A0.ravel()
data = np.hstack((A0vec.real, A0vec.imag))
# error = np.abs(data) * 0.01 + 0.00001
relativeError = np.ones_like(data) * 0.01
data *= (np.random.randn(len(data)) * relativeError + 1.0)
```
```python
depth_fixed = np.linspace(0., 300., 21)
fop = myFwd(depth_fixed)
resistivity = np.ones_like(depth_fixed) * 100
response = fop.response(resistivity)
showData(response);
```

We set up the pyGIMLi inversion usng the forward operator created by ```myFwd``` (fop) using ```setForwardOperator```

```python
inv = pg.Inversion()
inv.setForwardOperator(fop)
transModel = pg.trans.TransLog(1) # > 1 Ohmm
inv.transModel = transModel
```

```python
model = inv.run(data, relativeError, startModel=100, verbose=True)
```
![image](https://user-images.githubusercontent.com/101647060/181493514-0765ea62-be8f-4776-acbe-4e55c6f0d8e9.png)

```python
from pygimli.viewer.mpl import drawModel1D
fig, ax = plt.subplots()
drawModel1D(ax, np.diff(depth), res, color="red", label="synthetic")
drawModel1D(ax, np.diff(depth_fixed), model, color="blue", label="inverted")
```

![image](https://user-images.githubusercontent.com/101647060/181495311-fcc49be0-13de-4f56-b821-56e003cde073.png)

```python
# Plot the model response (to be compared to the data)
showData(response);
```

```python
# Plot the misfit
showData(inv.response-data);
```
