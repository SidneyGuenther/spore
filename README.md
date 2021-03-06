# Spore

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_header.png "spore header")

> Paintable particle node for scattering objects in Maya

Spore is a set of Maya plug-ins and scripts for Autodesk Maya.
The Spore toolset is based on the **sporeNode** and **sporeContext**.

The **sporeNode** is a custom dependency graph node that produces an *instanceData*
attribute which is designed to be hooked to Maya's particle instancer node.

The **sporeContext** is designed to place and manipulate points in the *instanceData* attribute.


# Motivation

While particle instancing in Maya is a popular way to populate environments,
the means of manipulating per-particle attributes are limited and require some
specific knowledge.

Spore tries to create a more intuitive interface to creating and manipulating points that drive
Maya's particles instancer. Spore allows to scatter, paint and modify points in an easy an intuitive way.
Points can either be place with one of four different sampling algorithms or with an interactive brush context.
Once points have been placed in the scene they can be interactively and effortless modified using one
of four different "grooming" brushes.


# Design

It is important to note that spore is probably not the right tool if you want to scatter
massive environments. However, following some simple guidelines you can still achieve
reasonable results.<br\>
Probably most important is to split up your environment in many different spore nodes.
It is better to have lots of nodes with fewer points than having a single node with millions
of points. This is because holding points is rather effortless for spore while
modifying them, especially large amounts, is expensive. Once you have placed a certain amount of
points, try to create a new node on the same target surface. The Spore Manager can help
to streamline this particular workflow.

	
# Installation

1. clone the git repo
```
git clone https://github.com/wiremas/spore
```
2. edit the spore.mod file to match the spore location on you machine:
```
spore any /path/to/spore/
```
3. finally copy the spore.mod file to a location your MAYA_MODULE_PATH environment variable points.<br/>
   Run the following mel command to find an appropriate location
```
getenv("MAYA_MODULE_PATH")
```


# Dependencies

In order to run **spore** you need to have **scipy** and **numpy** installed.<br/>

- If you're running Linux or Mac just install using pip.<br/>
- If you're running Windows you should run...
```
pip install -i https://pypi.anaconda.org/carlkl/simple numpy
pip install -i https://pypi.anaconda.org/carlkl/simple scipy
```
... or try to follow this [guide](https://forums.autodesk.com/t5/maya-programming/guide-how-to-install-numpy-scipy-in-maya-windows-64-bit/td-p/5796722).


# Usage

Load the spore plugin from Maya's plugin manager.

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_plugin.png "spore plugin")

From the *"Spore"* menu select *"Spore Manager"* to open the ui.

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_menu.png "spore menu")


# sporeNode

The **sporeNode** takes an **inMesh** input attribute and creates an **instanceData** output attribute.
The **instanceData** attribute is designed to drive a particle instancer. To specify a target mesh connect the target's shape
**outMesh** attribute to the spore's **inMesh** attribute.

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_network.png "spore node network")

---

### Instanced Objects

The *Instanced Objects* menu give quick acces to adding or removing objects to the
instancer without the need to specifically select the instancer node.
The list mirrors the list of source objects displayed in the instancer node.

In addition the list also acts as modifier for most of the brush and sample operations.
Selecting one or more objects from the list will enable *Exclusive Mode*.
In *Exclusive Mode* only the selected objects IDs are considered for the specified operation.

---

### Instance Transforms

The *Instance Transforms* menu allows set transformations for instanced objects

| Attribute					|														|
| ------------------------- |:-----------------------------------------------------:|
| Align To					| Define a target vector for rotation					|
| Weight					| Set weight of the target vector						|
| Min Rotation				| Minimum rotation values								|
| Max Rotation				| Maximum rotation values								|
| Uniform Scale				| Scale XYZ uniformly (X defines the uniform value)		|
| Min Scale					| Minimum scale values									|
| Max Scale					| Maximum scale values									|
| Scale Factor				| Factor for increasing/decreasing scale values			|
| Randomize / Smooth		| Factor for randomizing/smoothing scale values 		|
| Min Offset				| Minimum offset along surface normal					|
| Max Offset				| Maximum offset along surface normal					|

---

### Brush

The *Brush* menu exposes basic brush settings.
For more detail on the individual brushes see [sporeContext](#sporeContext) section.

| Attribute					|														|
| ------------------------- |:----------------------------------------------------- |
| Tool						| Set the brush context to the specified mode			|
| Radius					| Define the brush radius								|
| Number Of Samples			| Number of points created by a single brush tick 		|
| Min Distance 				| Minimum distance between each brush tick				|
| Falloff					| Define a falloff for the brush						| 

---

### Emit

The *Emit* menu controls settings used for the next sampling operation.

| Attribute					|														|
| ------------------------- |:----------------------------------------------------- |
| Type						| Set the sampling type									|
| Number of Samples			| Number of samples generated by the random sampler		|
| Cell Size					| Cell size for the jitter gird							|
| Min Radius				| Minimum radius for the 3d disk sampler				|
| Min Radius 2d				| Minimum radius for the 2d disk sampler				|

The *sporeNode* features four different sampling types:
1. **random sampling**<br/>
   Distribute points uniform randomly across the surface.
   Fast, but sampled points tend to clump together or form empty spots<br/>
   <br/>   
2. **jitter grid**<br/>
   A uniform grid is created around an initial point cloud, generated by the random sampler.
   Than from each cell a random point is choose. This results in higher sampling times since
   some of the sampled points are discarded but the result looks slightly more pleasing.<br/>
   <br/>
3. **3d poisson disk sampling**<br/>
   Generated poisson disk samples are at least the distance r apart from each other.<br/>
   This results in very pleasing result but takes a lot longer to sample.
   Note: be carefure with the minRadius attribute since small values increase computation<br/>
   http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.591.736&rep=rep1&type=pdf<br/>
   <br/>
4. **2d poisson disk sampling**<br/>
   Generate poisson disk samples in uv space that are at least the distance r apart from each other.<br/>
   Note: be carefure with the minRadius attribute since small values increase computation<br/>
   Note: samples will only be generated in the uv space from 0 to 1.
   Therefore the radius can not exceed 1.<br/>
   http://www.cs.ubc.ca/~rbridson/docs/bridson-siggraph07-poissondisk.pdf<br/>
   <br/>
   
![alt text](https://github.com/wiremas/spore/blob/master/res/spore_sampler.png "spore sampler")

*Geometry Cache*<br/>
<br/>
To accelerate sampling and other operations, target meshes will be cached in memory.
These cache objects are shared between nodes. Meaning that even if you create multiple spore nodes for the same mesh, the cache has to be created only once.
Depending on the size of the mesh, caching might take a few seconds.<br/>
To prevent a lot of computational overhead the cache will not update automatically when the mesh is transformed or deformed.
In general, every time the cache is requested, it will evaluate if there have been any changes to the mesh.
If this is the case it will automatically recache the mesh.
Just note, that whenever you modify your mesh you add some extra computation to your operations.

There is also the option to force a recache in the **Cache** menu.

#### Filtering

Sampled points can be filtered using one of the following operations:

1. **Texture Filter**<br/>
   The texture filter removes points based on an the incoming shading network's red channel.<br/>
   *Note*: Texture filtering is based on sampling uv coordinates for corresponding
   points. Sampling uv values take a long time on high poly meshes!<br/>
  
   | Attribute					|														|
   | -------------------------- |:----------------------------------------------------- |
   | Use Texture				| Enable texture filtering								|
   | Texutre					| Connect a shading network to evaluate					|

2. **Altitude Filter**<br/>
   Filters points based on hight relative to the targets bounding box.<br/>
   Use the fuzziness attribute to create a soft transition<br/>
   
   | Attribute					|														|
   | -------------------------- |:----------------------------------------------------- |
   | Min Altitude				| Set lower filter bound								|
   | Max Altitude				| Set upper filter bound								|
   | Min Altitude Fuzziness		| Set lower bound fuzziness								|
   | Max Altitude Fuzziness		| Set upper bound fuzziness								|

3. **Slope Filter**<br/>
   Filters points based on the angle between the surfaced normal and the world up vector<br/>
   Use the fuzziness attribute to create a soft transition<br/>

   | Attribute					|														|
   | -------------------------- |:----------------------------------------------------- |
   | Min Slope					| Minimal angle											|
   | Max Slope					| Maximal angle											|
   | Slope Fuzziness			| Set fuzziness											|


# sporeContext

The *sporeContext* is an interactive brush tool that can manipulate the sporeNode's 
"instanceData" attributes. The context features seven different modes plus modifiers.
Depending on the active context mode different brush controls are available directly
on the sporeNode.

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| b-click + drag  			| Modify brush radius									|


### Undo / Redo

The *sporeContext* fully supports undo.
Unfortunately, there is currently no way to redo operations.


### Place Mode

Place a singe instance per brush tick<br/>

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| Drag Mode												|
| Meta						| Align to stroke direction	*							|

* the brush radius attribute also affects the way the stroke direction vector is generated. 
A larger brush radius will generate a more steady stroke at the cost of less precision.

### Spray Mode

Place n instances per brush tick within the given radius<br/>

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| Drag Mode												|
| Meta						| Align to stroke direction	*							|

* the brush radius attribute also affects the way the stroke direction vector is generated. 
A larger brush radius will generate a more steady stroke at the cost of less precision.

### Scale Mode

Scale all instance within the given radius<br/>

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| Smooth												|
| Meta						| Randomize												|


### Align Mode

Align all instances within the given radius to the specified axis

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| Smooth (same as align to normal)						|
| Meta						| Randomize												|


### Move Mode

Move all instances within the given radius along the stroke direction


### Id Mode

Modify the objectIndex points within the radius to the specified ID.
ID can be specified by enabeling *Exclusive Mode*.
If the meta modifier key is pressed, n number of samples are randomly set to the specified objectIndex per brush ticks.
Brush ticks can be modified using the *Min Distance* attribute

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| ---													|
| Meta						| Assign random											|

### Delete Mode

Remove all instances within the given radius.
If the meta modifier key is pressed, n number of samples are randomly deleted per brush ticks.
Brush ticks can be modified using the *Min Distance* attribute.
While the tool is active objects can be restored using the shift modifier key.
Once you leave the context, it is no longer possible to restore objects.

| Modifier					|														|
| ------------------------- |:----------------------------------------------------- |
| Shift						| Restore (Works only until the context is left)		|
| Meta						| Delete random											|

### Exclusive Mode
In **Exclusive Mode** the context works only on certain objectIndex IDs.
To activate **Exclusive Mode** select the desired sources objects in the
sporeNode's instance object list.




# sporeCommand
The spore command can be used to create a new spore setup from Maya's script editor.
```
cmds.spore()
```


# Spore Manager

The **Spore Manager** is a interface which lists all spore nodes in the scene.<br/>
It helps to quickly switch between different setups and display modes or create new spore nodes<br/>

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_manager.png "spore manager")

| Navigation				|														|
| ------------------------- |:----------------------------------------------------- |
| Control-click				| Multi select											|
| Right-click				| Context menu											|
| Double-click				| Rename												|


| Display modes				|														|
| ------------------------- |:----------------------------------------------------- |
| geometry					| Set instancer LoD to geometry							|
| bounding box				| Set instancer LoD to bounding box						|
| bounding boxes			| Set instancer LoD to bounding boxes					|
| solo						| Solo the instancer connected to the specified spore node |

		
# Spore Reporter

The **Spore Reporter** helps to track issues and bugs or simply allows you to suggest a feature.
When *Report Bugs* is enabled in the spore globals the **Spore Reporter** pops up whenever
an unhandled error occurs.
To send reports automatically enable *Automatic Report* in the spore globals.

![alt text](https://github.com/wiremas/spore/blob/master/res/spore_reporter.png "spore reporter")

The report tool wraps an onlineservice called [anonymouse.org](http://anonymouse.org/anonemail_de.html).
Message delivery can take up to twelve hours.

