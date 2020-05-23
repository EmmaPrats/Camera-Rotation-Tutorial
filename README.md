# How to rotate the camera around an object in Unity3D

Excerpt: In this Unity3D tutorial you will learn a simple way to move the camera around an object in the scene while always looking at it. I shared all of the code, and I added a link to where you can download the full Unity project too.

If you have tried `RotateAround()` and it's not working for you, you have come to the right place.

You can jump to [the solution](#solution) at the end of the post, download, or follow the tutorial in [video](https://www.youtube.com/watch?v=rDJOilo4Xrg) format.

## Table of contents

1. [Detecting the drag](#step-1-detect-the-drag--user-input)
2. [Calculating the rotation](#step-2-calculate-the-rotation)
3. [Moving the camera](#step-3-move-the-camera)
4. [The solution](#solution)
5. [Extra credit: Modify the rotation speed](#extra-credit-modify-the-rotation-speed)

## We want to program the camera so that it rotates around an object in the scene, while looking at it, responding to player input

![Desired result](https://github.com/EmmaPrats/Camera-Rotation-Tutorial/blob/master/Tutorial%20Assets/goodrotation.gif)

As you can see, the camera is doing the **yaw** and **pitch** rotations.

_(Note: The method explained in this post can be applied to movement caused by any script too, it's not restricted to user input.)_

**We want that a drag from one side of the screen to the other rotates us 180º around the object. How do we do that?**

![Requirements]()

The screen (the _viewport_ rather) has a size of 1x1 in viewport coordinates.

Since a drag from left to right of the screen will have a distance of 1, we have just defined our relation:

**1 unit in viewport coordinates  < == >  180 degrees of camera rotation**

### Step 1: Detect the drag (= user input)

For this, I created an empty `GameObject` in the scene (I named it "**InputController**"), and added a script to it (that I created, and named "**CameraMovement**"). These are the steps:

1. Go to the Unity Editor.
2. In the Scene hierarcy, right click.
3. Create > Game Object > Emtpy. I named it "**InputController**".
4. In the **InputController**'s Inspector, click "Add Script".
5. Type "**CameraMovement**", hit enter. This created a "**CameraMovement.cs**" file in your Assets folder, which contains an empty `CameraMovement` class that extends `MonoBehaviour`.

_(This is the most basic setup working with an empty project. Of course you can arrange your files and classes however you prefer.)_

Open **CameraMovement.cs** with your editor, time to code!

```csharp
public class CameraMovement : MonoBehaviour
{
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            // Will be true only in the 1st frame in which it detects the mouse is down (or a tap is happening)
        }
        else if (Input.GetMouseButton(0))
        {
            // Will be true as long as the mouse is down or a touch is happening.
        }
    }
}
```

In Unity games, it is common practice to use the [`Update`](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html) function to detect player input. _(It is also good practice not to handle it in the `Update` function directly, but this is the smallest of projects and we're not going to over-engineer it.)_

We are using the functions [`Input.GetMouseButtonDown(0)`](https://docs.unity3d.com/ScriptReference/Input.GetMouseButtonDown.html) and [`Input.GetMouseButton(0)`](https://docs.unity3d.com/ScriptReference/Input.GetMouseButton.html) to detect whether the user clicked/tapped or dragged the pointer over the screen.

The [`Input`](https://docs.unity3d.com/ScriptReference/Input.html) class also has a convenient [`mousePosition`](https://docs.unity3d.com/ScriptReference/Input-mousePosition.html) field which gives us the position of the pointer (mouse or finger) in screen coordinates. We'll use it in the next step.

Now that we have the problem defined and the drag detected, we can move the camera.

### Step 2: Calculate the rotation

Every frame we will calculate how much the cursor/finger moved since last frame, and translate it into how many degrees the camera should rotate according to our previously defined relation:

**1 unit in viewport coordinates  < == >  180 degrees of camera rotation**

In order to do that, we need to store the previous position in a private field (`previousPosition`).

This is how the code looks:

```csharp
public class CameraMovement : MonoBehaviour
{
    private Vector3 previousPosition
    
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            previousPosition = cam.ScreenToViewportPoint(Input.mousePosition);
        }
        else if (Input.GetMouseButton(0))
        {
            Vector3 currentPosition = cam.ScreenToViewportPoint(Input.mousePosition);
            Vector3 direction = previousPosition - currentPosition;
            
            float rotationAroundYAxis = -direction.x * 180; // camera moves horizontally
            float rotationAroundXAxis = direction.y * 180; // camera moves vertically
            
            previousPosition = currentPosition;
        }
    }
}
```

On the first frame in which the mouse is down (or the finger touches the screen), we store that position as our `previousPosition`.

In each subsequent frames, we calculate `direction`, which is the difference between the current `mousePosition` and `previousPosition`.

`rotationAroundYAxis` and `rotationAroundXAxis` are the rotations we must add to the current rotation each frame. They are calculated using this very advanced mathematical formula:

(1 / 180º) = (direction / rotationAroundAxis)

**In UnityEngine, the `forward` vector is by convention (0, 0, 1). This is why we are working with the assumption that the camera starts looking "forward", and will rotate around the X and Y axes.**

At the end of the function, don't forget to store our `currentPosition` as our `previousPosition`!

**Okay, but how do we ACTUALLY MOVE THE CAMERA?**

### Step 3: Move the camera

UnityEngine's [`transform`](https://docs.unity3d.com/ScriptReference/Transform.html) class has a convenient function [`RotateAround(Vector3 point, Vector3 axis, float angleInDegrees)`](https://docs.unity3d.com/ScriptReference/Transform.RotateAround.html). Here you would use it like this:

```csharp
cam.transform.RotateAround(target.transform.position, new Vector3(1, 0, 0), rotationAroundXAxis);
cam.transform.RotateAround(target.transform.position, new Vector3(0, 1, 0), rotationAroundYAxis);
```

![Wrong camera movement](https://github.com/EmmaPrats/Camera-Rotation-Tutorial/blob/master/Tutorial%20Assets/badrotation.gif)

As you can see, it works, but it probably doesn’t yield the result you want:

**`RotateAround()` produces a weird movement that we can't control, awful UX, 0 stars, thumbs down.** _(This is due to the fact that when we rotate around 1 axis we are moving the other, and viceversa. `RotateAround()` works well when you only allow rotation around 1 axis.)_

**But don’t worry! There’s a way!**

We will use [`Rotate(Vector3 axis, float angleInDegrees, Space relativeTo)`](https://docs.unity3d.com/ScriptReference/Transform.Rotate.html) instead.

### Solution

![Desired result](https://github.com/EmmaPrats/Camera-Rotation-Tutorial/blob/master/Tutorial%20Assets/goodrotation.gif)

```csharp
public class CameraMovement : MonoBehaviour
{
    [SerializeField] private Camera cam;
    [SerializeField] private Transform target;
    [SerializeField] private float distanceToTarget = 10;
    
    private Vector3 previousPosition;
    
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            previousPosition = cam.ScreenToViewportPoint(Input.mousePosition);
        }
        else if (Input.GetMouseButton(0))
        {
            Vector3 newPosition = cam.ScreenToViewportPoint(Input.mousePosition);
            Vector3 direction = previousPosition - newPosition;
            
            float rotationAroundYAxis = -direction.x * 180; // camera moves horizontally
            float rotationAroundXAxis = direction.y * 180; // camera moves vertically
            
            cam.transform.position = target.position;
            
            cam.transform.Rotate(new Vector3(1, 0, 0), rotationAroundXAxis);
            cam.transform.Rotate(new Vector3(0, 1, 0), rotationAroundXAxis, Space.World); // <— This is what makes it work!
            
            cam.transform.Translate(new Vector3(0, 0, -distanceToTarget));
            
            previousPosition = newPosition;
        }
    }
}
```

#### Explanation

We created the following private serialized fields, that you must assign in the Unity Editor (simply drag the objects into their corresponding box):

* `cam` references the camera that you want to move.
* `target` is the `transform` component of the `gameObject` around which we want to rotate.
* `distanceToTarget` is the distance from the target that we want the camera to be.

**What makes this method work is the fact that we are rotating around the world's Y axis.**

By default, `Rotate()` and `RotateAround()` use the GameObject's local axes (in our case, the camera's). Since rotating around 1 axis messes up the other, we need to rotate around the world's axis instead of ours.

`Rotate()` simply rotates the object in its place. **When you rotate _around something_, you are also translating**. That's why we must call `cam.transform.Translate()`. The order in which you do it matters:

* If you `Rotate()` first and `Translate()` second, you are rotating your axes, and then moving "according" //TODO find a better word to them.
* If you `Translate()` first and `Rotate()` second, you are moving "according" to the world's axes, and then rotating around yourself in your new position.

![Rotation order explanation]()

So what we need to do for this application is:

**Each frame, we set the camera's position as the target's position. Then we apply the rotations. Finally, we translate the camera back `distanceToTarget` units in -Z**

![Rotation visualization]()

### Extra credit: Modify the rotation speed

We defined the rotation speed by the relation:

**1 unit in viewport coordinates  < == >  180 degrees of camera rotation**

While it is one that works very well, and is very intuitive from a UX perspective, you could make the argument that 180 is a hadcoded arbitrary number.

Therefore, we can turn the relationship into a private serialized field, so we can edit it in the inspector:

```csharp
[SerializeField] [Range(0, 360)] private int maxRotationInOneSwipe = 180;
```

And the code for calculating the camera rotation will look like this:

```csharp
float rotationAroundYAxis = -direction.x * maxRotationInOneSwipe; // camera moves horizontally
float rotationAroundXAxis = direction.y * maxRotationInOneSwipe; // camera moves vertically
```
