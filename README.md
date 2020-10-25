<h1>Creating colliders with Scripting - Spark AR</h1>

You’ll learn how to detect a 2D rectangle and 3D cuboid collision in Spark AR. We will use JavaScript and Reactive Programming style. As a collider figure we will use AABB (axis-aligned bounding box) - a figure, which edges are oriented parallel to the coordinate axes, like in the picture below.

We will set up a blank scene with two planes. Then we will write a script that will change the plane's material color when they collide. Also we will visualize collider bounding boxes to make it easier to change their size.

<h2>Necessary assets</h2>

* box.obj - we will use it to visualize collider bounding boxes

<h2>Setting up the scene</h2>

Open Spark AR Studio and create a blank scene. Create two planes at the scene. Create a script. Your project’s scene and assets hierarchy should look this:

<h2>Detect cuboid intersection mathematically</h2>

Let’s break the problem into parts. If we learn how to check 1D or line-with-line intersection, we can easily do similar check for 2D and 3D figures. Now we are going to check if two lines intersect.

<h3>Two lines intersection</h3>

Two lines are intersecting when they are close enough to each other. And they aren’t intersecting, when they are far away enough from each other. What describes each line’s position is the coordinate of it’s center. What describes the line's borders around the center is length, divided by 2.

pic

The lines above are not intersecting, but how do we know it? Well, we know each line’s center coordinate and length. Let’s find how close the lines are to each other. This is a distance between centers (d). To find it we calculate the difference in center coordinates and take the modulus (absolute) of the value.
d=Ax-Bx
The distance between lines changes as the lines move. So we can compare this distance (d) to some value, that will describe the minimal possible distance between lines (dmin)before they start to intersect each other. How close the lines should be to each other, or what value should take d, before lines start to intersect? 

pic

From the picture above we see, that d should be less than line sizes around the center, or the sum of each line’s length divided by 2.
dmin=length(A)2+length(B)2
Now we clearly see that lines are not intersecting, because d>dmin. If they intersected, the picture would look like this:

The lines above are intersecting, because d<dmin. 

Let’s write the final formula of intersection condition.
intersection=Ax-Bxlength(A)2+length(B)2
Square and cuboid intersection
When we can detect 1D collision (intersection), we can detect 2D and 3D collision by detecting collision on each dimension separately, thanks to the Separating axis theorem. We will use rectangle or cuboid edges as checkable lines. First we will project each parallel edges pair on the corresponding parallel axis. Then we will check these projections for intersections. If the rectangle or cuboid edges projections are intersecting both at X, Y and Z axes, these figures are intersecting each other. 

Don’t forget that the rules above are working only for axis-aligned bounding boxes, because their edge’s projections will be equal and parallel to these edges.

<h2>Using Reactive Programming to write mathematical formula</h2>

To implement intersection condition formula we will need the modulus, addition, subtraction, division and less or equal operators. You can find each operator method name in the ReactiveModule page of Spark AR scripting reference.

Math Function | Reactive Operator
--------------|------------------
modulus (abs) | Reactive.abs()
addition | Reactive.add()
subtraction | Reactive.sub()
division | Reactive.div()
less or equal | Reactive.le()

Open the script and remove everything from it. Import Scene, Reactive and Diagnostics modules.

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");
```

Scene module allows us to access objects on scene. We will use it to get position and scale.
Reactive module gives as methods for reactive programming.
Diagnostics module lets you write messages to the console or watch reactive variable values.

<h3>One axis collision</h3>
To implement line intersection formula, we will declare a function that will accept each line coordinate and length and check them for intersection. It will return a reactive boolean that will determine are the lines intersecting or not. Copy this code to the script:

```javascript
function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
```

We’ve just rewrited the intersection formula using reactive operators. The line’s position will be scene object’s X, Y or Z coordinates. The length will be the size of the scene object’s on the X, Y or Z axis. For the default planes this size equals 0.1 units. 

To accept scene object’s from code, we will use Scene.root.findFirst() method, which returns a promise that’ll be resolved as the object is found. To easily run asynchronous code, we will declare an immediately invoked async function and write all code inside it. Copy this code:

```javascript
(async () => {
    // async code goes here
});
```

This function will be called immediately after it is initialized, and we will be able to run code asynchronously inside it.

Let’s access the plane's scene objects. Copy this code inside newly created async function:
```javascript
const plane0 = await Scene.root.findFirst("plane0");
const plane1 = await Scene.root.findFirst("plane1");
```

Operator await tells the program to wait until the following operation is done. The script will pause and wait before the Scene.root.findFirst() function returns the reference to the plane object, which we then save to a variable. Now your script should look like this:

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");

function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
 
(async () => {
    const plane0 = await Scene.root.findFirst("plane0");
    const plane1 = await Scene.root.findFirst("plane1");
})();
```

Let’s call the checkCollision() function for each axis separately, so we can see at which axes these planes are intersecting. We will watch the result of this function using Diagnostics.watch() function, which shows reactive variable value in real time. Add this code below the planes find operations:

```javascript
Diagnostics.watch("collision X", checkIntersection(plane0.transform.x, plane1.transform.x, Reactive.val(0.1), Reactive.val(0.1)));
Diagnostics.watch("collision Y", checkIntersection(plane0.transform.y, plane1.transform.y, Reactive.val(0.1), Reactive.val(0.1)));
Diagnostics.watch("collision Z", checkIntersection(plane0.transform.z, plane1.transform.z, Reactive.val(0.1), Reactive.val(0.1)));
```

We used the transform property of planes scene objects to get their coordinates on X, Y and Z axes. We wrapped lines length inside the Reactive.val() function to convert a Number value to ScalarSignal value, because we’re performing reactive operations with them. Now your script should look like this:

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");
 
function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
 
(async () => {
    const plane0 = await Scene.root.findFirst("plane0");
    const plane1 = await Scene.root.findFirst("plane1");
 
    Diagnostics.watch("collision X", checkCollision(plane0.transform.x, plane1.transform.x, Reactive.val(0.1), Reactive.val(0.1)));
 
    Diagnostics.watch("collision Y", checkCollision(plane0.transform.y, plane1.transform.y, Reactive.val(0.1), Reactive.val(0.1)));
    
    Diagnostics.watch("collision Z", checkCollision(plane0.transform.z, plane1.transform.z, Reactive.val(0.1), Reactive.val(0.1)));
})();
```

Save the file and return to Spark AR Studio. At the top of the window click View and then Show/Hide Console.

Now you see the console and reactive variables at the bottom of the window.


It says that there’s a collision on each axis. Try to move one plane away from another. The values should change.


<h2>Rectange and cuboid (2D/3D) collision</h2>

The objects are colliding if they’re colliding at all axes at the same time. To get a single condition variable for 3D collision, we have to combine each axis collision check. Also we have to declare object size at X, Y and Z axes separately. 

Let’s declare a new function named checkCollision3D(), which will take two scene objects and their sizes as inputs. We will use Reactive.point() to create a PointSignal value to store object size. To combine each axis collision check we will use Reactive.andList() function, which returns true while every variable inside the specified array returns true. We’ll put every separate axis collision condition inside this list. Copy this code below checkCollision() function declaration:

```javascript
function checkCollision3D(objectA, objectB, sizeA, sizeB) {
    return Reactive.andList([
        checkCollision(objectA.transform.x, objectB.transform.x, sizeA.x, sizeB.x),
        checkCollision(objectA.transform.y, objectB.transform.y, sizeA.y, sizeB.y),
        checkCollision(objectA.transform.z, objectB.transform.z, sizeA.z, sizeB.z)
    ]);
}
```

Now remove all Diagnostics.watch() lines and replace them with the following one:

```javascript
Diagnostics.watch("plane0 with plane1", checkCollision3D(
    plane0, plane1, 
    Reactive.point(0.1, 0.1, 0.1), 
    Reactive.point(0.1, 0.1, 0.1)
));
```

Your script should look like this:

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");
 
function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
 
function checkCollision3D(objectA, objectB, sizeA, sizeB) {
    return Reactive.andList([
        checkCollision(objectA.transform.x, objectB.transform.x, sizeA.x, sizeB.x),
        checkCollision(objectA.transform.y, objectB.transform.y, sizeA.y, sizeB.y),
        checkCollision(objectA.transform.z, objectB.transform.z, sizeA.z, sizeB.z)
    ]);
}
 
(async () => {
    const plane0 = await Scene.root.findFirst("plane0");
    const plane1 = await Scene.root.findFirst("plane1");
 
    Diagnostics.watch("plane0 with plane1", checkCollision3D(
        plane0, plane1, 
        Reactive.point(0.1, 0.1, 0.1), 
        Reactive.point(0.1, 0.1, 0.1)
    ));
})();
```

Save file and return to Spark AR Studio. Try to move the planes and watch as the debug value changes.

<h2>Refactoring: creating Entity class to describe scene objects</h2>

This collision code looks simple before you start to scale the project. Imagine if we need to detect collision of one object with an array of objects. In that case we will have to write a separate checkCollision3D() call for each possible pair of objects and then combine these conditions using Reactive.orList(). To simplify this and do this in a loop, we have to make scene objects abstract, so we can operate over them without knowing which exact scene objects we have. Let’s create a class that will describe an object at scene. We can use this class later to extend object capabilities - for example, add some properties such as Health, Speed or Spawn Rate, if we’re creating a game filter.

Currently Entity class instance should do the following:
* Store reference to scene object
* Contain object size information

<h3>Declaring Entity class</h3>
Let’s declare a new class. Insert this code below collision functions declarations:

```javascript
class Entity {
    constructor(name, size) {
        this.name = name;
        this.size = size;
    }
}
```

When we create an Entity instance, we’ll be calling the constructor() method of Entity. We’ll have to provide it with a scene object name and it’s size vector. Let’s write a method that will find a scene object using its name and save it as an Entity instance property. Add a method called create() to the Entity class, so the class looks like this:

```javascript
class Entity {
    constructor(name, size) {
        this.name = name;
        this.size = size;
    }
 
    async create() {
        this.sceneObject = await Scene.root.findFirst(this.name);
        return this;
    }
}
```

Now find the lines where we were saving references to plane’s scene objects and replace them with the following code:

```javascript
const plane0 = await new Entity("plane0", Reactive.point(0.1, 0.1, 0.1)).create();
const plane1 = await new Entity("plane1", Reactive.point(0.1, 0.1, 0.1)).create();
```

We create a new Entity instance and provide it with the scene object name and it’s size. Then we immediately call create() async method on this instance to link this new Entity to the scene object, and wait until this operation is done. Then we do the same with the second plane. 

<h3>Changing checkCollision3D() method to accept Entities</h3>

Now we have to make changes in the checkCollision3D() function, because plane0 and plane1 are no longer scene objects. They’re Entity instances now, so to access objects position we should type plane0.sceneObject.transform instead of plane0.transform. Replace the old checkCollision3D() function code with this one:

```javascript
function checkCollision3D(entityA, entityB) {
    return Reactive.andList([
        checkCollision(entityA.sceneObject.transform.x, entityB.sceneObject.transform.x, entityA.size.x, entityB.size.x),
        checkCollision(entityA.sceneObject.transform.y, entityB.sceneObject.transform.y, entityA.size.y, entityB.size.y),
        checkCollision(entityA.sceneObject.transform.z, entityB.sceneObject.transform.z, entityA.size.z, entityB.size.z)
    ]);
}
```

Now change the way we call checkCollision3D(). You should use Entities as function arguments.

```javascript
    Diagnostics.watch("plane0 with others", checkArrayCollision(plane0, [plane1, plane2]));
    Diagnostics.watch("plane0 with plane1", checkCollision3D(plane0, plane1));
```

After we’ve done with refactoring, the code should look like this:

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");
 
function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
 
function checkCollision3D(entityA, entityB) {
    return Reactive.andList([
        checkCollision(entityA.sceneObject.transform.x, entityB.sceneObject.transform.x, entityA.size.x, entityB.size.x),
        checkCollision(entityA.sceneObject.transform.y, entityB.sceneObject.transform.y, entityA.size.y, entityB.size.y),
        checkCollision(entityA.sceneObject.transform.z, entityB.sceneObject.transform.z, entityA.size.z, entityB.size.z)
    ]);
}
 
class Entity {
    constructor(name, size) {
        this.name = name;
        this.size = size;
    }
 
    async create() {
        this.sceneObject = await Scene.root.findFirst(this.name);
        return this;
    }
}
 
(async () => {
    const plane0 = await new Entity("plane0", Reactive.point(0.1, 0.1, 0.1)).create();
    const plane1 = await new Entity("plane1", Reactive.point(0.1, 0.1, 0.1)).create();
    const plane2 = await new Entity("plane2", Reactive.point(0.1, 0.1, 0.1)).create();
 
    Diagnostics.watch("plane0 with plane1", checkCollision3D(plane0, plane1));
})();
```

<h2>Detecting collision of one object with an array of objects</h2>

Let’s declare a new function called checkArrayCollision3D(). It will accept an entity and an array of other entities which we will be checking for collision with an entity. We will return an Reactive.orList(), which will contain collision checks of every pair of entity with other entities. Copy this code below other functions in the script:

```javascript
function checkArrayCollision3D(entityA, otherEntities) {
    return Reactive.orList(otherEntities
        .map(otherEntity => checkCollision3D(entityA, otherEntity))
    );
}
```

We’ve used .map() method on otherEntities array to return a new array. This new array contains collision checks between every pair of entity with another entities. We placed these collision checks inside Reactive.orList() function, which returns true if any of conditions in the list returns true.

Create another plane object on scene. Now we need to create an Entity instance and link it to this scene object. Add the following code below the lines where we get other plane objects:

```javascript
const plane2 = await new Entity("plane1", Reactive.point(0.1, 0.1, 0.1)).create();
```

Now let’s check the collision between plane0 and pair1, pair2. Add this code just after plane2 entity initialization:

```javascript
Diagnostics.watch("plane0 with others", checkArrayCollision3D(plane0, [plane1, plane2]));
```

The final script should look like this:

```javascript
const Scene = require("Scene");
const Reactive = require("Reactive");
const Diagnostics = require("Diagnostics");
 
function checkCollision(positionA, positionB, lengthA, lengthB) {
    return Reactive.abs(positionA.sub(positionB))
        .le(Reactive.add(lengthA.div(2), lengthB.div(2)));
}
 
function checkCollision3D(entityA, entityB) {
    return Reactive.andList([
        checkCollision(entityA.sceneObject.transform.x, entityB.sceneObject.transform.x, entityA.size.x, entityB.size.x),
        checkCollision(entityA.sceneObject.transform.y, entityB.sceneObject.transform.y, entityA.size.y, entityB.size.y),
        checkCollision(entityA.sceneObject.transform.z, entityB.sceneObject.transform.z, entityA.size.z, entityB.size.z)
    ]);
}
 
function checkArrayCollision3D(entityA, otherEntities) {
    return Reactive.orList(otherEntities
        .map(otherENtity => checkCollision3D(entityA, otherEntity))
    );
}
 
class Entity {
    constructor(name, size) {
        this.name = name;
        this.size = size;
    }
 
    async create() {
        this.sceneObject = await Scene.root.findFirst(this.name);
        return this;
    }
}
 
(async () => {
    const plane0 = await new Entity("plane0", Reactive.point(0.1, 0.1, 0.1)).create();
    const plane1 = await new Entity("plane1", Reactive.point(0.1, 0.1, 0.1)).create();
    const plane2 = await new Entity("plane2", Reactive.point(0.1, 0.1, 0.1)).create();
 
    Diagnostics.watch("plane0 with others", checkArrayCollision3D(plane0, [plane1, plane2]));
    Diagnostics.watch("plane0 with plane1", checkCollision3D(plane0, plane1));
})();
```

Save the file and return to Spark AR Studio. Try to move plane0 and look at the debug lines below. It displaces true when plane0 collides with any other plane. 


For example, this can be useful when creating gaming filters to check if a player collides with a set of enemies or collectible bonuses.

<h2>Doing action on collision</h2>
We will change the plane's color when they collide, and write a message to the console. To change plane color, we will swap it’s material.

Create two materials by clicking Add Asset - Material in the Assets window. We will use material0 when planes aren’t in collision, and material1 when they’re.

Change material1 color to a unique one, so we can distinguish planes in collision. Select material and use the properties window at the right.


Now select each plane scene object and add material0 to it. Plane should change its color.


Return to script. To access material assets, we have to import MaterialsModule. Add this code at the top of the script:

```javascript
const Materials = require("Materials");
```

Now let’s get reference to material0 and material1 assets. Copy this code below the lines when we’re creating Entities:

```javascript
    const material0 = await Materials.findFirst("material0");
    const material1 = await Materials.findFirst("material1");
```

We will be detecting collision between plane0 and plane1 and change the color of plane1 when they collide. Call the checkCollision3D(plane0, plane1) and then subscribe to the onOn() event of this reactive boolean. Copy the following code below materials variables declaration:

```javascript
    checkCollision3D(plane0, plane1).onOn().subscribe(() => {
        plane1.sceneObject.material = material1;
    });
```

Return to Spark AR and try to move plane0 around.


Now plane1 changes color when it starts to collide with plane0, but it doesn’t return it’s initial color, when the collision ends. Let’s fix this by subscribing to onOff() event of collision reactive boolean. We’ll change color back to the initial. Copy the following code below previous one:

```javascript
    checkCollision3D(plane0, plane1).onOf().subscribe(() => {
        plane1.sceneObject.material = material0;
    });
```

Try to move planes now.

Now plane1 changes color back when collision ends.
Visualizing collider bounding boxes
To make setting up collider sizes easier, we will visualize collider bounding boxes. We will attach a 3D cube model to each plane and apply semi-transparent material to this cube. Then we will be changing this cube’s scale as the size of the plane collider changes.

Drag cube.obg file to Assets window.


This model comes with default material, but it’s gray and opaque, which is not the best choice for collider visualization. Select this material.


At the right side of the Spark AR Studio, change material color to something bright. Change Blend Mode to Associative Alpha and set Opacity to 50%. This way material becomes semi-transparent.


Now drag the cube model inside every plane scene object, so cube becomes their child.


Return to code. We have to link collider size with cube child size. Add this line to Entity.create() method just after we get reference to entity’s scene object:

```javascript
(await this.sceneObject.findFirst("cube")).transform.scale = this.size;
```

Method Entity.create() should look like this:

```javascript
    async create() {
        this.sceneObject = await Scene.root.findFirst(this.name);
        (await this.sceneObject.findFirst("cube")).transform.scale = this.size;
        return this;
    }
```

Note that we call findFirst() method on entity sceneObject property. This findFirst() method will search objects only inside the object on which we called this method. So it will never return other scene objects named cube, because they are children of different planes. 

Save code and return to Spark AR. You should see that cubes don’t correspond to real collider sizes.


Cube model scale is too large. Select all Cube models inside each plane and change it’s scale, so the cubes are the same size as planes. This size value will be 0.5 for each axis.


Now we’ve visualized collider bounding boxes size. Try to play around with collider sizes to see if visualized cubes change.


Return to Spark AR Studio.


Collider visualizations changed their size. When you’re ready with setting up collider sizes, you can just disable Visible property of cube object.


