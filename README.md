# Overview

Easy to use on-screen thumbstick controls. Key features:

- Activates wherever you tap, and then follows the touch.
- Visual representation of sticks is optional and customizable.
- Has special support to not interfere with UGUI controls.
- Compatible with both legacy and new input systems.
- Flexible options on how to access thumbstick input.
- Options to fallback to mouse and keyboard.
- Distributed as package to not spam the Assets folder.


## Quick Start

Quickest way to add thumbstick controls is to locate a prefab ```Packages/Anywhere Pad/Prefabs/NonUI/SingleStick``` and drag it into your scene. After this you'll be able to use ```Stick.Single.Input```.

If you want two sticks, just use a different prefab - ```Packages/Anywhere Pad/Prefabs/NonUI/DualStickPad``` and use ```Stick.Left.Input``` / ```Stick.Right.Input``` to read inputs.


## Sticks vs. UI

If you have any UI, you will notice that sticks added this way will react to touch and mouse even when cursor is above UI controls. This behavior is usually unwanted. In this case you should use prefabs from the ```Packages/Anywhere Pad/Prefabs/UGUI``` folder. They are expected to be added as a child of UI Canvas. When stick component is located on a UI game object, it will react only to the input that hits this exact object.

There are three styles of UGUI sticks that are bundled with the package:
- **Default** look is white semi-transparent base with the white stick, with a slight shadow and small direction arrows on the base.
- **Simplified** look is the same but without shadow and without arrows.
- **Invisible** look is the instance of stick with no StickView at all.

So, if you want to visualize the sticks or want them to play nice with the other UI, you should prefer UI based stick option.


## Stick View

You can customize the look of the thumbstick in several ways. The simplest option is to grab a ```SimplifiedStickView``` prefab and change the textures/colors used for base and the knob. Then if you need extra tweaking you can go further with modification of your custom game objects with UguiStickView in the base.

If you want something more advanced, maybe 3D visualization of a stick or non-UGUI visualization, you can extend StickView, which is abstract and you will be expected to implement the following method:

```protected abstract void UpdateView(IStick stick, bool immediately);```

Lastly, if you don't want to be bound to StickView or even to MonoBehaviour, then you can always just read the stick input in any place of your code and use it how you need.


## Reading the Input

There are three main options how to read input from sticks:
- Explicitly referencing ```Stick``` component and reading from its ```Input``` property.
- Configuring stick to write to static interface using ```Static Property``` field in stick inspector, and then reading from the corresponding static property.
- Configuring stick to send it as ```New Input System Control``` value and bind this control to your game action.

> Note that **static interface is never null**, so it's safe to access ```Stick.Left.Input``` value even if you don't have any stick instances in scene. It will just have default zero values.


## Stick Public Interface

Once you have an instance of IStick (or its implementation - Stick), it will have the following public properties:
- **```bool IsTracking```** - bool that indicates if stick is now controlled by pointer input, i.e. if it's touched or dragged with the mouse.
- **```Vector2 Input```** - Vector2 indicating the direction and magnitude of stick offset from the base center. Magnitude is never more than one.
- **```float X```** - horizontal component of the input. Same as ```Input.x```
- **```float Y```** - vertical component of the input. Same as ```Input.y```
- **```Vector2 Direction```** - Normalized value of Input, when it's non-zero. If Input is zero, Direction will be zero too. Magnitude of direction is always either zero or one.
- **```bool HasDirection```** - bool that indicates if Direction is non-zero.
- **```bool TryGetDirection(out Vector2 direction)```** - convenience method to get both ```HasDirection``` and ```Direction``` in a single call.

> Note, that when stick is controlled from keyboard, ```Input``` will have non-zero values, while ```IsTracking``` will be ```false```.


## Editor Testing

By default, sticks are configured to always react to touch. And in editor, they are also set to use mouse and keyboard to control stick input. You'll find these settings in stick inspector, all input sources (```Touch```, ```Mouse```, ```Keyboard```) can have the following usage options: ```Always```, ```EditorOnly``` and ```Never```. For keyboard you can also choose keys to control the stick. Note that you can have mouse and keyboard input enabled in builds, but it's configuration is not very flexible and it's primary usage is editor testing.


## Sensitivity Settings

When stick decides what should be the magnitude of ```Input``` vector, it will measure the distance of stick from the drag origin (relative to screen size) and if it's less then ```threshold```, ```Input``` is considered to be zero. Then as you drag the stick further, Input value linearly grows to the length of one until offset from origin reaches the ```limit``` then Input stays clamped at the magnitude of one.

Screen size in this case is considered to be a single value, calculated based on width and height of the screen. You can choose how exactly it's calculated using the ```Reference Size``` option, with possible values: ```Width```, ```Height```, ```AverageDimension```, ```MaxDimension```, ```MinDimension```.

Lastly, there's a ```Bounds``` option, which limits the detection of initial touch to the given area of the screen. rect ranging from 0 to 1 in both directions is the whole screen. Note that this option affect UI based sticks too, but in most cases it's redundant to use custom Bounds value for UI based sticks.


## Advanced Use

You are not limited to one or two sticks. You can potentially have any number of sticks assigned to different areas of the screen. The only limitation is the access via static interface, which can address three sticks at most, so you'll need to reference the exceeding sticks explicitly or use new input system.


# Examples

Most basic example is ```Player Controller``` It uses two sticks to control a player in 3D space. Left stick is to move. Right stick is to look around. No UI involved. Sticks are invisible and referenced via static references.

Another example is ```Space Fighter``` It also uses two sticks. Left stick is to move space fighter on a plane. Right stick is to point and fire. Sticks are integrated into the UI to not conflict with other UI interactions. Space fighter reads input from explicit reference to the sticks.

Last example is a ```Snake```. It uses a single stick to control a snake in 2D space. Stick is a part of the UI and will be visualized when you drag the screen. It illustrates the third option of input access - snake itself is controlled by the Snake/Move action in the new input system. And thumbstick writes to the <Gamepad>/leftStick. Binding between Snake/Move and <Gamepad>/leftStick is done in the Input Actions window.

> Version of snake for the old input system is the same, but stick is configured to write input to the static interface and the snake reads the input from there.
