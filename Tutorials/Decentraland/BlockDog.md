
This is a tutorial on creating a basic **AI in Decentraland**. We'll add a dog which follows you around and listens to some of your commands.

Full source code is available below and on [GitHub](https://github.com/hardlydifficult/BlockDog).

If you are new to Decentraland development, you may want to start with our [beginner tutorial, creating a Jukebox](https://steemit.com/tutorial/@hardlydifficult/decentraland-tutorial-creating-a-music-jukebox).

<hr>

##  Setting Up the Environment 

One time setup:

 - [Node.js](https://nodejs.org/en/download/) 
 - [Python](https://www.python.org/downloads/)

```
npm install -g decentraland
```

With a cmd prompt in the project's directory, run:

```
dcl init
```

Start with a `basic` scene for this tutorial.  This will populate the directory with a few files for us to start from.  We require `scene.tsx` for this tutorial which only comes with the `basic` and `interactive` templates.

Now start the game:

```
dcl start
```

This should open a new tab automatically to [http://localhost:8000](http://localhost:8000)


##  Add Assets

Add the art and sounds for our app to the project's directory.

Download the [dog's model](https://github.com/hardlydifficult/BlockDog/raw/master/art.zip) we've created or use your own of course.

## Add the models to the Scene

Modify `scene.tsx`, removing everything between the `scene` tags and then adding gltf models:

```typescript
<scene>
  <gltf-model 
    id="Dog"
    src="art/BlockDog.gltf"
  />
  <gltf-model
    id="Bowl"
    src="art/BlockDogBowl.gltf"
  />
</scene>
```

#### `gltf-model`

gltf is the model format Decentraland uses.  It's supported by Blender by installing [a glTF exporter](https://github.com/KhronosGroup/glTF-Blender-Exporter), [see Decentralands Scene Content Guide](https://docs.decentraland.org/sdk-reference/scene-content-guide/) for more information.

## Add Babylonjs

We will be using [Babylonjs](https://www.babylonjs.com/) for their Vector3 and Quaternion support.  

```
npm install babylonjs
```

```typescript
import {Vector3, Quaternion} from "babylonjs"; 
```

Note: The Vector3Component that Decentraland includes by default is just data.  To ease coding, we'll use Babylon's instead which includes methods such as `.subtract` and `.normalize`.  You could use another library for this, or code the math yourself.


## Add `state` for position and rotation

Above the `SampleScene` class, add an interface for state:

```typescript
export interface IState 
{
  characterPosition: Vector3, 
  bowlPosition: Vector3, 
  dogPosition: Vector3, 
  dogRotation: Quaternion,
}
```

Add `IState` to the `SampleScene` class definition:

```typescript
export default class SampleScene extends DCL.ScriptableScene<any, IState>
```

Then inside the class, add the state object itself:

```typescript
state = { 
  characterPosition: new Vector3(0, 0, 0), 
  bowlPosition: new Vector3(1, 0, 1), 
  dogPosition: new Vector3(9, 0, 9), 
  dogRotation: new Quaternion(0, 0, 0, 1), 
};
```

And finally leverage this state in the `render` method:

```typescript
<gltf-model...
  position={this.state.dogPosition} 
  rotation={this.state.dogRotation.toEulerAngles().scale(180 / Math.PI)} 
/>
<gltf-model...
  position={this.state.bowlPosition} 
/>
```

#### `export interface IState`

This is declaring the type of information that will be stored by the state object.  This declaration is optional, but a best practice and will allow your IDE to display better hints.

#### `Vector3` 

For more information see [Babylonjs's doc on Vector3](https://doc.babylonjs.com/api/classes/babylon.vector3).

#### `Quaternion` 

Quaternion is the preferred format used by games for storing and modifying rotation.  For more information see [Babylonjs's doc on Quaternion](https://doc.babylonjs.com/api/classes/babylon.quaternion).

To better understand Quaternion and why that format is preferred over Euler or other representations, see [our tutorial on Quaternions](https://youtu.be/kYOtk5a6_x4).


#### `<any, IState>`

This format is for [Generics in Typescript](https://www.typescriptlang.org/docs/handbook/generics.html).  Generics allow you to write more general purpose code and then have specific types slotted in after.  In this example we are allowing our base class, `DCL.ScriptableScene`, to know exactly what state will be stored.

#### `.toEulerAngles().scale(180 / Math.PI)`

Decentraland is expecting rotations in Eulers, using degrees.  This is converting our Quaternion to that desired format.














##  Position and Scale Jukebox

Adjust the position and scale until it looks good.

```typescript
<gltf-model ...
  position={{x: 5, y: 0, z: 9.5}}
  scale={.6}>
</gltf-model>
```


#### `position`

The position entered is relative to the tile's origin.

## Define the Songs to be Included

In the `SampleScene` class, add `songs`:

```typescript
export default class SampleScene extends DCL.ScriptableScene 
{
  songs: {src: string, name: string}[] = 
  [
    {src: "sounds/Concerto a' 4 Violini No 2 - Telemann.mp3", name: "Telemann"},
    {src: "sounds/Double Violin Concerto 1st Movement - J.S. Bach.mp3", name: "Bach"},
    {src: "sounds/Rhapsody No. 2 in G Minor – Brahms.mp3", name: "Brahms"},
    {src: "sounds/Scherzo No1_ Chopin.mp3", name: "Chopin"},
  ];
```

#### `{src: string, name: string}[]`

This is Typescript syntax for defining the type of information the `songs` variable will include.  Here we have an array of JSON data.  The JSON data includes a `src`, which is the path to the song's file, and a `name` which will appear in Decentraland.

## Add Buttons

Add a model for a button on the Jukebox for each song.  We'll make these interactive soon.

First add `{this.renderButtons()}` as a child of the model:

```typescript
  <gltf-model ...>
    {this.renderButtons()}
  </gltf-model>
```

Then create the `renderButtons()` method:

```typescript
renderButtons() 
{
  return this.songs.map((song, index) =>
  {
    let x = index % 2 == 0 ? -.4 : .1;
    let y = Math.floor(index / 2) == 0 ? 1.9 : 1.77;
    return (
      <entity
          position={{x, y, z: -.7}}>
        <cylinder
          id={"song" + index}
          rotation={{x: 90, y: 0, z: 0}}
          scale={{x: .05, y: .2, z: .05}}
          color="#990000" />
      </entity>
    )
  });
}
```

#### `this.songs.map((song, index) =>`

`.map` will call the following method for each item in the array.

#### `x` and `y`

We are positioning the buttons automatically based on their index.  You may need to make adjustments if you have a different jukebox model or if you have more/less songs.  Alternatively you could position each explicitly by defining the button's position in the `songs` JSON data.

#### `entity`

The `entity` here acts as a parent object to assist with positioning.  We will be adding a text label for the name of the song below. This approach allows the button and the text to be positioned together, and still allow the button to move when pressed without also moving the text.

#### `rotation`

Rotation here is in Euler form, defined in degrees.

##  Add Text for Each Song

Inside the `entity` for each song, add a `text` label displaying the song's `name`.

```typescript
  <cylinder ... />
  <text 
    hAlign="left"
    value={this.songs[index].name} 
    position={{x: .26, y: 0, z: -.1}}
    scale={.4} />
</entity>
```

## Add State for the Buttons

In the `SampleScene` class, add `state`:

```typescript
songs: ...;

state: {buttonState: boolean[], lastSelectedButton: number} = {
  buttonState: Array(this.songs.length).fill(false),
  lastSelectedButton: 0,
};
```

```Array(this.songs.length).fill(false)```

This creates an array to track state for each song.  `.fill(false)` sets the default state for each button to be unpressed, without this the default would be `undefined`.

## Position Buttons when Pressed

Set the `position` for the button's `cylinder` when pressed, and use a `transition` to animate it.

```typescript
let buttonZ = 0;
if(this.state.buttonState[index]) 
{
  buttonZ = .1;
}
return (
  <entity ...>
    <cylinder ...
      position={{x: 0, y: 0, z: buttonZ}} 
      transition={{position: {duration: 100}}} />
      <text ...
```

Note you could change the buttonState manually at this point to test.

#### `transition`

Transitions can be used in Decentraland to animate from one state to another.  This works for position, rotation, scale, and color.  See [Scene State in the Decentraland docs](https://docs.decentraland.org/sdk-reference/scene-state/) for more information.  

##  Push buttons

Subscribe to the `click` event for each button.  When clicked, update the button state.  The previously clicked button (if any) should automatically unclick when another is selected.

```typescript
sceneDidMount() 
{
  for(let i = 0; i < this.songs.length; i++)
  {
    this.eventSubscriber.on(`song${i}_click`, () => 
    {
        let newButtonState = this.state.buttonState.slice();
        if(i != this.state.lastSelectedButton)
        {
          newButtonState[this.state.lastSelectedButton] = false;
        }
        newButtonState[i] = !this.state.buttonState[i];
        this.setState({
          buttonState: newButtonState,
          lastSelectedButton: i
        });
    });
  }
}
```

#### `sceneDidMount`

`sceneDidMount` is an event which is called after the scene has loaded.  We use this for any initialization required, such as subscribing to object events.

#### `this.eventSubscriber.on('song${i}_click', () =>`

The `eventSubscriber` allows you to respond to object specific events, in this example when the user clicks on the object.  `song${i}` is the id of the object and `click` is the event type.

#### `.slice()`

We start by copying the current `buttonState` array.  This allows us to make changes without potentially causing unintended consequences. 

#### `this.setState`

Use `setState` to modify any of the variables stored by the `state` variable.  When using this method, render is called automatically, without any need to call forceUpdate (which would be less efficient than only rendering changes).   See [Scene State in the Decentraland docs](https://docs.decentraland.org/sdk-reference/scene-state/) for more information.  


## Play the Selected Song

Add `sound` to the entity for each song to play when that button is selected.

```typescript
<entity ...
  sound={{ 
    src:song.src, 
    playing:this.state.buttonState[index]}}>
```

<hr>

<br>

That’s it!  You now know the basics and with a bit of time should be able to create some interesting interactive experiences in Decentraland.  Hope this was helpful and let us know if you have questions.  

Some possible next steps:
 - Maybe charge for playing songs, see the [pay to open sample](https://github.com/decentraland/sample-scene-payments).
 - Maybe add multiplayer support, so people may listen to music together.  See the [multiplayer sample](https://github.com/decentraland/sample-scene-server).
 - Maybe add a dance floor, thumping speakers, and a bar.

<br>

<hr>

<h1>Source Code</h1>

**`scene.tsx`**:

```typescript
import * as DCL from 'metaverse-api'

export default class SampleScene extends DCL.ScriptableScene 
{
  songs: {src: string, name: string}[] = 
  [
    {src: "sounds/Concerto a' 4 Violini No 2 - Telemann.mp3", name: "Telemann"},
    {src: "sounds/Double Violin Concerto 1st Movement - J.S. Bach.mp3", name: "Bach"},
    {src: "sounds/Rhapsody No. 2 in G Minor – Brahms.mp3", name: "Brahms"},
    {src: "sounds/Scherzo No1_ Chopin.mp3", name: "Chopin"},
  ];

  state: {buttonState: boolean[], lastSelectedButton: number} = { 
    buttonState: Array(this.songs.length).fill(false),
    lastSelectedButton: 0,
  };

  sceneDidMount() 
  {
    for(let i = 0; i < this.songs.length; i++)
    {
      this.eventSubscriber.on(`song${i}_click`, () => 
      {
        let newButtonState = this.state.buttonState.slice();
        if(i != this.state.lastSelectedButton)
        {
          newButtonState[this.state.lastSelectedButton] = false;
        }
        newButtonState[i] = !this.state.buttonState[i];
        this.setState({
          buttonState: newButtonState,
          lastSelectedButton: i
        });
      });
    }
  }

  renderButtons() 
  {
    return this.songs.map((song, index) =>
    {
      let x = index % 2 == 0 ? -.4 : .1;
      let y = Math.floor(index / 2) == 0 ? 1.9 : 1.77;
      let buttonZ = 0;
      if(this.state.buttonState[index]) 
      {
        buttonZ = .1;
      }
      return (
        <entity
            position={{x, y, z: -.7}}
            sound={{ 
              src:song.src, 
              playing:this.state.buttonState[index]}}>
          <cylinder
            id={"song" + index}
            position={{x: 0, y: 0, z: buttonZ}} 
            transition={{position: {duration: 100}}} 
            rotation={{x: 90, y: 0, z: 0}}
            scale={{x: .05, y: .2, z: .05}}
            color="#990000" />
            <text 
              hAlign="left"
              value={this.songs[index].name} 
              position={{x: .26, y: 0, z: -.1}}
              scale={.4}/>
        </entity>
      )
    });
  }

  async render() 
  {
    return (
      <scene>
        <gltf-model
          src="art/Jukebox.gltf"
          scale={.6}
          position={{x: 5, y: 0, z: 9.5}}>
          {this.renderButtons()}
        </gltf-model>
      </scene>
    )
  }
}
```