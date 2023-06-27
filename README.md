> This document introduces how to use fcad_pcb to export copper layers and vias to Freecad and then output to 3d step file for FEM analysis.
# 1 fcad_pcb

Reference:
>  [GitHub - realthunder/fcad_pcb: FreeCAD scripts for PCB CAD/CAM](https://github.com/realthunder/fcad_pcb)

> Requirement
> FreeCAD >= v0.17
> 
## 1.1 Steps
### 1.1.1 step 1
Download fcad_pcb from the github link [fcad_pcb.rar (github.com)](https://github.com/qaqcvc/fcad_pcb_new/releases/tag/1.0).

Extract and put fcad_pcb folder into the FreeCAD macro folder.
This folder can be opened in freecad macros setting shown below.
(e.g.: D:\\Users\\...\\Roaming\\FreeCAD\\Macro)

![image](https://github.com/qaqcvc/fcad_pcb_new/assets/52302145/9163dcb0-2f2e-4a50-8a1a-69c76e3350da)

![image](https://github.com/qaqcvc/fcad_pcb_new/assets/52302145/cd574673-4344-431d-b106-81b3d9494ded)



### 1.1.2 Step 2
Enable python console.
![image](https://github.com/qaqcvc/fcad_pcb_new/assets/52302145/fd91a1e2-09ee-454c-a98f-4ebed8371acd)


Create a new macro and type these codes:
![image](https://github.com/qaqcvc/fcad_pcb_new/assets/52302145/54393ca8-f7c9-42d2-b41c-f333dcfc1d43)

```python
from fcad_pcb import kicad
  
pcb = kicad.KicadFcad(r'your file location')
# e.g.: pcb = kicad.KicadFcad(r'D:\...\layout.kicad_pcb')
#pcb.makeCoppers()
# zone_inflate = 0.5*minimum trace width of the zones

pcb.add_feature = False

coppers = pcb.makeCoppers(shape_type='solid', holes=True, fuse=True)
# fuse means to make vias

Part.show(coppers)
```

Change 'your file location' to your model's location then click Run. The solid copper layers will be created including the Vias.

![image](https://github.com/qaqcvc/fcad_pcb_new/assets/52302145/218288bb-d1f8-4707-90ce-8b180753e368)


If Kicad cannot find fcad_pcb, consider mannually add path, see trouble shooting below.

### 1.1.3 Step 3
In case pads are needed (for assigning source and sink), we can use the plugin KiCadStepUP.

It can read the tracks from the layout file, then unite the copper layers with the pads. 

![[Pasted image 20230302122407.png]]
Then extrude certain height, e.g., 0.035mm
Then move pad above the track and merge.

Now the model is ready to go for Ansys simulation like the figure below:
![[Pasted image 20220901143604.png]]

# 2 Model preparation
## 2.1 Vias

Blind vias and burried vias are supported.
Minimal via hole 0.08mm.

the thickness of the vias is 35um
![[Pasted image 20230523122416.png]]
section view
![[Pasted image 20230523122345.png]]

## 2.2 Simplifed vias
---
Reference:
[inconsistent shape of via after applying pcb.via_bound=1 · Issue #38 · realthunder/fcad_pcb · GitHub](https://github.com/realthunder/fcad_pcb/issues/38)

[Simplify vias · Issue #22 · realthunder/fcad_pcb · GitHub](https://github.com/realthunder/fcad_pcb/issues/22)

---
For complex model, the vias need to be simplifed to speed up the simulation, if the dimension of the vias does not obviously affect the result, e.g. compared with skin depth.

The vias can be simplifed with these codes:

```python
from fcad_pcb import kicad

pcb = kicad.KicadFcad(r'D:\...\test.kicad_pcb')

pcb.add_feature = False
pcb.via_bound = 0.7862
pcb.zone_inflate = -0.1
coppers = pcb.makeCoppers(shape_type='solid', fuse=True)
Part.show(coppers)
```

pcb.via_bound = Ratio
Ratio means the ratio between the square via and circular via hole diameter.

For example: diameter = 130um, via thickness = 35um

pcb.zone_inflate: to minus the zone inflate from KiCaD, which may cause zone mistakenly intersect

Comparison between normal vias and simplified vias. (holes also removed from the copper layers)
![[Pasted image 20230524125939.png|500]]
The amount of the mesh is reduced 20 times with the same mesh operations.


## 2.3 Zones
If possible, minimal width 0.0254mm, as small as possible

# 3 Trouble shooting
## 3.1 Cannot find fcad_pcb
We can mannually add path into the macro file
```python
import sys
sys.path.append(r'C:\users\...\Roaming\FreeCAD\Macro')

# To double check
print(sys.path)
```
It could be also possible that freecad cannot find module 'future'. We can also manually download it and put into the macro document.

## 3.2 Part is not defined
(Not sure)
It is related to the layout itself. Probably, the exported model cannot be transformed into a shape. 

## 3.3 <class 'Part.OCCError'>: creation of circle failed
The size of the via is too small.
0.31,0.13 is acceptable

Another possible reason is ill-defined vias. E.g. a micro via is defined as burried via can also cause this problem.

## 3.4 No shape added
Could be a via issue. Could try change micro vias to through hole vias.

## 3.5 <class 'NameError'>: free variable 'cut_wires' referenced before assignment in enclosing scope
Could be no pads on the top and bottom layers. Leave some components at top and bottom layers with pads (not necessarily connected) could help to solve this issue.
