This is a tutorial on how to create a simple jukebox in Decentraland.

Full source code is available below and on [GitHub](https://github.com/hardlydifficult/DecentralandJukebox).

<hr>
<br>

##  Setting Up the Environment 

Install [Node.js](https://nodejs.org/en/download/) and [Python](https://www.python.org/downloads/) if you don't already have them installed. 

Then run:

```
npm install -g decentraland
```

This command is only run once per machine.

##  Create a Project

```
dcl init
```

Start with a 'basic' scene for this tutorial.

#### `dcl`

`dcl` is the command for the 'decentraland' SDK.

#### `init`

`dcl init` will ask a series of questions to configure a project template you can then build from.


## Start the Game

```
dcl start
```

This should open a new tab automatically to [http://localhost:8000](http://localhost:8000)


##  Add Assets

Add the art and sounds for our app to the project's directory.

Download the [model and music](https://github.com/hardlydifficult/DecentralandJukebox/raw/master/Jukebox.zip) or use your own of course.

## Add Jukebox to the scene

Modify `scene.tsx`, removing everything between the scene tags and then adding a glfx model:

```typescript
<scene>
  <gltf-model
    src="art/Jukebox.gltf">
  </gltf-model>
</scene>
```

##  Position and Scale Jukebox

Adjust the position and scale until it looks good.

```typescript
<gltf-model
  src="art/Jukebox.gltf"
  position={{x: 5, y: 0, z: 9.5}}
  scale={.6}>
</gltf-model>
```

#### `scale`

The scale can either be a single number as shown here or a Vector3 (similiar to the position).  Use a Vector3 to scale the model unevenly (e.g. in order to stretch the height).

#### `position`

The position entered is relative to the tile's origin.

## Define the songs to be included

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

```typescript
<entity ...>
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
export default class SampleScene extends DCL.ScriptableScene 
{
  songs...;

  state: {buttonState: boolean[], lastSelectedState: number} = {
    buttonState: Array(this.songs.length).fill(false),
    lastSelectedState: 0,
  };
```

```Array(this.songs.length).fill(false)```

This creates an array to track state for each song.  `.fill(false)` sets the default state for each button to be unpressed.

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
    <cylinder
      ...
      position={{x: 0, y: 0, z: buttonZ}} 
      transition={{position: {duration: 100}}} />
      <text ... />
  </entity>
)
```

Note you could change the buttonState manually at this point to test.

#### `transition`

Transitions can be used in Decentraland to animate from one state to another.  This works for position, rotation, scale, and color.  For more info on the [`TransitionValue`](https://docs.decentraland.org/sdk-reference/entity-interfaces/).

##  Push buttons

Subscribe to the `click` event for each button.  When clicked, update the button state.  The previously clicked button (if any) should automatically unclick when another is selected.

```typescript
sceneDidMount() 
{
  for(let i = 0; i < this.songs.length; i++)
  {
    this.eventSubscriber.on(`song${i}_click`, () => 
    {
      if(i != this.state.lastSelectedState)
      {
        this.state.buttonState[this.state.lastSelectedState] = false;
      }
      this.state.buttonState[i] = !this.state.buttonState[i];
      this.state.lastSelectedState = i;
      this.forceUpdate();
    });
  }
}
```

#### `sceneDidMount`

`sceneDidMount` is an event which is called after the scene has loaded.  We use this for any initialization required, such as subscribing to object events.

#### `this.eventSubscriber.on(`song${i}_click`, () =>`

The `eventSubscriber` allows you to respond to object specific events, in this example when the user clicks on the object.  `song${i}` is the id of the object and `click` is the event type.

#### `this.forceUpdate();`

`forceUpdate()` will cause the scene to re-render.

## Play the Selected Song

Add `sound` to the entity for each song to play when that button is selected.

```typescript
<entity ...
  sound={{ 
    src:song.src, 
    playing:this.state.buttonState[index]}}>
```

# 




<hr>

<br>

That’s it!  You now know the basics and with a bit of time should be able to create some interesting interactive experiences in Decentraland.  Hope this was helpful and let us know if you have questions.  

Some possible next steps:
 - Maybe see pay to open to make a pay to play

<br>

<hr>

<h1>Source Code</h1>

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

  state: {buttonState: boolean[], lastSelectedState: number} = { 
    buttonState: Array(this.songs.length).fill(false),
    lastSelectedState: 0,
  };

  sceneDidMount() 
  {
    for(let i = 0; i < this.songs.length; i++)
    {
      this.eventSubscriber.on(`song${i}_click`, () => 
      {
        if(i != this.state.lastSelectedState)
        {
          this.state.buttonState[this.state.lastSelectedState] = false;
        }
        this.state.buttonState[i] = !this.state.buttonState[i];
        this.state.lastSelectedState = i;
        this.forceUpdate();
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