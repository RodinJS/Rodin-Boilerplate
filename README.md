# Rodin-Boilerplate
Rodin JS Boilerplate project for better understanding Library structure (__Rodin Starters Guide__)

### A basic VR experience in Rodin starts with the following two lines:

First you import the components of the library.

```javascript
import * as RODIN from 'rodin/core';
```

Then you start the rendering loop ( animationFrame loop ).

```javascript
RODIN.start();
```

`animationFrame` loops up to the frequency limited by the VRDisplay ( 60 fps for Mobiles and PCs, 90 fps for advanced VR Headsets, etc. )
>see [window.requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) for details


## SCENE

In Rodin you build your experience in scenes. You may have multiple scenes in your project, and switch from one to another whenever it’s needed.

After `RODIN.start()` call, Rodin creates a default scene and sets it as the `active` one.

The `animationFrame` loop will render the active scene only.

Whenever you add an object to `RODIN.Scene`, it is automatically added to the active scene:

```javascript
RODIN.Scene.add(object);
```

To __create__ other scenes just create a new object:

```javascript
let scene2 = new RODIN.Scene(“mySecondScene”);
```

To __add__ objects to non-active scene:

```javascript
scene2.add(object);
```

To __change__ the active scene use one of the following:

* by the scene name

```javascript
RODIN.Scene.go(“mySecondScene”); //not ready yet
```

* by the scene object

```javascript
RODIN.Scene.go(scene2);
```

* by the scene index (first scene has the index 0)

```javascript
RODIN.Scene.go(1);
```

The `Rodin.Scene` controls the rendering loop in its `render()` function, which is called on every animation frame.

If you want a custom function to be called on each cycle, you can add it to

```javascript
RODIN.Scene.preRender(myFunc)  //to be called before rendering the scene
```

  or
  
```javascript
RODIN.Scene.postRender(myFunc)  //to be called after rendering the scene
```
 

In the case above, the `myFunc` will be called on every frame render, disregarding which scene is active.

If you want the `myFunc` to be called only when the current scene is active, use the following:

```javascript
RODIN.Scene.active.preRender(myFunc) //to be called before rendering the scene
```

  or

```javascript
RODIN.Scene.active.postRender(myFunc) //to be called after rendering the scene
```

This way when you switch to another scene, `myFunc` will not be called.

Let’s look at a basic example of how you can create two different scenes, and switch between them by mouse click:

```javascript
import * as RODIN from 'rodin/core';
RODIN.start();

let scene2 = new RODIN.Scene("mySecondScene");

let currentSceneIndex = 0;

let counter = 0;

RODIN.Scene.preRender(() => {
    if (++counter % 60 === 0) {
        console.log(RODIN.Scene.active.name)
    }
});

document.addEventListener('click', function () {
    currentSceneIndex = (currentSceneIndex + 1) % 2;
    RODIN.Scene.go(currentSceneIndex);
});
```

In this *example* you can see that the first scene is created by running `RODIN.start()` function.

The second scene is created manually. And as we know that there are two scenes, indexed accordingly as 0 and 1, we can switch between them using those indexes (`currentSceneIndex`).

We also have created a pre-render function that prints the current scene name once per 60 frames.

In a browser rendering loops at 60fps max, so we will see current scene name printed every second, and as we click, the scene switches and the printed scene name changes. Here is the example 1 output from two clicks in 5 seconds:

```
Main
Main
mySecondScene
mySecondScene
Main
```
> see [Scene](https://doc.rodin.io/Scene.html) for details.


## SCULPT

Sculpt is essentially the 3D object container class.

As you might have noticed, Rodin library is based on [Three.js](https://threejs.org/) library (thanks [Ricardo Cabello](https://twitter.com/mrdoob) and co.), and in Three.js all objects are extensions of `THREE.Object3D`.

Sculpt class enhances the features of `THREE.Object3D` objects, by binding them to `RodinEvents` (see EVENTS for details).

As a constructor argument `Sculpt` must receive an object:


```javascript
{ 
	name: “myObject”, 
	//and 
	threeObject: a THREE.Object3D instance, 
	//or 
	sculpt: another Sculpt instance, 
	//or 
	url: url of an exported model file 
}
```

You can also provide at `THREE.Object3D`, a `Sculpt` instance or a url string directly, without putting them in an object as a key:value (see example below).

With events enabled, all Sculpt objects will be listening to fired `RodinEvents` and executing their corresponding event handlers if any.

Let’s see a basic example to make it clear:

```javascript
import * as RODIN from 'rodin/core';
RODIN.start();

let sphere = new RODIN.Sculpt(
    new THREE.Mesh(
        new THREE.SphereGeometry(1, 12, 12),
        new THREE.MeshBasicMaterial({ wireframe: true })
    )
);

RODIN.Scene.add(sphere);

sphere.on(RODIN.CONST.READY, function (event) {
    event.target.position = new THREE.Vector3(0, 1.6, -2);
});

sphere.on(RODIN.CONST.UPDATE, function (event) {
    event.target.rotation.y += RODIN.Time.delta / 1000;
});

sphere.on(RODIN.CONST.UPDATE, (event) => {
    event.target.rotation.z += RODIN.Time.delta / 10000;
});
```

In this example 2 we create Three.js sphere mesh object and pass it to RODIN.Sculpt() constructor.

Then we add the sphere to the current (active) scene, followed by three event listeners being added to it.

You can also gain access to the THREE.Object3D object that is wrapped by the Sculpt by a reference, for example : sphere._threeObject. Though this not recommended.

The first handler function goes to RODIN.CONST.READY event (see all event types listed in EVENTS chapter). Whenever the Sculpt object is created, it fires a READY event. This handler then sets the needed position to the event target (which is the sphere itself).

The second handler function goes to RODIN.CONST.UPDATE event. This event is fired for all Sculpt objects of active scene on each iteration of renderer during animationFrame loop, after preRender functions and before postRender functions.

This handler rotates the sphere by 1000th fraction of RODIN.Time.delta (the time elapsed after previous render call, e.g. 16.6ms for 60fps, 33.3ms for 30fps etc…).

In simpler words - this function is called on each rendering event, which generally occurs 60 times per second, and each time rotates the sphere by an angle that depends on how intense the calls are (60fps, 30fps, 90fps). The higher the fps rate, smaller the angle of each rotation.

This way we can make sure that the rotation speed won’t vary on different devices with different fps rates.

The third handler function again goes to RODIN.CONST.UPDATE event. This handler also rotates the sphere, but on a different axis and 10 times slower (try to understand why). This shows that Sculpt objects can have multiple handlers for each event and they all will be executed correspondingly.

Sculpt class provides public add() and remove() functions for adding or removing child objects. Besides, Sculpt has the .parent attribute you can set. By setting the .parent attribute, you remove the object from the current parent and add it to the target parent, while maintaining the object's global position, rotation, and scale (unlike add/remove).

> NOTE: Because of some restrictions that Three.js has, when changing parent, make sure the target parent or the parents of the target parent have a uniform scale (when object is scaled by the same value on all axes).

> see [Sculpt](https://doc.rodin.io/Sculpt.html) for details.


## EVENTS

There is a number of custom event types used in Rodin:

```javascript
READY = 'ready';
UPDATE = 'update';
START = 'start';
STOP = 'stop';
COMPLETE = 'complete';
GAMEPAD_HOVER = 'gamepadhover';
GAMEPAD_HOVER_OUT = 'gamepadhoverout';
GAMEPAD_BUTTON = 'gamepadbutton';
GAMEPAD_BUTTON_DOWN = 'gamepadbuttondown';
GAMEPAD_BUTTON_UP = 'gamepadbuttonup';
GAMEPAD_BUTTON_CHANGE = 'gamepadbuttonchange';
```

These constants are available through `RODIN.CONST.{EVENT}`. All those events can be used wrapped in RodinEvent class. Rodin has included this custom class because the controllers do not have the event concept natively, all they do is changing state and value, so with Rodin we’ve created a more JS developer-friendly interface to work with. You have seen the events usage in example 2. There are more samples available in our platform.

RodinEvent constructor requires a target object and parameters object:

```javascript
{
	type: 'event',
	domEvent: null,
	button: null,
	hand: '',
	controller: null
}
```
You can find several usages of events in examples below.


## ANIMATION

`AnimationClip` class is for creating animation clips on Sculpt objects. To do so we create a new instance of the class, provide the parameters that need to be animated (position, rotation, scale…), the duration in milliseconds and the target value of that parameters. Optionally we can provide the starting value as well, but by default the starting value is considered the parameter’s current value at the time the animation clip was started.

Let’s look at an example:

```javascript
import * as RODIN from 'rodin/core';
RODIN.start();

let sphere = new RODIN.Sphere();
let box = new RODIN.Box(.2, .2, .2, new THREE.MeshBasicMaterial({ wireframe: true, color: 0x996633 }));

sphere.on(RODIN.CONST.READY, function () {
    sphere.position.z = -2;
    sphere.position.y = 2;
    sphere.parent = box;
});

RODIN.Scene.add(new RODIN.Sculpt(new THREE.AmbientLight()));

let hoverAnimation = new RODIN.AnimationClip("hoverAnim", {
    scale: {
        x: {from: 1.0, to: 1.5},
        y: {from: 1.0, to: 1.5},
        z: {from: 1.0, to: 1.5}
    }
});
hoverAnimation.duration(200);

let hoverOutAnimation = new RODIN.AnimationClip("hoverOutAnim", {
    scale: {
        x: 1,
        y: 1,
        z: 1
    }
});
hoverOutAnimation.duration(200);

box.animation.add(hoverAnimation, hoverOutAnimation);

box.on(RODIN.CONST.GAMEPAD_HOVER, function () {
    if (box.animation.isPlaying('hoverOutAnim')) {
        box.animation.stop('hoverOutAnim', false);
    }
    box.animation.start('hoverAnim');
});

box.on(RODIN.CONST.GAMEPAD_HOVER_OUT, function () {
    if (box.animation.isPlaying('hoverAnim')) {
        box.animation.stop('hoverAnim', false);
    }
    box.animation.start('hoverOutAnim');
});

box.on(RODIN.CONST.READY, function () {
    box.position.set(1, 1.6, -2);
    RODIN.Scene.add(box);
});
```

Starting from the first lines we can see two new concepts - `new RODIN.Sphere();` and `new RODIN.Box();`. These are just time saving utilities in Rodin library, for creating simple Sculpt objects very fast.

After adding a new ambient light to the scene, we start creating the animations.

The first animation clip is the hoverAnimation. Constructor receives a name “hoverAnim”, and the parameters of the clip, that say “change the scale of the object from 1.0 to 1.5 for all three axes”. Please note that the clip has an assigned value to start from, this means that no matter what the state of the object is, the scale will be set to 1.0 at the beginning of the animation and end up being 1.5 when the animation is finished.

The duration of the animation clip is then set to 200ms.

The next animation clip is the hoverOutAnimation. Just like the hoverAnimation it changes the scale of the object. Note that this hoverOutAnimation does not assign any specific value to the property to start animating from, this means that whenever this animation starts it will consider the current value of the property as the starting point. So now, that we have the animation clips, let’s assign them to an object and start/stop them on some user interaction cases. Every Sculpt object has an animation member that is responsible for controlling assigned animation clips. We add the created clips to the box.animation, and set hover/hoverOut event handlers to start and stop the clips accordingly.


> see [AnimationClip](https://doc.rodin.io/AnimationClip.html) and [Animation](https://doc.rodin.io/Animation.html) for details.


## TIME

`RODIN.Time` is an extended integration of the native JS Date to our library. This class allows to set different time speeds for each scene you have and dynamically change it for advanced animation effects or calculations. In simple words, in Rodin we can change the time speed wherever and whenever we want.

This class also provides utilities like delta, that returns the elapsed time in milliseconds after latest tick (tick is called an every render).  

delta’s value is calculated according to the current time speed.

If you go back to the , you’ll notice that the sphere rotation depends on the RODIN.Time.delta, so if we change the speed of the scene by adding the following line:

```javascript
RODIN.Time.speed = 0.25;
```

we will basically slow down the sphere rotation speed on both axes by three fourths.

> NOTE: the rendering frequency does not depend on RODIN.Time.speed, so slowing down the speed won’t affect your FPS.


## GAMEPAD

Rodin provides access to the majority of current VR Headset gamepads (a.k.a. Controllers), including HTC Vive controllers, Oculus Touch controllers, cardboard button, Mouse, Keyboard etc…

All the devices that help you to interact with the VR experience in Rodin is considered to be a gamepad device. This approach helps to unify the gamepads logic.

By default, right after `RODIN.start()` Rodin initializes several types of gamepads that are available in `RODIN.GamePad` object. All controllers are enabled and available when a corresponding device is detected.

If you look at the example 3 for instance, you can see that no gamepad initialization is present in the code, however when running within HTC Vive, you’ll be able to see the default Vive gamepads being tracked and interacting with the box when hovered in and out. Same works for mouse, Oculus Rift and cardboard.

Of course you can make your custom gamepads for each device, and as long as you keep the Class structure, the custom gamepad will be fully functional. (We will leave this now, and go into details coming releases, sorry :)

Let’s make another example with gamepad interaction:

```javascript
import * as RODIN from 'rodin/core';
RODIN.start();

RODIN.Scene.add(new RODIN.Sculpt(new THREE.AmbientLight()));

let hoverAnimation = new RODIN.AnimationClip("hoverAnim", {scale: {x: 1.2, y: 1.2, z: 1.2}});
hoverAnimation.duration(100);

let hoverOutAnimation = new RODIN.AnimationClip("hoverOutAnim", {scale: {x: 1, y: 1, z: 1}});
hoverOutAnimation.duration(100);

for (let i = 0; i < 40; i++) {
    let box = new RODIN.Box(.2, .2, .2, new THREE.MeshNormalMaterial({wireframe: true, color: 0x996633}));
    box.animation.add(hoverAnimation, hoverOutAnimation);
    box.on(RODIN.CONST.READY, onReady);
    box.on(RODIN.CONST.GAMEPAD_HOVER, hover);
    box.on(RODIN.CONST.GAMEPAD_HOVER_OUT, hoverOut);
}

function onReady(evt) {
    evt.target.position.set(Math.random() * 4 - 2, Math.random() * 4 - 0.4, Math.random() * 4 - 2);
    RODIN.Scene.add(evt.target);
}
function hover(evt) {
    if (evt.target.animation.isPlaying('hoverOutAnim')) {
        evt.target.animation.stop('hoverOutAnim', false);
    }
    evt.target.animation.start('hoverAnim');
}
function hoverOut(evt) {
    if (evt.target.animation.isPlaying('hoverAnim')) {
        evt.target.animation.stop('hoverAnim', false);
    }
    evt.target.animation.start('hoverOutAnim');
}
//example 4
```

That’s it! We’ve just created 40 randomly positioned boxes with hover and hover out actions.

Now let’s move forward and make this experience more interactive – add a drag and drop feature, where all those objects can be moved by a gamepad.

Let’s also talk about the Raycasting. Raycasting is a process of retrieving the objects that are intersected with a particular line/vector in the space. This way we can know which objects are targeted by a vive controller line, by the normal vector of the camera, or by the cursor of the mouse.

While getting intersections from a line/vector, it is important to know and/or define how many objects are Raycasted by that line. In some cases, we’ll need all the objects, and in some cases maybe only the first object or two. For that purpose Rodin gives you the option of defining the number of returned intersections by setting raycastLayers value on any gamepad.

As we have mentioned before, the `RODIN.start()` prepares the initial scene and initializes all available gamepads. Those gamepads are available in RODIN.GamePad which ofcourse can be replaced by your custom ones.

For the next example we will use only mouse and Vive controllers, as all other controllers have behavior similar to the one or the other:

```javascript
import * as RODIN from 'rodin/core';
RODIN.start();
const pickedItems = [];
const elements = [];
const types = [RODIN.Sphere, RODIN.Box];
RODIN.GamePad.mouse.raycastLayers = 2;

const mouseToWorld = function mouseToWorld(obj) {
    if (!obj.drag_mouseGamepad) return null;
    const intersection = new THREE.Vector3();
    obj.drag_mouseGamepad.raycaster.ray.intersectPlane(obj.drag_plane, intersection);
    return intersection;
};

const elementReady = function (evt) {
    const obj = evt.target;
    obj.position.x = Math.random() * 20 - 10;
    obj.position.y = Math.random() * 20 - 10;
    obj.position.z = Math.random() * 20 - 10;
    obj.parent = RODIN.Scene.active;
};

const buttonDown = function (evt) {
    const obj = evt.target;
    if (evt.gamepad.navigatorGamePadId === 'mouse') {
        navigator.mouseGamePad.stopPropagationOnMouseMove = true;
        obj.drag_mouseGamepad = evt.gamepad;
        obj.drag_plane = new THREE.Plane();
        obj.drag_plane.setFromNormalAndCoplanarPoint(RODIN.Scene.activeCamera.getWorldDirection(), obj.globalPosition);
        obj.drag_mouseOriginalPosition = mouseToWorld(obj);
        obj.drag_draggedObjectOriginalPosition = obj.globalPosition.clone();
        obj.dragging = true;
    } else {
        if (obj.oldParent) return;
        obj.oldParent = obj.parent;
        obj.parent = evt.gamepad.sculpt;
    }
    pickedItems.push(obj);
};

const buttonUp = function (evt) {
    const obj = evt.target;
    if (evt.gamepad.navigatorGamePadId === 'mouse') {
        navigator.mouseGamePad.stopPropagationOnMouseMove = false;
        obj.dragging = false;
    } else if (obj.oldParent) {
        obj.parent = obj.oldParent;
        delete obj.oldParent;
    }
    const index = pickedItems.indexOf(evt.target);
    if (index > -1) {
        pickedItems.splice(index, 1);
    }
};

const update = function (evt) {
    const obj = evt.target;
    if (!obj.dragging) return;
    const mousePos = mouseToWorld(obj);
    obj.globalPosition = new THREE.Vector3(
        obj.drag_draggedObjectOriginalPosition.x - obj.drag_mouseOriginalPosition.x + mousePos.x,
        obj.drag_draggedObjectOriginalPosition.y - obj.drag_mouseOriginalPosition.y + mousePos.y,
        obj.drag_draggedObjectOriginalPosition.z - obj.drag_mouseOriginalPosition.z + mousePos.z
    );
};

for (let i = 0; i < 40; i++) {
    elements.push(new types[parseInt(Math.random() + 0.5)](.7, .7, .7, new THREE.MeshNormalMaterial()));
    elements[i].on(RODIN.CONST.READY, elementReady);
    elements[i].on(RODIN.CONST.GAMEPAD_BUTTON_DOWN, buttonDown);
    elements[i].on(RODIN.CONST.GAMEPAD_BUTTON_UP, buttonUp);
    elements[i].on(RODIN.CONST.UPDATE, update);
}

RODIN.Scene.active.on(RODIN.CONST.GAMEPAD_BUTTON_UP, (evt) => {
    while (pickedItems.length) {
        evt.target = pickedItems[0];
        pickedItems[0].emit(RODIN.CONST.GAMEPAD_BUTTON_UP, evt);
    }
});
//example 5
```

In the beginning of this example we have changed the raycastLayers value of the RODIN.GamePad.mouse gamepad, so that it will use only first two objects of the intersections.

The first function we define is the mouseToWorld function, which returns coordinates on an invisible plane, where a vector of the mouse cursor intersects with that plane. Imagine the cursor has an infinite line going into the screen strictly perpendicular to it, so that it appears as a dot (at the cursor point) when projected on the screen. This vector is a reverse projection of the 2D position of the cursor into the 3D dimensions of the scene.

When this line intersects an object at a certain point, this object creates an invisible infinite plane running right through that point of the object and angled parallel to the screen/camera perspective.

mouseToWorld function finds the position of the intersection with that plane.

Look at the buttonDown function. When the gamepad that triggered this event is a mouse gamepad, we link this gamepad to the intersected object, then we create the mentioned plane, and save the first coordinates of intersection as a starting point as well as the initial position of the object in the scene. Then we set the dragging flag to true and let the controller’s update function to take care of the rest.

During the update we track the changes of the intersection coordinates with the plane, and reposition the object according to the changed coordinates.

Think of this as if the object followed the point of cursor-plane intersection, like the cat follows the laser pointer on the floor.

On buttonUp event we stop the following by setting the dragging flag to false.

For HTC Vive the algorithm is much easier, as we don’t need to create any invisible planes. Because there is a physical object in the scene to represent the gamepad, we just change the intersected object’s parent to the controller’s sculpt object, and change back to the initial parent when the button is un-pressed.

Within just 80 lines of code, we have a fully functional drag and drop experience, working with mouse and HTC Vive controllers.

Try to enable the cardboard drag and drop feature, using the Camera normal to Raycast.

> see [GamePad](https://doc.rodin.io/GamePad.html) for details.


## ELEMENT

Even though VR experience is often perceived as “all 3D”, it is also about simulating reality, and the reality is that we use 2D elements in our 3D world very often (writing on a paper, viewing on a screen, posters, photos, walls…). And it is important to be able to easily create 2D elements in VR as well. For conventional 2D web developers `HTML` solves this perfectly (well …).

Trying to solve this problem in Rodin we introduce the experimental `Element` class which creates customized 2D planes.

The following is the pattern of parameters object that can be passed to the `Element` class constructor:

```javascript
{
	name: string,
	width: number,
	height: number,
	background: {
		opacity: number,
		color: hex,
		image: { url: string }
	},
	border: {
		radius: number,
		color: hex,
		width: number,
		opacity: number
	},
	label: {
		text: string,
		position: { v: number, h: number },
		fontFamily: string,
		fontSize: number,
		opacity: number,
		color: Hex
	},
	image: {
		url: string,
		position: { v: number, h: number },
		width: number,
		height: number,
		opacity: number
	},
	transparent: boolean,
	ppm: number
}
```

The `width` and `height` parameters are in meters, and the positions are the vertical and horizontal offset percentage from top left corner, opacity ranges from 0.0 to 1.0.

All elements are by default considered transparent even if the opacities are set to 1.0. If you want to make it non-transparent, just set the transparent value false in the params.

Let’s look at an example:

```javascript
RODIN.start();

let elParams = {name: "sample", width: 0.5, height: 0.5, ppm: 1000};
elParams.background = {
    color: 0xFFFFFF,
    opacity: 0.3
};

elParams.border = {
    radius: 0.1,
    width: 0.01,
    color: 0xFFFFFF,
    opacity: 0.6
};

elParams.image = {
    url: "img/rodin.png",
    width: 0.2,
    height: 0.3,
    position: {h: 50, v: 40}
};

elParams.label = {
    text: "RODIN",
    position: {v: 80, h: 50},
    fontFamily: "Arial",
    fontSize: 0.08,
    opacity: 0.8,
    color: 0xFFFFFF
};

let element = new RODIN.Element(elParams);
element.on(RODIN.CONST.READY, function (evt) {
    evt.target.parent = RODIN.Scene.active;
    evt.target.position.set(0, 1.6, -1);
});
//example 6
```

The result is:

![](https://rodin.space/images/tutorial/image002.png)


Unfortunately there is no way of placing multiple images or labels on a single element for now, but this feature along with many other enhancements will be added in the coming releases.

Except Element class Rodin provides also Text and DynamicText (wrappable text) classes that allow you to create flat text planes very similar to the Element.

> see [Element](https://doc.rodin.io/Element.html) for details.









