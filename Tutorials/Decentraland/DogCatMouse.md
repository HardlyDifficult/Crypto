# Dog, Cat, Mouse, Cheese.  Decentraland.

We'll be creating a scene in which a predator guards its home, attacking any prey in sight.  The prey is on a mission: sneak through the fence, get some cheese, and get out safely.

In this example, we will create a stack-based finite state machine (FSM) to manage AI for animals.

TODO TOC
TODO tabs to spaces
  
## Getting Started

We are starting with a scene and a collection of components already created.  For more information about components and how to get started with Decentraland, see one of our previous tutorials: 

 - [Music Jukebox](https://steemit.com/tutorial/@hardlydifficult/decentraland-tutorial-creating-a-music-jukebox)
    Beginners, start here.

 - [Block Dog](https://steemit.com/tutorial/@hardlydifficult/decentraland-tutorial-basic-ai-with-block-dog)
    The Block Dog tutorial shows a way of controlling motion in your scene that's very different from this tutorial.  You may want to consider both before deciding which approach may be best for your project.

 - [Tower Defense](https://steemit.com/tutorial/@hardlydifficult/decentraland-tutorial-a-simple-tower-defense-game)
    Here we create a basic game, introducing components and how you might start to scale up more complex scenes and interactions.

Decentraland also has a page showing a [collection of example scenes](https://docs.decentraland.org/examples/sample-scenes/) you could learn from.

### Download the Starting Scene

Get the scene and art from:

 - https://github.com/hardlydifficult/DogCatMouseCheese/archive/Static_Scene.zip

And extract it anywhere.

### Start Decentraland

Open a command prompt, navigate to the project's directory, and run:

```
dcl start
```

Note this assumes you have installed Decentraland's SDK.  If not, please refer to one of the other tutorials mentioned above to get started.

This will open a new browser tab with the scene.

**Test**: Walk around and take a look. Nice, right?  Thanks James (the artist).

### About the Starting Scene

Due to the size of the scene we are creating, we are kicking off the tutorial with a static scene and some basic logic.  Once you have completed a basic Decentraland tutorial or two, I hope most of the code included here will make sense.

 - All art will be rendered by a component (in the components directory). 
 - `scene.tsx` includes some default state and calls to render each of the components.
 - `ts/SharedProperties.ts`: includes common types.
 - `ts/MathHelper.ts`: includes basic Vector3 math to make writing logic a bit easier.
 - `ts/SceneHelper.ts`: includes positioning information for static scenery.

## Adding a Grid

For path finding and collision handling, we will be logically positioning objects into a grid in which each cell is 1m x 1m.

For simplicity, when an object moves, it jumps from cell to cell.  In the rendered component, we use a `transition` in order to animate that change in position. 

To check for collisions, we simply check if the target grid cell is already occupied.

The grid will also make integrating a-star pathfinding (later in this tutorial) easy.

### Create a Grid

Create `ts\Grid.ts`:

```typescript
import { Vector3Component } from "metaverse-api";
import { add } from "ts/MathHelper";

export namespace Grid
{
  const grid: boolean[][] = [];

  export function init(width: number, depth: number)
  {
    grid.length = 0;
    for (let x = 0; x < width; x++)
    {
      grid.push([]);
      for (let y = 0; y < depth; y++)
      {
        grid[x].push(false);
      }
    }
  }

  export function set(position: Vector3Component, canBeOccupiedAlready: boolean = false)
  {
    const x = Math.round(position.x);
    const z = Math.round(position.z);
    if (grid[x][z] && !canBeOccupiedAlready)
    {
      throw new Error("Grid cell is already set");
    }
    grid[x][z] = true;
  }

  export function clear(position: Vector3Component, canBeEmpty: boolean = false)
  {
    const x = Math.round(position.x);
    const z = Math.round(position.z);
    if (!grid[x][z] && !canBeEmpty)
    {
      throw new Error("Grid cell wasn't set");
    }
    grid[x][z] = false;
  }

  export function isAvailable(position: Vector3Component)
  {
    const x = Math.round(position.x);
    const z = Math.round(position.z);
    if (x < 0 || z < 0 || grid.length <= x || grid[x].length <= z)
    {
      return false;
    }
    return !grid[x][z];
  }

  export function randomPosition(border: number = 1, mustBeAvailable: boolean = true): Vector3Component
  {
    let position;
    do
    {
      position = {
        x: Math.random() * (grid.length - border * 2) + border,
        y: 0,
        z: Math.random() * (grid[0].length - border * 2) + border
      };
    } while (!isAvailable(position) && mustBeAvailable);

    return position;
  }

  export function getNeighbors(startingPosition: Vector3Component): Vector3Component[]
  {
    let neighbors: Vector3Component[] = [];

    for (const neighborDirection of [
      { x: 1, y: 0, z: 0 },
      { x: -1, y: 0, z: 0 },
      { x: 0, y: 0, z: 1 },
      { x: 0, y: 0, z: -1 },
      //If enabling diag, update the 'distance' above with a formula
      //{ x: 1, y: 0, z: 1 },
      //{ x: -1, y: 0, z: -1 },
      //{ x: -1, y: 0, z: 1 },
      //{ x: 1, y: 0, z: -1 },
    ])
    {
      let position = add(startingPosition, neighborDirection);
      if (!isAvailable(position))
      {
        continue;
      }
      neighbors.push(position);
    }

    return neighbors;
  }

  export function hasClearance(position: Vector3Component, range: number): boolean
  {
    if (!isAvailable(position))
    {
      return false;
    }
    const neighbors = getNeighbors(position);
    if (neighbors.length < 4)
    {
      return false;
    }

    if (range > 1)
    {
      for (const neighbor of neighbors)
      {
        if (!hasClearance(neighbor, range - 1))
        {
          return false;
        }
      }
    }

    return true;
  }
}
```

Note: you could change the precision of the grid by changing all the `Math.round(position.x)` like lines.  For example:

```
const gridCellX = Math.round(position.x * 2)
const gridCellY = Math.round(position.y * 2)
```

This would make each grid cell .5m x .5m (twice as precise).

### Initialize the Grid

Add the following to `sceneDidMount` in the `scene.tsx` file to initialize the grid:

```typescript
  sceneDidMount()
  {
    Grid.init(30, 30);
```

This will initialize the grid's arrays to the correct size for our world.

Note that you will also need to import the `Grid` for this to compile.  IDEs (such as VS Code) will present a hint, allowing you to auto-complete the missing reference (usually by pressing Ctrl+Space).  This will come up several times throughout this tutorial (and we will not be mentioning it explicitly each time).  The import for `Grid` will look like:

```typescript
import { Grid } from 'ts/Grid';
```

### Spawn Trees in Random Locations

Update `spawnTrees` in the `scene.tsx` file to create a number of trees in random locations:

```typescript
  spawnTrees()
  {
    let trees: ISceneryProps[] = [];
    const range = config.trees.max - config.trees.min;
    let counter = 0;
    for (let i = 0; i < Math.random() * range + config.trees.min; i++)
    {
      let position;
      do
      {
        position = Grid.randomPosition(2, true);
        if (counter++ > 500)
        { // Don't get stuck working too hard
          break;
        }
      } while (!Grid.hasClearance(position, 4));
      Grid.set(position);

      trees.push({
        position,
        rotation: { x: 0, y: Math.random() * 360, z: 0 },
        scale: { x: 1, y: Math.random() * .4 + 1, z: 1 }
      });
    }
    this.setState({ trees });
  }
```

**Test**: Each time you refresh the browser, there should be a new random layout of trees.  Note that trees may overlap scenery at the moment.

We are using a JSON config file to make adjusting certain settings easy.  You can modify `config.json` to change the number of trees spawned:

```json
"trees": {
  "min": 5,
  "max": 20
},
```

### Click the Exit to Start Over

To make testing easier, we'll add a click event which will restart the world.

Update `sceneDidMount` in `scene.tsx` to subscribe to the event:

```typescript
  sceneDidMount()
	{
		...
		this.eventSubscriber.on("Exit_click", e => this.onExitClick());
	}
```

Then add an event handler:

```typescript
  onExitClick()
  { 
    for (const tree of this.state.trees)
    {
      Grid.clear(tree.position);
    }
    this.spawnTrees();
  }
```


**Test**: Click on the exit mound (which is the dirt pile closer to the dog house). The trees should re-spawn with new random positions.

### Add Static Scenery to the Grid

The `spawnTrees` algorithm above includes a loop to select a position with clearance / free space around it.  For this to work, we'll need to register the position of each of our static scenery objects with the grid.

Update `ts/SceneHelper.ts` by adding the following method:

```typescript
  export function updateGridWithStaticScenery()
  {
    for (const fence of fenceProps)
    {
      Grid.set(fence.position, true);
      if (fence.rotation.y == 0 || fence.rotation.y == 180)
      {
        Grid.set(add(fence.position, { x: 1, y: 0, z: 0 }), true);
        Grid.set(add(fence.position, { x: -1, y: 0, z: 0 }), true);
      }
      else
      {
        Grid.set(add(fence.position, { x: 0, y: 0, z: 1 }), true);
        Grid.set(add(fence.position, { x: 0, y: 0, z: -1 }), true);
      }
    }
    for (const corner of fenceCornerProps)
    {
      Grid.set(corner.position, true);
      if (corner.rotation.y == 0 || corner.rotation.y == 180)
      {
        Grid.set(add(corner.position, { x: 1, y: 0, z: 0 }), true);
        Grid.set(add(corner.position, { x: -1, y: 0, z: 0 }), true);
      }
      else
      {
        Grid.set(add(corner.position, { x: 0, y: 0, z: 1 }), true);
        Grid.set(add(corner.position, { x: 0, y: 0, z: -1 }), true);
      }
    }
    for (const spinner of fenceSpinnerProps)
    {
      Grid.clear(spinner.position, true);
    }
    for (let x = -1; x <= 1; x++)
    {
      for (let z = -1; z <= 1; z++)
      {
        if (x == 0 && z == 0 || z == -1 && x == 0)
        {
          continue;
        }
        Grid.set(add(houseProps.position, { x, y: 0, z }), true);
      }
    }
    for (let x = 0; x < 2; x++)
    {
      for (let z = -2; z <= 2; z++)
      {
        if (x == 0 && z == 0)
        {
          continue;
        }
        Grid.set(add(exitProps.position, { x, y: 0, z }), true);
      }
    }
    for (let x = -1; x <= 0; x++)
    {
      for (let z = -1; z <= 1; z++)
      {
        if (x == 0 && z == 0)
        {
          continue;
        }
        Grid.set(add(entranceProps.position, { x, y: 0, z }), true);
      }
    }
  }
```

Then in `scene.tsx`, update `sceneDidMount` to update the grid (note there are two new lines here):

```typescript
  sceneDidMount()
	{
		Grid.init(30, 30);
		SceneHelper.updateGridWithStaticScenery();
    Grid.set(this.state.baitProps.position);
		this.spawnTrees();
		this.eventSubscriber.on("Exit_click", e => this.onExitClick());
	}
```

**Test**: Click the exit mound several times and confirm the trees are never overlapping scenery.

### Render Grid for Debugging

As we add more experiences, we'll need a better way to confirm that the grid is configured correctly.  Let's add a `renderGrid` method to `scene.tsx`:

```typescript
  renderGrid()
  {
    let trees: ISceneryProps[] = [];
    for (let x = 0; x < 30; x++)
    {
      for (let z = 0; z < 30; z++)
      {
        let position = { x, y: 0, z };
        if (Grid.isAvailable(position))
        {
          continue;
        }

        trees.push({
          position,
          rotation: { x: 0, y: Math.random() * 360, z: 0 },
          scale: { x: 1, y: Math.random() * .4 + 1, z: 1 }
        });
      }
    }
    this.setState({ trees });
  }
```

Call it from `sceneDidMount`, after `spawnTrees`:

```typescript
  sceneDidMount()
  {
    ...
    //this.spawnTrees();
    this.renderGrid(); // For debugging
    this.eventSubscriber.on("Exit_click", e => this.onExitClick());
  }
```

Note: commenting out `spawnTrees` is optional.

**Test**: There should be a tree rendered on top of each fence post as well as on other scenery in the world.  

Animals will only be able to walk where there is no tree (i.e., the grid cell is not occupied).  So it's important that there are gaps in the fence, for example, so they can navigate through.

Turn off `renderGrid`, but remember this for debugging when you need it:

```typescript
  sceneDidMount()
  {
    ...
    this.spawnTrees();
    //this.renderGrid(); // For debugging
    ...
```

## Animals

There will be two types of animals in the scene, and we have 3 models to choose from.  Model selection is driven from our `config.json` file.  It's intended to allow you to change the scene from a cat chasing a mouse to a dog chasing a cat.  It also shows how separating components from logic allow for more flexibility - the cat here can either play the role of a predator, or a prey.

### Spawn a Predator (Dog or Cat)

The predator starts at the dog house and then will patrol the area, looking for prey. 

Add an event to `sceneDidMount` for when the user clicks on the `House`:

```typescript
  sceneDidMount()
  {
    ...
    this.eventSubscriber.on("House_click", e => this.onHouseClick());
  }
```

Add the following method to respond to the click event by spawning a predator:

```typescript
  onHouseClick()
  { // Spawn predator
    this.spawnAnimal(
      config.predator.animalType,
      SceneHelper.houseProps.position,
      add(SceneHelper.houseProps.position, { x: 0, y: 0, z: -1 }),
      config.predator.patrolSpeed);
  }
```

Add a helper method for spawning animals, which we will use again for the prey:

```typescript
  spawnAnimal(animalKey: keyof typeof AnimalType,
    position: Vector3Component,
    lookAtPosition: Vector3Component,
    moveDuration: number): IAnimalProps | null
  {
    if (!Grid.isAvailable(position))
    { // Space is occupied, can't spawn
      return null;
    }
    Grid.set(position);

    const animal: IAnimalProps = {
      id: "Animal" + this.objectCounter++,
      animalType: AnimalType[animalKey],
      position,
      lookAtPosition,
      moveDuration,
      animationWeights: [
        { animation: AnimationType.Idle, weight: 1 },
        { animation: AnimationType.Walk, weight: 0 },
        { animation: AnimationType.Drink, weight: 0 },
        { animation: AnimationType.Dead, weight: 0 },
        { animation: AnimationType.Run, weight: 0 },
        { animation: AnimationType.Sit, weight: 0 },
      ],
      isDead: false,
      scale: 1,
    };
    this.setState({ animals: [...this.state.animals, animal] });

    return animal;
  }
```

**Test**: Click on the dog house to see a cat spawn.  Modify the `config.json` to see a `Dog` spawn instead.

### Spawn the Prey (Cat or Mouse)

Add an event to `sceneDidMount` for when the user clicks on the `Entrance`:

```typescript
  sceneDidMount()
  {
    ...
    this.eventSubscriber.on("Entrance_click", e => this.onEntranceClick());
  }
```

Add the following method to respond to the click event by spawning prey:

```typescript
  onEntranceClick()
  { // Spawn prey
    this.spawnAnimal(
      config.prey.animalType,
      SceneHelper.entranceProps.position,
      add(SceneHelper.entranceProps.position, { x: 1, y: 0, z: 0 }),
      config.prey.sneakSpeed);
  }
```

**Test**: Click on the entrance mound... and it will look like nothing happened.  The mouse spawns in the dirt mound, but we cannot see it.  Modify the `config.json` to see a `Cat` spawn instead and you'll see a head poking through:

```json
  "prey": {
		"animalType": "Cat",
```

Switch back to `Mouse` when your done testing.

Note that clicking on the exit mound no longer fully resets the scene.

## Event Manager

We are creating an `EventManager` namespace to make it easy to interface with the `eventSubscriber` found in `scene.tsx`.  

### Create the Event Manager

Create an `ts\EventManager.ts` file with the following:

```typescript
import { EventSubscriber } from "metaverse-api";

export namespace EventManager
{
  let eventSubscriber: EventSubscriber;

  export function init(_eventSubscriber: EventSubscriber)
  {
    eventSubscriber = _eventSubscriber;
  }

  export function emit(eventType: string, ...params: any[])
  {
    eventSubscriber.emit(eventType, ...params);
  }
}
```

Update `sceneDidMount` in `scene.tsx` with:

```typescript
  sceneDidMount()
  {
    EventManager.init(this.eventSubscriber);
    ...
  }
```

### Render Animals on Event

When one of the display properties for an animal changes, such as the position, we will fire a `renderAnimals` event.  Update `scene.tsx` to re-render the animals when this event occurs:

```typescript
  sceneDidMount()
  {
    ...
    this.eventSubscriber.on('renderAnimals', e => this.onRenderAnimals());
  }
```

Add a method to respond to the event:

```typescript
  onRenderAnimals()
  {
    this.setState({ animals: this.state.animals });
  }
```

## State Machine

We'll be creating a stack-based finite state machine to handle the AI for the animals.  This means that for each animal, there is a single state which currently defines its behavior.  That state may add another state to the stack in order to achieve an interm goal, or pop itself once it achieves its own goal.

For the prey, we will be working towards the following scenario:

 - The prey spawns with a stack of Despawn, GoTo (exit), and Eat.
 - Eat is at the top of the stack, so that executes first.
 - In order to eat, the animal must be near the food.  If the food is out of reach, it adds a GoTo state.
 - GoTo paths to the cheese and animates the walk there.
 - Once there, GoTo pops itself off.
 - Eat kicks in again, this time the food is within range so it plays an eating animation and then pops itself off the stack.
 - GoTo (exit) is next on the stack.  That paths the animal to the exit.
 - Once at the exit, GoTo pops itself off and Despawn begins.
 - Despawn waits a second and then removes the animal from the scene.

### Create a Shared, Abstract State

Create `ts/StateMachine/AnimalState.ts`, which will be inherited by each of the states we implement below:

```typescript
import { AnimationType, IAnimalProps } from "ts/SharedProperties";
import { AnimalStateMachine } from "ts/StateMachine/AnimalStateMachine";
import { setTimeout, clearTimeout } from "timers";

export class AnimalState
{
  animalProps: IAnimalProps;
  animationTimeout?: NodeJS.Timer = undefined;

  constructor(animal: IAnimalProps)
  {
    this.animalProps = animal;
  }

  start(): void { }

  stop(): void
  {
    if (this.animationTimeout)
    {
      clearTimeout(this.animationTimeout);
    }
  }

  processMessage(message: string): boolean
  {
    return false;
  }

  animate(steps: { animation: AnimationType, for: number }[], then: () => void, stepNumber: number = 0)
  {
    if (stepNumber >= steps.length)
    {
      then();
      return;
    }

    AnimalStateMachine.changeAnimation(this.animalProps.id, steps[stepNumber].animation);
    this.animationTimeout = setTimeout(() =>
    {
      this.animate(steps, then, ++stepNumber);
    }, steps[stepNumber].for);
  }
}
```

### Create the State Machine

Create `ts/StateMachine/AnimalStateMachine.ts`, which manages the state machine for each individual animal:

```typescript
import { AnimationType, IAnimalProps } from "ts/SharedProperties";
import { setInterval, clearInterval } from "timers";
import { EventManager } from "ts/EventManager";
import { AnimalState } from "ts/StateMachine/AnimalState";

export namespace AnimalStateMachine
{
  interface AnimalStateObject
  {
    animalProps: IAnimalProps,
    stateStack: AnimalState[],
    animationInterval?: NodeJS.Timer 
  };
  const animalStates: AnimalStateObject[] = [];

  export function getAnimals(where: (a: AnimalStateObject) => boolean)
  {
    return animalStates.filter(where);
  }

  export function getAnimalProps(id: string): IAnimalProps | undefined
  {
    let state = animalStates.find(a => a.animalProps.id == id);
    if (state)
    {
      return state.animalProps;
    }

    return undefined;
  }

  export function pushState(state: AnimalState)
  {
    let animalState = animalStates.find(s => s.animalProps.id == state.animalProps.id);
    if (!animalState)
    {
      animalState = {
        animalProps: state.animalProps,
        stateStack: [state],
        animationInterval: undefined
      };
      animalStates.push(animalState);
    }
    else
    {
      const previousState = animalState.stateStack[animalState.stateStack.length - 1];
      if (previousState)
      {
        previousState.stop();
      }
      animalState.stateStack.push(state);
    }

    animalState.stateStack[animalState.stateStack.length - 1].start();
  }

  export function pushStates(states: AnimalState[])
  {
    let animalState = animalStates.find(s => s.animalProps.id == states[0].animalProps.id);
    if (!animalState)
    {
      animalState = {
        animalProps: states[0].animalProps,
        stateStack: states,
        animationInterval: undefined
      };
      animalStates.push(animalState);
    }
    else
    {
      const previousState = animalState.stateStack[animalState.stateStack.length - 1];
      if (previousState)
      {
        previousState.stop();
      }
      for (const state of states)
      {
        animalState.stateStack.push(state);
      }
    }

    animalState.stateStack[animalState.stateStack.length - 1].start();
  }

  export function popState(id: string)
  {
    const animalState = animalStates.find(s => s.animalProps.id == id);
    if (!animalState)
    {
      throw new Error("Animal not found");
    }
    if (animalState.stateStack.length <= 1)
    {
      throw new Error("You're popping everything!");
    }

    const previousState = animalState.stateStack.pop();
    if (previousState)
    {
      previousState.stop();
    }
    animalState.stateStack[animalState.stateStack.length - 1].start();
  }

  export function sendMessage(objectId: string, message: string)
  {
    const animalState = animalStates.find(s => s.animalProps.id == objectId);
    if (!animalState)
    {
      throw new Error("Animal not found");
    }
    while (!animalState.stateStack[animalState.stateStack.length - 1].processMessage(message))
    {
      const previousState = animalState.stateStack.pop();
      if(previousState)
      {
        previousState.stop();
      }
    }
    animalState.stateStack[animalState.stateStack.length - 1].start();
  }

  export function terminate(objectId: string)
  {
    const index = animalStates.findIndex(s => s.animalProps.id == objectId);
    if (index < 0)
    {
      throw new Error("Animal not found");
    }
    const animalState = animalStates[index];
    animalState.stateStack[animalState.stateStack.length - 1].stop();
    if (animalState.animationInterval)
    {
      clearInterval(animalState.animationInterval);
    }

    animalStates[index] = animalStates[animalStates.length - 1];
    animalStates.length--;
  }

  export function changeAnimation(id: string, animation: AnimationType)
  {
    const animalState = animalStates.find(s => s.animalProps.id == id);
    if (!animalState)
    {
      throw new Error("Animal not found");
    }
    if (animalState.animationInterval)
    {
      clearInterval(animalState.animationInterval);
    }
    const animationDeltaPerFrame = .25 / (1000 / 60);
    animalState.animationInterval = setInterval(() =>
    {
      let isDone = true;
      for (let animationWeight of animalState.animalProps.animationWeights)
      {
        if (animationWeight.animation == animation)
        {
          animationWeight.weight += animationDeltaPerFrame;
          if (animationWeight.weight >= 1)
          {
            animationWeight.weight = 1;
          }
          else
          {
            isDone = false;
          }
        }
        else
        {
          animationWeight.weight -= animationDeltaPerFrame;
          if (animationWeight.weight <= 0)
          {
            animationWeight.weight = 0;
          }
          else
          {
            isDone = false;
          }
        }
      }
      EventManager.emit("renderAnimals");

      if (isDone)
      {
        if (animalState.animationInterval)
        {
          clearInterval(animalState.animationInterval);
        }
      }
    }, 1000 / 60);
  }
}
```

### Create a State for Idling

Create `ts/StateMachine/StateIdle.ts`, a simple state which idles for a few moments and then pops itself off the stack:

```typescript
import { AnimalState } from "ts/StateMachine/AnimalState";
import { AnimalStateMachine } from "ts/StateMachine/AnimalStateMachine";
import { AnimationType, IAnimalProps } from "ts/SharedProperties";

export interface IStateIdleConfig
{
  minLength: number;
  maxLength: number;
  oddsOfSitting: number;
}

export class StateIdle extends AnimalState
{
  config: IStateIdleConfig;

  constructor(animal: IAnimalProps, config: IStateIdleConfig)
  {
    if (!config)
    {
      throw new Error("Missing config");
    }
    super(animal);
    this.config = config;
  }

  start()
  {
    const howLong = Math.random() * (this.config.maxLength - this.config.minLength) + this.config.minLength;
    let steps;
    if (Math.random() < this.config.oddsOfSitting)
    {
      steps = [
        { animation: AnimationType.Idle, for: 500 },
        { animation: AnimationType.Sit, for: Math.max(500, howLong - 1000) },
        { animation: AnimationType.Idle, for: 500 },
      ];
    }
    else
    {
      steps = [
        { animation: AnimationType.Idle, for: howLong },
      ];
    }
    this.animate(steps, () =>
    {
      AnimalStateMachine.popState(this.animalProps.id);
    });
  }

  stop()
  {
    super.stop();
  }
}
```

### Set the initial state to idle

Update `onEntranceClick` in `scene.tsx` to add `StateIdle` to prey that spawns in:

```typescript
  onEntranceClick()
  {
    const animalProps = this.spawnAnimal(
        config.prey.animalType,
        SceneHelper.entranceProps.position,
        add(SceneHelper.entranceProps.position, { x: 1, y: 0, z: 0 }),
        config.prey.sneakSpeed);
    if (animalProps)
    {
      AnimalStateMachine.pushState(
        new StateIdle(animalProps, config.prey.blockedConfig),
      );
    }
  }
```

**Test**: The scene will look the same as it did previously.  Spawn a prey and it idles for a period of time... and then an error is thrown (which you can see in the console).  The error will be thrown promptly. For testing you could modify `config.json` to increase the `minLength` \ `maxLength`.

## Path Finding

We are going to use an open-sounce implementation of a-star, which is an effecient way of finding the best path between points.

### Install A-Star

We are going to use a package for the a-star implementation.  You could, of course, implement your own as well.

In the command prompt, navigate to the project's directory and run:

```
npm install a-star
```

### Create an API for Pathing

Add a `calcPath` method to `Grid.ts`:

```typescript
const aStar = require('a-star');

export namespace Grid
{
  ...
  export function calcPath(startingPosition: Vector3Component, targetPosition: Vector3Component): Vector3Component[]
  {
    targetPosition = round(targetPosition);
    const results = aStar({
      start: round(startingPosition),
      isEnd: (n: Vector3Component): boolean =>
      {
        return inSphere(n, targetPosition, Grid.isAvailable(targetPosition) ? 0 : 1);
      },
      neighbor: (x: Vector3Component): Vector3Component[] =>
      {
        return Grid.getNeighbors(x);
      },
      distance: (a: Vector3Component, b: Vector3Component): number =>
      {
        return 1;
      },
      heuristic: (x: Vector3Component): number =>
      {
        return lengthSquared(subtract(x, targetPosition));
      },
      hash: (x: Vector3Component): string =>
      {
        return JSON.stringify(x);
      },
      timeout: 10
    });
    if (results.status == "success")
    {
      return results.path;
    }

    return [];
  }
}
```

### Create a State For Pathing

This state will take a target position, use a-star to find a path to that position, and then animate the walk there as well as handle and collisions that arrise.

Create `ts/StateMachine/StateGoTo.ts`:

```typescript
import { Vector3Component } from "metaverse-api";
import { AnimalState } from "ts/StateMachine/AnimalState";
import { IAnimalProps, AnimationType } from "ts/SharedProperties";
import { setInterval, clearInterval } from "timers";
import { subtract, isZero, add, div, equals, lengthSquared, mul } from "ts/MathHelper";
import { AnimalStateMachine } from "ts/StateMachine/AnimalStateMachine";
import { Grid } from "ts/Grid";
import { IStateIdleConfig, StateIdle } from "ts/StateMachine/StateIdle";

export interface IStateGoToConfig
{
  moveSpeed: number;
  panicSpeed?: number;
}

export class StateGoTo extends AnimalState
{
  target: {
    position: Vector3Component,
    isDead?: boolean
  };
  config: IStateGoToConfig;
  blockedConfig?: IStateIdleConfig;
  interval?: NodeJS.Timer = undefined;
  inPanic: boolean = false;

  constructor(animal: IAnimalProps, target: { position: Vector3Component, isDead?: boolean }, config: IStateGoToConfig, blockedConfig?: IStateIdleConfig)
  {
    super(animal);

    this.target = target;
    this.config = config;
    this.blockedConfig = blockedConfig;
  }

  start()
  {
    const speed = this.inPanic ? (this.config.panicSpeed || this.config.moveSpeed) : this.config.moveSpeed;
    this.animalProps.moveDuration = speed;
    const targetPosition = this.target.position;
    const path = Grid.calcPath(this.animalProps.position, targetPosition);
    if (this.target.isDead || path.length <= 0)
    {
      if (this.blockedConfig && !this.target.isDead)
      {
        return AnimalStateMachine.pushState(new StateIdle(this.animalProps, this.blockedConfig));
      }
      else
      {
        return AnimalStateMachine.popState(this.animalProps.id);
      }
    }

    if (path.length == 1)
    {
      return AnimalStateMachine.popState(this.animalProps.id);
    }

    let pathIndex = 1;
    if (this.interval)
    {
      clearInterval(this.interval);
    }
    this.interval = setInterval(() =>
    {
      let target = path[pathIndex];
      if (pathIndex < path.length - 1)
      { // Smooth diag movement
        target = add(target, path[pathIndex + 1]);
        target = div(target, 2);
      }
      try
      {
        this.walkTowards(target);
      }
      catch (e)
      {
        return this.repath();
      }
      pathIndex++;
      if (pathIndex >= path.length
        || this.target.isDead)
      {
        return AnimalStateMachine.popState(this.animalProps.id);
      }

      if (!equals(this.target.position, targetPosition))
      {
        return this.repath();
      }
    }, speed);
  }

  repath()
  {
    this.stop();
    this.start();
  }

  stop()
  {
    if (this.interval)
    {
      clearInterval(this.interval);
    }
  }

  walkTowards(targetPosition: Vector3Component)
  {
    const toTarget = subtract(targetPosition, this.animalProps.position);
    if (isZero(toTarget))
    { // Already there
      AnimalStateMachine.changeAnimation(this.animalProps.id, AnimationType.Idle);
      return;
    }

    Grid.clear(this.animalProps.position);
    if (!Grid.isAvailable(targetPosition))
    {
      Grid.set(this.animalProps.position);
      throw new Error("Space occupied, can't walk there.");
    }
    this.animalProps.position = targetPosition;
    Grid.set(this.animalProps.position);
    if (lengthSquared(toTarget) > .1)
    {
      // Look past the target
      this.animalProps.lookAtPosition = add(targetPosition, mul(toTarget, 10));
    }
    AnimalStateMachine.changeAnimation(this.animalProps.id, this.inPanic ? AnimationType.Run : AnimationType.Walk);
  }

  processMessage(message: string): boolean
  {
    if (message == "panic")
    {
      this.inPanic = true;
      this.repath();
      return true;
    }

    return super.processMessage(message);
  }
}
```

### Change the Prey to Use StateGoTo

Change the prey's initial state from `StateIdle` to `StateGoTo`:

```typescript
  onEntranceClick()
  {
    ...
    if (animalProps)
    {
      AnimalStateMachine.pushState(
        new StateGoTo(animalProps, SceneHelper.exitProps, config.prey.exitConfig, config.prey.blockedConfig),
      );
    }
  }
```

**Test**: When you spawn in prey, it will start to walk towards the exit, navigating around obstacles such as the fence.  Once they reach the exit, they start to line up (until we add the ability to despawn).


## Despawn 

When an animal is eaten or reaches the end, we need to despawn it, removing it from the scene and freeing up resources.  We'll be using events to communicate from the state machine (or anywhere in the application) to the front-end (`scene.tsx`) for removal.

### Remove Animal on Despawn

Add a new event subscription to `sceneDidMount`:

```typescript
this.eventSubscriber.on('despawn', (animalId, delay) => this.onDespawn(animalId, delay));
```

And an event handler:

```typescript
async onDespawn(animalId: string, delay: number)
{
  const animal = this.state.animals.find(a => a.id == animalId);
  if (animal)
  {
    AnimalStateMachine.terminate(animalId);
    animal.isDead = true;
    await sleep(delay);
    Grid.clear(animal.position);
    this.setState({ animals: this.state.animals.filter((a) => a.id != animal.id) });
  }
}
```

Update the `onExitClick` handler to despawn any existing animals when resetting the scene:

```typescript
onExitClick()
{ 
  for (const animal of this.state.animals.slice())
  {
    EventManager.emit("despawn", animal.id);
  }
  ...
}
```

**Test**: Spawn one or more prey in, click the exit, and confirm they despawn.  Note that this does not yet work for predators.

### Add a State to Despawn an Animal

Create `ts\StateMachine\StateDespawn.ts`:

```typescript
import { AnimalState } from "ts/StateMachine/AnimalState";
import { IAnimalProps, AnimationType } from "ts/SharedProperties";
import { EventManager } from "ts/EventManager";

export interface IStateDespawnConfig
{
  delay?: number;
}

export class StateDespawn extends AnimalState
{
  config: IStateDespawnConfig;
  timeout?: NodeJS.Timer = undefined;

  constructor(animal: IAnimalProps, config: IStateDespawnConfig)
  {
    super(animal);
    this.config = config;
  }

  start()
  {
    this.animate([{ animation: AnimationType.Dead, for: this.config.delay || 0 }], () => this.despawn());
  }

  despawn()
  {
    EventManager.emit("despawn", this.animalProps.id);
  }

  processMessage(message: string): boolean
  {
    return true;
  }
}
```

And update the prey to fallback in `scene.tsx` to despawn once goto completes:

```typescript
if (animalProps)
{
  AnimalStateMachine.pushStates([
    new StateDespawn(animalProps, {delay: 1000}),
    new StateGoTo(animalProps, SceneHelper.exitProps, config.prey.exitConfig, config.prey.blockedConfig),
  ]);
}
```

**Test**: Spawn in several prey. Once they get to the end, they should all despawn (previously they would have lined up at the exit).


## Eat

Create `ts\StateMachine\StateEat.ts`:

```typescript
import { AnimalState } from "ts/StateMachine/AnimalState";
import { AnimalStateMachine } from "ts/StateMachine/AnimalStateMachine";
import { AnimationType, IAnimalProps } from "ts/SharedProperties";
import { lengthSquared, subtract } from "ts/MathHelper";
import { StateGoTo, IStateGoToConfig } from "ts/StateMachine/StateGoTo";
import { EventManager } from "ts/EventManager";
import { Vector3Component } from "metaverse-api";
import { IStateIdleConfig } from "ts/StateMachine/StateIdle";
import { StateDespawn } from "ts/StateMachine/StateDespawn";

export interface IStateEatConfig
{
  eatRange: number;
  huntConfig: IStateGoToConfig;
}

interface IPrey
{
  id: string,
  position: Vector3Component,
  isDead?: boolean
}

export class StateEat extends AnimalState
{
  prey: IPrey;
  config: IStateEatConfig;
  blockedConfig?: IStateIdleConfig;

  constructor(animal: IAnimalProps, prey: IPrey, config: IStateEatConfig, blockedConfig?: IStateIdleConfig)
  {
    super(animal);
    this.prey = prey;
    this.config = config;
    this.blockedConfig = blockedConfig;

    if (!this.config.huntConfig)
    {
      throw new Error("Missing huntConfig");
    }
  }

  start()
  {
    if (this.prey.isDead)
    {
      AnimalStateMachine.popState(this.animalProps.id);
    }
    else if (lengthSquared(subtract(this.prey.position, this.animalProps.position)) <= this.config.eatRange * this.config.eatRange)
    {
      if (this.prey.isDead !== undefined)
      {
        let animal = AnimalStateMachine.getAnimalProps(this.prey.id);
        if (animal)
        {
          AnimalStateMachine.pushState(new StateDespawn(animal, { delay: 1000 }))
        }
      }
      else
      {
        EventManager.emit("captureBait", this.prey.id, 1000);
      }
      this.animalProps.lookAtPosition = this.prey.position;
      EventManager.emit("renderAnimals");
      this.animate([
        { animation: AnimationType.Drink, for: 1500 },
        { animation: AnimationType.Idle, for: 500 },
        { animation: AnimationType.Sit, for: 2000 },
        { animation: AnimationType.Idle, for: 500 },
      ], () =>
        {
          AnimalStateMachine.popState(this.animalProps.id);
        });
    }
    else
    {
      if (this.prey.isDead !== undefined)
      {
        AnimalStateMachine.sendMessage(this.prey.id, "panic");
      }
      AnimalStateMachine.pushState(new StateGoTo(this.animalProps, this.prey, this.config.huntConfig, this.blockedConfig));
    }
  }
}
```

Update the prey's initial state machine in `scene.tsx` to eat the cheese before exiting:

```typescript
AnimalStateMachine.pushStates([
  new StateDespawn(animalProps, {delay: 1000}),
  new StateGoTo(animalProps, SceneHelper.exitProps, config.prey.exitConfig, config.prey.blockedConfig),
  new StateEat(animalProps, this.state.baitProps, config.prey.eatConfig, config.prey.blockedConfig),
]);
```

**Test**: Spawn a mouse. It should go touch the cheese and then proceed to the exit.

## Patrol

Create `ts\StateMachine\StatePatrol.ts`:

```typescript
import { AnimalStateMachine } from "ts/StateMachine/AnimalStateMachine";
import { AnimalState } from "ts/StateMachine/AnimalState";
import { Vector3Component } from "metaverse-api";
import { IAnimalProps, AnimalType } from "ts/SharedProperties";
import { lengthSquared, subtract, inSphere } from "ts/MathHelper";
import { StateEat, IStateEatConfig } from "ts/StateMachine/StateEat";
import { StateIdle, IStateIdleConfig } from "ts/StateMachine/StateIdle";
import { StateGoTo, IStateGoToConfig } from "ts/StateMachine/StateGoTo";
import { Grid } from "ts/Grid";

interface IStatePatrolConfig
{
  eatConfig: IStateEatConfig,
  idleConfig: IStateIdleConfig,
  wanderConfig: IStateGoToConfig,

  minRadius: number,
  maxRadius: number,
  chanceOfMoving: number,

  preyType: AnimalType,
  scanRadius: number,
}

export class StatePatrol extends AnimalState
{
  config: IStatePatrolConfig;
  patrolAround: { position: Vector3Component };

  constructor(animal: IAnimalProps, patrolAround: { position: Vector3Component }, config: IStatePatrolConfig)
  {
    super(animal);
    this.config = config;
    this.patrolAround = patrolAround;

    if (!this.config.eatConfig)
    {
      throw new Error("Missing eatConfig")
    }
  }

  start()
  {
    let prey = this.lookForPrey()
    if (prey)
    { // Hunt
      AnimalStateMachine.pushState(new StateEat(this.animalProps, prey, this.config.eatConfig));
    }
    else if (Math.random() < this.config.chanceOfMoving)
    { // Move
      let targetPosition;
      do
      {
        targetPosition = Grid.randomPosition();
      } while (!inSphere(targetPosition, this.patrolAround.position, this.config.maxRadius)
        || inSphere(targetPosition, this.patrolAround.position, this.config.minRadius));

      AnimalStateMachine.pushState(new StateGoTo(this.animalProps, { position: targetPosition }, this.config.wanderConfig));
    }
    else
    { // Idle
      AnimalStateMachine.pushState(new StateIdle(this.animalProps, this.config.idleConfig))
    }
  }

  lookForPrey(): IAnimalProps | null
  {
    for (const prey of AnimalStateMachine.getAnimals((a) => a.animalProps.animalType == this.config.preyType && !a.animalProps.isDead))
    {
      const distanceSquared = lengthSquared(subtract(prey.animalProps.position, this.animalProps.position));
      if (distanceSquared <= this.config.scanRadius * this.config.scanRadius)
      {
        return prey.animalProps;
      }
    }

    return null;
  }

  processMessage(message: string): boolean
  { 
    return true;
  }
}
```

Update `onHouseClick` in `scene.tsx` to spawn in predators with `StatePatrol`:

```typescript
onHouseClick()
{ 
  const animalProps = this.spawnAnimal(
    config.predator.animalType,
    SceneHelper.houseProps.position,
    add(SceneHelper.houseProps.position, { x: 0, y: 0, z: -1 }),
    config.predator.patrolSpeed);
  if (animalProps)
  {
    AnimalStateMachine.pushState(new StatePatrol(
      animalProps,
      SceneHelper.houseProps,
      config.predator.patrolConfig
    ));
  }
}
```

**Test**: This one enabled a lot:
 - Clicking on the dog house will spawn a cat (which you can change to a `Dog` in the config).
 - The cat will wander around patroling the dog house.
 - If a mouse comes near a cat, it may be spotted.
 - Clicking on the exit to despawn / respawn the scene works again.


## Polish

Now we'll add a couple of visual effects to improve our scene.

### Capture bait

Add a new event to `sceneDidMount` in `scene.tsx`:

```typescript
this.eventSubscriber.on('captureBait', e => this.onCaptureBait());
```

Then add a method to handle the event:

```typescript
async onCaptureBait()
{
  await sleep(750);
  this.state.baitProps.isVisible = false;
  this.setState({ baitProps: this.state.baitProps });
  await sleep(2000);
  this.state.baitProps.isVisible = true;
  this.setState({ baitProps: this.state.baitProps });
}
```

This event is already included in `StateEat`.

**Test**: When a mouse eats the cheese, a few crumbs should disappear and then spawn in again a couple seconds later.


### Spinning Fence Door

When any animal walks under one of the broken fence segments, let's play an animation to make it spin.

First, let's emit an event from the Grid class allowing our program to react to grid changes:

```typescript
export function set(position: Vector3Component, canBeOccupiedAlready: boolean = false)
{
  ...
  EventManager.emit("gridCellSet", position);
}
```
Then register to the event in `sceneDidMount` of `scene.tsx`:

```typescript
this.eventSubscriber.on('gridCellSet', cell => this.onGridCellSet(cell));
```

And add a method to handle the event:

```typescript
async onGridCellSet(position: Vector3Component)
{
  let index = -1;
  for (let i = 0; i < SceneHelper.fenceSpinnerProps.length; i++)
  {
    if (approxEquals(position, SceneHelper.fenceSpinnerProps[i].position))
    {
      index = i;
      break;
    }
  }
  if (index >= 0)
  {
    let fenceSpinState = this.state.fenceSpinState.slice();
    if (fenceSpinState[index] != SpinState.None)
    { // One at a time to keep the animation timing
      return;
    }
    // Note this is not always correct..
    fenceSpinState[index] = index == 0 ? SpinState.Enter : SpinState.Exit; 
    this.setState({ fenceSpinState });
    await sleep(75 * 1000 / 25);
    fenceSpinState = this.state.fenceSpinState.slice();
    fenceSpinState[index] = SpinState.None;
    this.setState({ fenceSpinState });
  }
}
```

**Test**: Spawn in a few mice and watch them walk through the fence.  The same effect should work if a cat or dog travels that way, but this is harder to test as our current patrol settings keep them away from the fence.
