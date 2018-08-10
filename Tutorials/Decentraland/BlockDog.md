
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

Download the [models we've created](https://github.com/hardlydifficult/BlockDog/raw/master/art.zip) or use your own of course.

## Add the Models to the Scene

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

**Test**: You should see both the dog and his bowl.

#### `gltf-model`

gltf is the model format Decentraland uses.  It's supported by Blender by installing a [glTF exporter](https://github.com/KhronosGroup/glTF-Blender-Exporter), see [Decentralands Scene Content Guide](https://docs.decentraland.org/sdk-reference/scene-content-guide/) for more information.

## Add Babylonjs

We will be using [Babylonjs](https://www.babylonjs.com/) for their Vector3 and Quaternion support.  

With a cmd prompt in the project's directory, run:

```
npm install babylonjs
```

Then add the following to the top of `scene.tsx`:

```typescript
import {Vector3, Quaternion} from "babylonjs"; 
```

Note: The Vector3Component that Decentraland uses by default is just data.  To ease coding, we'll use Babylon's instead which includes functions such as `.subtract` and `.normalize`.  You could use another library for this or code the math yourself.


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

Then inside the class, add the state object itself including their default values:

```typescript
state = { 
  characterPosition: new Vector3(0, 0, 0), 
  bowlPosition: new Vector3(1, 0, 1), 
  dogPosition: new Vector3(9, 0, 9), 
  dogRotation: new Quaternion(0, 0, 0, 1), 
};
```

And finally leverage this state in the `render` function:

```typescript
<gltf-model...
  position={this.state.dogPosition} 
  rotation={this.state.dogRotation.toEulerAngles().scale(180 / Math.PI)} 
  transition={{
    rotation: {
      duration: 300
    }
  }}
/>
<gltf-model...
  position={this.state.bowlPosition} 
/>
```

Note: we'll be leveraging the `characterPosition` later in the tutorial.

**Test**: The dog is in one corner, the bowl in the other.

#### `export interface IState`

This is declaring the type of information that will be stored by the state object.  This declaration is optional, but a best practice and will allow your IDE to display better hints.

#### `Vector3` 

For more information see [Babylonjs's doc on Vector3](https://doc.babylonjs.com/api/classes/babylon.vector3).

#### `Quaternion` 

For more information see [Babylonjs's doc on Quaternion](https://doc.babylonjs.com/api/classes/babylon.quaternion).

Quaternion is the preferred format used by games for storing and modifying rotation. For more information on the Quaternion format, see [our tutorial on Quaternions](https://youtu.be/kYOtk5a6_x4).

#### `<any, IState>`

This format is for [Generics in Typescript](https://www.typescriptlang.org/docs/handbook/generics.html).  Generics allow you to write more general purpose code and then have specific types slotted in after.  In this example we are allowing our base class, `DCL.ScriptableScene`, to know exactly what state will be stored.

#### `.toEulerAngles().scale(180 / Math.PI)`

Decentraland is expecting rotations in Eulers, using degrees.  This is converting our Quaternion to that desired format.

#### `transition`

Transitions can be used in Decentraland to animate from one state to another.  This works for position, rotation, scale, and color.  See [Scene State in the Decentraland docs](https://docs.decentraland.org/sdk-reference/scene-state/) for more information.  

## Animate the Dog

There should always be an animation playing on the dog.  For this tutorial, we'll be supporting an idle animation, walking, and sitting.  Weights will be used in order to transition between them.  

To begin, add the animations to the dog's `gltf-model` and hardcode the weight for each:

```typescript
const animationWeights = {idle: 1, walk: 0, sit: 0};
return (
  <scene>
    <gltf-model... 
      skeletalAnimation={[
        { 
          clip: "Idle", 
          weight: animationWeights.idle,
        },
        { 
          clip: "Walking", 
          weight: animationWeights.walk,
        },
        { 
          clip: "Sitting", 
          weight: animationWeights.sit,
        },
      ]}
    />
```

**Test**: The dog should be wagging his tail. Change the hard coded weights to view the other animations, e.g. `{idle: 0, walk: 1, sit: 0}`

## Define Goals for the Dog

We'll be implementing the dog's AI as a state machine.  Define each of the possible states in an `enum`:

```typescript
enum Goal
{
  Idle,
  Sit,
  Follow,
  GoDrink,
  Drinking,
}
```

Then track the current goal, previous goal, and a weight to transition between them:

```typescript
export interface IState 
{
  ...
  dogGoal: Goal,
  dogPreviousGoal: Goal,
  dogAnimationWeight: number,
}

...

state = { 
  ...
  dogGoal: Goal.Idle,
  dogPreviousGoal: Goal.Idle,
  dogAnimationWeight: 1,
};
```

Now implement a function to calculate each animation's weight based on the dog's goals.

```typescript
getAnimationRates() : {idle: number, sit: number, walk: number} 
{
  const weight = Math.min(Math.max(this.state.dogAnimationWeight, 0), 1);
  const inverse = 1 - weight;

  let sit = 0;
  let walk = 0;
  
  switch(this.state.dogPreviousGoal)
  {
    case Goal.Sit:
      sit = inverse;
      break;
    case Goal.Follow:
    case Goal.GoDrink:
      walk = inverse;
      break;
  }

  switch(this.state.dogGoal)
  {
    case Goal.Sit:
      sit = weight;
      break;
    case Goal.Follow:
    case Goal.GoDrink:
      walk = weight;
      break;
  }

  return {idle: 1 - (sit + walk), sit, walk};
}
```

Then update `animationWeights` in `render` to use this function:

```typescript
const animationWeights = this.getAnimationRates();
```

**Test**: The dog should be wagging his tail exactly as he did before.  Try changing the `dogGoal` to change the animation.  If the `dogAnimationWeight` is less than 1, you should see the animation selected by `dogPreviousGoal` blending in.

#### `Math.min(Math.max(x, 0), 1)`

This clamps the value of x to be between 0 and 1.

#### `idle: 1 - (sit + walk)`

The total weight across all animations should equal 1.  By setting the idle weight to be the remainder, we are ensuring the dog is always moving a little.


## Sit on Command

Change the dog's goal to `Sit` when you click on him.  If he's already sitting, return to `Idle`.

Add an interval which fires each frame to increase the `dogAnimationWeight` smoothly.

```typescript
sceneDidMount()
{
  this.eventSubscriber.on("Dog_click", () =>
  {
    this.setDogGoal(this.state.dogGoal == Goal.Sit ? Goal.Idle : Goal.Sit);
  });
    
  setInterval(() => 
  {
    const weight = Math.min(Math.max(this.state.dogAnimationWeight, 0), 1);
    this.setState({dogAnimationWeight: weight + .01});
  }, 1000/60);
}

setDogGoal(goal: Goal)
{
  this.setState({
    dogGoal: goal,
    dogAnimationWeight: 1 - this.state.dogAnimationWeight,
    dogPreviousGoal: this.state.dogGoal
  });
}
```

**Test**: Clicking the dog will transition between sitting and standing.

#### `sceneDidMount`

`sceneDidMount` is an event which is called after the scene has loaded.  We use this for any initialization required, such as subscribing to object events.

#### `this.eventSubscriber.on(...`

The `eventSubscriber` allows you to respond to object specific events, in this example when the user clicks on the object. 

#### `this.setState`

Use `setState` to modify any of the variables stored by `state`.  When using this function, render is called automatically.   See [Decentraland's Scene State](https://docs.decentraland.org/sdk-reference/scene-state/).

## Go Drink Some Water

When you click on the bowl, the dog should walk over and have a drink.

Update the `sceneDidMount` function to add click event for the bowl. In the existing interval, add logic to `walkTowards` the target.

```typescript
this.eventSubscriber.on("Bowl_click", () =>
{
  this.setDogGoal(Goal.GoDrink);
});

setInterval(() => 
{
  ...

  switch(this.state.dogGoal)
  {
    case Goal.Follow:
    case Goal.GoDrink:
      const targetLocation = this.state.dogGoal == Goal.Follow ? this.state.characterPosition : this.state.bowlPosition;
      const delta = targetLocation.subtract(this.state.dogPosition);
      if(delta.lengthSquared() < 2) 
      {
        this.setDogGoal(this.state.dogGoal == Goal.Follow ? Goal.Sit : Goal.Drinking);
      }
      else
      {
        this.walkTowards(targetLocation);
      }
  }
}, 1000/60);
```

Create a function for `walkTowards` which smoothly updates the dog's position and set's his rotation to face the target.

```typescript
walkTowards(position: Vector3)
{
  let delta = position.subtract(this.state.dogPosition);
  delta = delta.normalize().scale(.015); // .015 is the walk speed
  delta.y = 0;
  const newPosition = this.state.dogPosition.add(delta);
  if(!isInBounds(newPosition)) 
  {
    this.setDogGoal(Goal.Idle);
  }
  else
  {
    this.setState({
      dogPosition: newPosition,
      dogRotation: lookAt(this.state.dogPosition, position, -Math.PI / 2)
    });
  }
}
```

Add the following functions to the bottom of the `scene.tsx` file:

```typescript
function isInBounds(position: Vector3): boolean
{
  return position.x > .5 && position.x < 9.5
    && position.z > .5 && position.z < 9.5;
}
```

Note: If you own multiple parcels of land, you may want to update the bounds used here.

Unfortunately there is not a `lookAt` implementation easily accessible.  So we copied the following from Babylon's [TransformNode](https://github.com/BabylonJS/Babylon.js/blob/master/src/Mesh/babylon.transformNode.ts):

```typescript
// Math from Babylon TransformNode
function lookAt(
  pos: Vector3,
  targetPoint: Vector3, 
  yawCor: number = 0, 
  pitchCor: number = 0, 
  rollCor: number = 0): Quaternion 
{
  const dv = targetPoint.subtract(pos);
  const yaw = -Math.atan2(dv.z, dv.x) - Math.PI / 2;
  const len = Math.sqrt(dv.x * dv.x + dv.z * dv.z);
  const pitch = Math.atan2(dv.y, len);
  return Quaternion.RotationYawPitchRoll(yaw + yawCor, pitch + pitchCor, rollCor);
}
```

**Test**: Click on the water bowl and the dog should walk over to it and then stop and stand in front of it.

#### `.lengthSquared()`

This checks the distance remaining to the target.  When calculating distance, the lengthSquared is first calculated and then a square root is taken to get the length.  Game developers often use lengthSquared where possible as a best practice anytime we do not need the specific length.

#### `normalize().scale(.015)`

By normalizing the delta position we get a direction.  Then the direction is scaled, providing the correct offset for a single frame of movement.

#### `-Math.PI / 2`

This is defining a rotation offset for the model.  Without this, in our example the dog would walk sideways.

## Follow the Character Around

The dog should randomly decide to follow the character from time to time.  When he catches up to the character, the dog sits (to ask for a treat).

Add the following to the `sceneDidMount` function to make the player's position available in `state` and to `considerGoals` periodically: 

```typescript
this.subscribeTo("positionChanged", (e) =>
{
  this.setState({characterPosition: new Vector3(e.position.x, e.position.y, e.position.z)});
}); 

setInterval(() =>
{
  if(this.state.dogAnimationWeight < 1)
  {
    return;
  }

  switch(this.state.dogGoal)
  {
    case Goal.Idle:
      this.considerGoals([
        {goal: Goal.Sit, odds: .1},
        {goal: Goal.Follow, odds: .9},
      ]);
    case Goal.Drinking:
      this.considerGoals([
        {goal: Goal.Sit, odds: .1},
      ]);
    case Goal.Follow:
      this.considerGoals([
        {goal: Goal.Idle, odds: .1},
      ]);
    case Goal.GoDrink:
    case Goal.Sit:
      this.considerGoals([
        {goal: Goal.Idle, odds: .1},
      ]);
  } 
}, 1500);
```

Note: We are using a second, slower interval here so the dog does not change his mind too quickly.

Then implement the `considerGoals` function:

```typescript
considerGoals(goals: {goal: Goal, odds: number}[])
{
  for(let i = 0; i < goals.length; i++)
  {
    if(Math.random() < goals[i].odds)
    {
      switch(goals[i].goal)
      {
        case Goal.Follow:
          if(!isInBounds(this.state.characterPosition))
          {
            continue;
          }
      }

      this.setDogGoal(goals[i].goal);
      return;
    }
  }
}
```

**Test**: When you are in bounds, the dog may randomly start following you.

#### `subscribeTo("positionChanged"...`

Decentraland communicates information by way of events whereever possible. See [Decentraland's Event Handling](https://docs.decentraland.org/sdk-reference/event-handling/).

#### `new Vector3`

Here we are creating a Babylonjs Vector3 from Decentraland's Vector3Component.

<hr>

<br>

That’s it!  Hope this was helpful and let us know if you have questions.  

Next steps:
 - The model has an animation called 'Drinking'. Add this animation to the code above.
 - Use slerp to adjust the rotation in code and then the dot product to scale the dog's movement speed. 
 - When the animation states change rapidly, it does not transition smoothly.  Can you fix this?  Maybe better consideration of previous states or by leveraging animation transitions defined by your modeler.
 - Add more puppies!

<br>

<hr>

<h1>Source Code</h1>

**`scene.tsx`**:

```typescript
import * as DCL from 'metaverse-api'
import {Vector3, Quaternion} from "babylonjs"; 

export interface IState 
{
  characterPosition: Vector3, 
  bowlPosition: Vector3, 
  dogPosition: Vector3, 
  dogRotation: Quaternion,
  dogGoal: Goal,
  dogPreviousGoal: Goal,
  dogAnimationWeight: number,
}

enum Goal
{
  Idle,
  Sit,
  Follow,
  GoDrink,
  Drinking,
}

export default class SampleScene extends DCL.ScriptableScene<any, IState>
{
  state = { 
    characterPosition: new Vector3(0, 0, 0), 
    bowlPosition: new Vector3(1, 0, 1), 
    dogPosition: new Vector3(9, 0, 9), 
    dogRotation: new Quaternion(0, 0, 0, 1), 
    dogGoal: Goal.Idle,
    dogPreviousGoal: Goal.Idle,
    dogAnimationWeight: 1,
  };

  getAnimationRates() : {idle: number, sit: number, walk: number} 
  {
    const weight = Math.min(Math.max(this.state.dogAnimationWeight, 0), 1);
    const inverse = 1 - weight;

    let sit = 0;
    let walk = 0;
    
    switch(this.state.dogPreviousGoal)
    {
      case Goal.Sit:
        sit = inverse;
        break;
      case Goal.Follow:
      case Goal.GoDrink:
        walk = inverse;
        break;
    }

    switch(this.state.dogGoal)
    {
      case Goal.Sit:
        sit = weight;
        break;
      case Goal.Follow:
      case Goal.GoDrink:
        walk = weight;
        break;
    }

    return {idle: 1 - (sit + walk), sit, walk};
  }

  sceneDidMount()
  {
    this.eventSubscriber.on("Dog_click", () =>
    {
      this.setDogGoal(this.state.dogGoal == Goal.Sit ? Goal.Idle : Goal.Sit);
    });

    this.eventSubscriber.on("Bowl_click", () =>
    {
      this.setDogGoal(Goal.GoDrink);
    });

    setInterval(() => 
    {
      const weight = Math.min(Math.max(this.state.dogAnimationWeight, 0), 1);
      this.setState({dogAnimationWeight: weight + .01});

      switch(this.state.dogGoal)
      {
        case Goal.Follow:
        case Goal.GoDrink:
          const targetLocation = this.state.dogGoal == Goal.Follow ? this.state.characterPosition : this.state.bowlPosition;
          const delta = targetLocation.subtract(this.state.dogPosition);
          if(delta.lengthSquared() < 2) 
          {
            this.setDogGoal(this.state.dogGoal == Goal.Follow ? Goal.Sit : Goal.Drinking);
          }
          else
          {
            this.walkTowards(targetLocation);
          }
      }
    }, 1000/60);

    this.subscribeTo("positionChanged", (e) =>
    {
      this.setState({characterPosition: new Vector3(e.position.x, e.position.y, e.position.z)});
    }); 

    setInterval(() =>
    {
      if(this.state.dogAnimationWeight < 1)
      {
        return;
      }
  
      switch(this.state.dogGoal)
      {
        case Goal.Idle:
          this.considerGoals([
            {goal: Goal.Sit, odds: .1},
            {goal: Goal.Follow, odds: .9},
          ]);
        case Goal.Drinking:
          this.considerGoals([
            {goal: Goal.Sit, odds: .1},
          ]);
        case Goal.Follow:
          this.considerGoals([
            {goal: Goal.Idle, odds: .1},
          ]);
        case Goal.GoDrink:
        case Goal.Sit:
          this.considerGoals([
            {goal: Goal.Idle, odds: .1},
          ]);
      } 
    }, 1500);
  }

  considerGoals(goals: {goal: Goal, odds: number}[])
  {
    for(let i = 0; i < goals.length; i++)
    {
      if(Math.random() < goals[i].odds)
      {
        switch(goals[i].goal)
        {
          case Goal.Follow:
            if(!isInBounds(this.state.characterPosition))
            {
              continue;
            }
        }

        this.setDogGoal(goals[i].goal);
        return;
      }
    }
  }

  setDogGoal(goal: Goal)
  {
    this.setState({
      dogGoal: goal,
      dogAnimationWeight: 1 - this.state.dogAnimationWeight,
      dogPreviousGoal: this.state.dogGoal
    });
  }

  walkTowards(position: Vector3)
  {
    let delta = position.subtract(this.state.dogPosition);
    delta = delta.normalize().scale(.015); // .015 is the walk speed
    delta.y = 0;
    const newPosition = this.state.dogPosition.add(delta);
    if(!isInBounds(newPosition)) 
    {
      this.setDogGoal(Goal.Idle);
    }
    else
    {
      this.setState({
        dogPosition: newPosition,
        dogRotation: lookAt(this.state.dogPosition, position, -Math.PI / 2)
      });
    }
  }

  async render() 
  {
    const animationWeights = this.getAnimationRates();
    return (
      <scene>
        <gltf-model 
          id="Dog"
          src="art/BlockDog.gltf"
          position={this.state.dogPosition} 
          rotation={this.state.dogRotation.toEulerAngles().scale(180 / Math.PI)} 
          transition={{
            rotation: {
              duration: 300
            }
          }}
          skeletalAnimation={[
            { 
              clip: "Idle", 
              weight: animationWeights.idle,
            },
            { 
              clip: "Walking", 
              weight: animationWeights.walk,
            },
            { 
              clip: "Sitting", 
              weight: animationWeights.sit,
            },
          ]}
        />
        <gltf-model
          id="Bowl"
          src="art/BlockDogBowl.gltf"
          position={this.state.bowlPosition} 
        />
      </scene>
    )
  }
}

function isInBounds(position: Vector3): boolean
{
  return position.x > .5 && position.x < 9.5
    && position.z > .5 && position.z < 9.5;
}

// Math from Babylon TransformNode
function lookAt(
  pos: Vector3,
  targetPoint: Vector3, 
  yawCor: number = 0, 
  pitchCor: number = 0, 
  rollCor: number = 0): Quaternion 
{
  const dv = targetPoint.subtract(pos);
  const yaw = -Math.atan2(dv.z, dv.x) - Math.PI / 2;
  const len = Math.sqrt(dv.x * dv.x + dv.z * dv.z);
  const pitch = Math.atan2(dv.y, len);
  return Quaternion.RotationYawPitchRoll(yaw + yawCor, pitch + pitchCor, rollCor);
}
```