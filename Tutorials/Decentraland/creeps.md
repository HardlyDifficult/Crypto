
This is a tutorial on creating a simple **Tower Defense Game in Decentraland**. Creeps are making their way through your base.  Stop as many as you can by springing traps at the right moment.  This is multiplayer, who will win: Humans or the Creeps?

Full source code is available on [GitHub](https://github.com/hardlydifficult/DecentralandCreeps).

If you are new to Decentraland development, you may want to start with our [beginner tutorial, creating a Jukebox](https://steemit.com/tutorial/@hardlydifficult/decentraland-tutorial-creating-a-music-jukebox).

This tutorial was sponsored by Decentraland.

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

 - **Parcels**: Select 4 parcels for this tutorial.  Any 2x2 plot is fine for testing locally, for example:

```
42,42; 43,42; 42,43; 43,43
```

- **Scene Template**: select `Remote`

For everything else, the defaults are fine.

In the `server\` directory, run:

```
npm install
npm install -G nodemon
```

Note: nodemon is optional, however we are using it to auto-refresh the server when a build happens.

Modify `server\package.json`:

```json
"scripts": {
  "build": "metaverse-compiler build.json",
  "watch": "metaverse-compiler build.json --watch",
  "start": "nodemon build/index.js"
},
```

## Start the Scene

You'll want three different command prompts for this.

In the first command prompt, navigate to the `server\` directory and run:

```
npm run watch
```

This will build your application.  If any files are modified, it will rebuild automatically.

In the second command prompt, also in the the `server\` directory, run:

```
npm start
```

This hosts your server for local testing, at ws://localhost:8087

And in the third prompt, navigate the the project directory and run:

```
dcl start
```

This starts the game and should open a new tab automatically to [http://localhost:8000](http://localhost:8000)

##  Add Assets

Add the art for the game to the project's root directory.

You can download the [models we've created](https://github.com/hardlydifficult/DecentralandCreeps/raw/master/assets.zip) or use your own of course.

## Add a Random Path

We'll generate a path which always starts from the same location and then travels randomly until it reaches the other side.

Modify the `state` variable in `server\state.ts` to add a `path`:

```typescript
import { Vector2Component } from 'metaverse-api'

let state: {
  path: Vector2Component[],
} = {
  path: [],
};
```

Modify `server\RemoteScene.tsx` to generate and render the `path`:

```typescript
import * as DCL from 'metaverse-api'
import { Vector2Component } from 'metaverse-api'
import { setState, getState } from './State'

export default class CreepsScene extends DCL.ScriptableScene 
{
  sceneDidMount() 
  {
    if(getState().path.length == 0)
    {      
      this.newGame();
    }
  }

  newGame()
  {
    while(true)
    {
      try 
      {
        setState({
          path: generatePath(),
        });
  
        break;
      }
      catch {}
    }
  }

  renderTiles()
  {
    return getState().path.map((gridPosition) =>
    {
      return (
        <box position={{x: gridPosition.x, y: 0, z: gridPosition.y}} />
      );
    });
  }

  async render() 
  {
    return (
      <scene>
        {this.renderTiles()}
      </scene>
    );
  }
}

function getStartPosition(): Vector2Component
{
  return {x: 10, y: 1};
}

function isValidPosition(position: Vector2Component)
{
  return position.x >= 1 
    && position.x < 19 
    && position.y >= 1 
    && position.y < 19
    && (position.x < 18 || position.y < 18)
    && (position.x > 1 || position.y > 1);
}

function generatePath(): Vector2Component[]
{
  const path: Vector2Component[] = [];
  let position = getStartPosition();
  path.push(JSON.parse(JSON.stringify(position)));
  for(let i = 0; i < 2; i++)
  {
    position.y++;
    path.push(JSON.parse(JSON.stringify(position)));
  }

  let counter = 0;
  while(position.y < 18)
  {
    if(counter++ > 1000)
    {
      throw new Error("Invalid path, try again");
    }
    let nextPosition = {x: position.x, y: position.y};
    switch(Math.floor(Math.random() * 3))
    {
      case 0:
        nextPosition.x += 1;
        break;
      case 1:
        nextPosition.x -= 1;
        break;
      default:
        nextPosition.y += 1;
    }
    if(!isValidPosition(nextPosition) 
      || path.find((p) => p.x == nextPosition.x && p.y == nextPosition.y)
      || getNeighborCount(path, nextPosition) > 1)
    {
      continue;
    }
    position = nextPosition;
    path.push(JSON.parse(JSON.stringify(position)));
  }
  position.y++;
  path.push(JSON.parse(JSON.stringify(position)));
  return path;
}

function getNeighborCount(path: Vector2Component[], position: Vector2Component)
{
  const neighbors: {x: number, y: number}[] = [
    {x: position.x + 1, y: position.y},
    {x: position.x - 1, y: position.y},
    {x: position.x, y: position.y + 1},
    {x: position.x, y: position.y - 1},
  ];

  let count = 0;
  for(const neighbor of neighbors)
  {
    if(path.find((p) => p.x == neighbor.x && p.y == neighbor.y))
    {
      count++;
    }
  }

  return count;
}
```

**Test**: A random path should appear, rendered as white boxes (we'll style next).

## Create a Component to Render Tiles

For this tutorial, we will be separating out the render logic for various components into their own file.  This helps with readability as your app becomes more elaborate. 

Components only include the render information.  Any logic, including responding to events, is still owned by the main scene's class (`server\RemoteScene.tsx`).

Data, including state information, is communicated from the scene's class to the component by using properties. 

Here's [Decentraland's docs on components](https://docs.decentraland.org/sdk-reference/scene-state/#reference-the-state-from-a-child-object).

Create a `components` directory and a file `server\components\Tile.tsx`:

```typescript
import * as DCL from 'metaverse-api'
import { Vector2Component } from 'metaverse-api';

export interface ITileProps 
{
  gridPosition: Vector2Component,
}

export const Tile = (props: ITileProps) => 
{
  return (
    <plane
      position={{x: props.gridPosition.x, y: .01, z: props.gridPosition.y}}
      material="#floorTileMaterial"
      rotation={{x: 90, y: 0, z: 0}}
    />
  )
}
```

Add a `material` tag defining the texture for the Tiles to use.  

```typescript
<scene>
  <material 
    id="floorTileMaterial" 
    albedoTexture="./assets/StoneFloor.png"
  />
  {this.renderTiles()}
```

The material is defined once and then leveraged for every individual tile.  See [Decentraland's doc on Materials](https://docs.decentraland.org/sdk-reference/scene-content-guide/#materials).

Change the `renderTiles` function to leverage the `Tile` component we created:

```typescript
import { Tile, ITileProps } from './components/Tile'
...

renderTiles()
{
  return getState().path.map((gridPosition) =>
  {
    const tileProps: ITileProps = {
      gridPosition
    };
    return Tile(tileProps);
  });
}
```

## Static Scenery 

Add a bit of static scenery to pretty the place up a bit:

```typescript
  const endOfPath = getState().path[getState().path.length - 2];

  return (
    <scene>
      <material 
        id="floorTileMaterial" 
        albedoTexture="./assets/StoneFloor.png"
      />
      {this.renderTiles()}

      <plane
        position={{x: 10, y: 0, z: 10}}
        rotation={{x: 90, y: 0, z: 0}}
        scale={19.99}
        color="#666666"
      />        
      <gltf-model
        src="assets/Archway/StoneArchway.gltf"
        position={{x: 10, y: 0, z: 2}}
        rotation={{x: 0, y: 180, z: 0}}
        scale={{x: 1, y: 1, z: 1.5}}
      />
      <gltf-model
        src="assets/Archway/StoneArchway.gltf"
        position={{x: endOfPath.x, y: 0, z: endOfPath.y}}
        scale={{x: 1, y: 1, z: 1.5}}
      />
    </scene>
  );
```

## Add Creeps

Create a component to render a creep at `server\components\Creep.tsx`:

```typescript
import * as DCL from 'metaverse-api'
import { Vector2Component } from 'metaverse-api';

export interface ICreepProps 
{
  id: string,
  gridPosition: Vector2Component,
  isDead: boolean,
}

export const Creep = (props: ICreepProps) => 
{
  return (
    <gltf-model
      id={props.id}
      src="../assets/BlobMonster/BlobMonster.gltf" 
      position={{x: props.gridPosition.x, y: .1, z: props.gridPosition.y}}
      lookAt={{x: props.gridPosition.x, y: 0, z: props.gridPosition.y}}
      skeletalAnimation={[
        {
          clip: "Walking",
          playing: !props.isDead
        },
        {
          clip: "Dying",
          playing: props.isDead
        },
      ]}
      transition={{
        position: {
          duration: 500,
        },
        lookAt: {
          duration: 250,
        }
      }}
    />
  )
}
```

Update `server\State.ts` to add Creeps:

```typescript
import { ICreepProps } from './components/Creep'

let state: {
  path: Vector2Component[],
  creeps: ICreepProps[],
} = {
  path: [],
  creeps: [],
};
```

Import the component and add a `sleep` method, a timer, and an object counter to the `server\RemoteScene.tsx`:

```typescript
import { Creep, ICreepProps } from './components/Creep'

function sleep(ms: number): Promise<void> 
{
  return new Promise(resolve => setTimeout(resolve, ms));
} 
let spawnInterval: NodeJS.Timer;
let objectCounter = 0;
```

After the while loop in `newGame`, add:

```typescript
clearInterval(spawnInterval);
spawnInterval = setInterval(() =>
{
  this.spawnEntity();
}, 3000 + Math.random() * 17000);
```

Add the `spawnEntity` and `kill` functions below:

```typescript
async spawnEntity()
{
  for(const creep of getState().creeps)
  {
    if(JSON.stringify(creep.gridPosition) == JSON.stringify(getStartPosition()))
    {
      return;
    }
  }

  let creep: ICreepProps = {
    id: "Creep" + objectCounter++,
    gridPosition: getStartPosition(),
    isDead: false,
  };
  setState({creeps: [...getState().creeps, creep]});

  let pathIndex = 1;
  while(true)
  {
    if(creep.isDead)
    {
      return;
    }

    if(pathIndex >= getState().path.length)
    {
      this.kill(creep);
    }
    else
    {
      creep.gridPosition = getState().path[pathIndex];
      pathIndex++;        
      setState({creeps: getState().creeps});
    }

    await sleep(2000);
  }
}

async kill(creep: ICreepProps)
{
  creep.isDead = true;
  setState({creeps: getState().creeps});

  await sleep(2000);
  let creeps = getState().creeps.slice();
  creeps.splice(creeps.indexOf(creep), 1);
  setState({creeps});
}
```

Add creeps to `render`:

```typescript
<scene>
...
  {this.renderCreeps()}
</scene>
```

And create a function `renderCreeps`:

```typescript
renderCreeps()
{
  return getState().creeps.map((creep) =>
  {
    return Creep(creep);
  });
}
```

## Add Traps

The traps have three components.  There are two levers and a set of spikes.  When one lever has been pulled the other unlocks.  Then when the second lever is pulled the spikes trigger for about a second, killing any creeps standing above.

Create a component for the traps at `server\components\Trap.tsx`:

```typescript
import * as DCL from 'metaverse-api'
import { Vector2Component } from 'metaverse-api';

export const enum TrapState 
{
  Available,
  PreparedOne,
  PreparedBoth,
  Fired,
  NotAvailable,
}

export interface ITrapProps 
{
  id: string,
  gridPosition: Vector2Component,
  trapState: TrapState,
}

export const Trap = (props: ITrapProps) => 
{
  return (
    <entity>
      <gltf-model
        src="../assets/Lever/LeverBlue.gltf"
        id={props.id + "LeverLeft"}
        position={{x: props.gridPosition.x - 1, y: 0, z: props.gridPosition.y}}
        scale={.5}
        rotation={{x: 0, y: 90, z: 0}}
        skeletalAnimation={[
          {
            clip:"LeverOff", 
            playing: props.trapState <= TrapState.Available
          },
          {
            clip:"LeverOn", 
            playing: props.trapState == TrapState.PreparedOne
          },
          {
            clip:"LeverDeSpawn", 
            playing: props.trapState >= TrapState.Fired
          },
        ]}
      />
      <gltf-model
        id={props.id}
        src="../assets/SpikeTrap/SpikeTrap.gltf"
        position={{x: props.gridPosition.x, y: 0, z: props.gridPosition.y}}
        skeletalAnimation={[
          {
            clip:"SpikeUp", 
            playing: props.trapState == TrapState.Fired,
          },
          {
            clip:"Despawn", 
            playing: props.trapState == TrapState.NotAvailable
          },
        ]}
        scale={.5}
      />
      <gltf-model
        id={props.id + "LeverRight"}
        src="../assets/Lever/LeverRed.gltf"
        position={{x: props.gridPosition.x + 1, y: 0, z: props.gridPosition.y}}
        scale={.5}
        rotation={{x: 0, y: 90, z: 0}}
        skeletalAnimation={[
          {
            clip:"LeverOff", 
            playing: props.trapState <= TrapState.Available
          },
          {
            clip:"LeverOn", 
            playing: props.trapState == TrapState.PreparedBoth
          },
          {
            clip:"LeverDeSpawn", 
            playing: props.trapState >= TrapState.Fired
          },
        ]}
      /> 
    </entity>
  )
}
```

Add trap to `server\State.ts`:

```typescript
import { ITrapProps} from './components/Trap'

let state: {
  ...
  traps: ITrapProps[],
} = {
  ...
  traps: [],
};
```

In `server\RemoteScene.tsx` add:

```typescript
import { Trap, ITrapProps, TrapState } from './components/Trap'
```

Then inside the `newGame` function spawn two traps:

```typescript
newGame()
  {
    while(true)
    {
      try 
      {
        ...
        this.spawnTrap();
        this.spawnTrap();
  
        break;
```

Add functions for spawning traps and responding to click events:

```typescript
spawnTrap()
{
  let trap: ITrapProps = {
    id: "Trap" + objectCounter++,
    gridPosition: this.randomTrapPosition(),
    trapState: TrapState.Available,
  };
  setState({traps: [...getState().traps, trap]});
  this.subToTrap(trap);
}

subToTrap(trap: ITrapProps)
{
  this.eventSubscriber.on(trap.id + "LeverLeft_click", () =>
  {
    if(trap.trapState != TrapState.Available)
    {
      return;
    }
    trap.trapState = TrapState.PreparedOne;
    setState({traps: getState().traps});
  });

  this.eventSubscriber.on(trap.id + "LeverRight_click", async () =>
  {
    if(trap.trapState != TrapState.PreparedOne)
    {
      return;
    }
    trap.trapState = TrapState.PreparedBoth;
    setState({traps: getState().traps});

    await sleep(1000);
    trap.trapState = TrapState.Fired;
    setState({traps:  getState().traps});
    let counter = 0;

    while(true)
    {
      await sleep(100);
      
      for(const entity of getState().creeps)
      {
        if(JSON.stringify(entity.gridPosition) == JSON.stringify(trap.gridPosition) && !entity.isDead)
        {
          this.kill(entity);
        }
      }
      if(counter++ > 10)
      {
        trap.trapState = TrapState.NotAvailable;
        setState({traps: getState().traps});
        
        await sleep(1000);
        let traps = getState().traps.slice();
        traps.splice(traps.indexOf(trap), 1)
        setState({traps});
        
        await sleep(1000);
        this.spawnTrap(); 

        break;
      }
    };
  });
}

randomTrapPosition()
{
  let counter = 0;
  while(true)
  {
    if(counter++ > 1000)
    {
      throw new Error("Invalid path, try again");
    }

    const position = {x: Math.floor(Math.random() * 19), y: Math.floor(Math.random() * 19)};
    if(getState().path.find((p) => p.x == position.x && p.y == position.y)
      && !getState().path.find((p) => p.x == position.x - 1 && p.y == position.y)
      && !getState().path.find((p) => p.x == position.x + 1 && p.y == position.y)
      && position.y > 2
      && position.y < 18
      && position.x > 2
      && position.x < 18
      && !getState().traps.find((t) => JSON.stringify(position) == JSON.stringify(t.gridPosition)))
    {
      return position;  
    }
  } 
}
```

To ensure that someone joining a game-in-progress subscribes to events for the existing traps add the following to `sceneDidMount`.  This will subscribe to events for all the existing traps:

```typescript
sceneDidMount() 
{
  if(getState().path.length == 0)
  ...
  }
  else
  {
    for(const trap of getState().traps)
    {
      this.subToTrap(trap);
    }
  }
```

Add `renderTraps`:

```typescript
<scene>
  ...
  {this.renderTraps()}
</scene>
```

And the function itself:

```typescript
renderTraps()
{
  return getState().traps.map((trap) =>
  {
    return Trap(trap);
  });
}
```

## Score

Create a `server\components\ScoreBoard.tsx` component:

```typescript
import * as DCL from 'metaverse-api'

export interface IScoreBoardProps 
{
  humanScore: number,
  creepScore: number,
}

export const ScoreBoard = (props: IScoreBoardProps) => 
{
  return (
    <entity
        position={{x: 18.99, y: 0, z: 19}}
    >
      <gltf-model
        src="../assets/ScoreRock/ScoreRock.gltf"
      />
      <text 
        value={props.humanScore.toString()}
        position={{x: -.4, y: .35, z: -.38}}
        fontSize={200}
        color={props.humanScore > props.creepScore ? "#22ff22" : "#ffffff"}
      />
      <text 
        value="humans"
        position={{x: -.4, y: .1, z: -.38}}
        fontSize={50}
      />
      <text 
        value="vs"
        position={{x: 0, y: .35, z: -.38}}
        fontSize={100}
      />
      <text 
        value={props.creepScore.toString()}
        position={{x: .4, y: .35, z: -.38}}
        fontSize={200}
        color={props.creepScore > props.humanScore ? "#ff2222" : "#ffffff"}
      />
      <text 
        value="creeps"
        position={{x: .4, y: .1, z: -.38}}
        fontSize={50}
      />
    </entity>
  )
}
```

Update `server\State.ts`:

```typescript
import { IScoreBoardProps } from './components/ScoreBoard'

let state: {
  ...
  score: IScoreBoardProps,
} = {
  ...
  score: {humanScore: 0, creepScore: 0},
};
```

In `scene\RemoteScene.tsx`:

```typescript
import { ScoreBoard } from './components/ScoreBoard'
```

And add the score board to `render`:

```typescript
<scene>
  ...
  {ScoreBoard(getState().score)}
</scene>
```

**Test**: The scoreboard should appear, `0 v 0`.

Now let's update the score when a trap kills the creep:

```typescript
if(JSON.stringify(entity.gridPosition) == JSON.stringify(trap.gridPosition) && !entity.isDead)
{
  this.kill(entity);

  let score = getState().score;
  score.humanScore++;
  setState({score});
}
```

And when the creep makes it to the end:

```typescript
if(pathIndex >= getState().path.length)
{
  this.kill(creep);
  
  let score = getState().score;
  score.creepScore++;
  setState({score});
}
```

**Test**: Kill a creep or two and allow some to reach the end.  You should see the scoreboard update appropriately. 

## New Game Button

Create a `server\components\Button.tsx` component:

```typescript
import * as DCL from 'metaverse-api'
import { Vector3Component } from 'metaverse-api';

export enum ButtonState
{
  Normal,
  Pressed,
}

export interface IButtonProps 
{
  id: string,
  position: Vector3Component,
  state: ButtonState,
  label: string,
}

export const Button = (props: IButtonProps) => 
{
  let buttonZ = 0;
  if(props.state == ButtonState.Pressed)
  {
    buttonZ = .06;
  }
  return (
    <entity
      position={props.position}>
      <cylinder
        id={props.id}
        position={{x: 0, y: 0, z: buttonZ}}
        transition={{
          position: {
            duration: 100,
          },
        }} 
        rotation={{x: 90, y: 0, z: 0}}
        scale={{x: .05, y: .2, z: .05}}
        color="#990000" 
        />
      <text 
        hAlign="left"
        value={props.label} 
        position={{x: .4, y: 0, z: -.15}}
        scale={.6}
      />
    </entity>
  )
}
```

Update `server\State.ts`:

```typescript
import { IButtonProps, ButtonState } from './components/Button'

let state: {
  ...
  startButton: IButtonProps,
} = {
  ...
  startButton: {
    id: "newGame",
    position: {x: 18.65, y: .7, z: 18.75},
    state: ButtonState.Normal,
    label: "New Game",
  }
};
```

In `server\RemoteScene.tsx`:

```typescript
import { Button, ButtonState } from './components/Button'
```

Add the following to `sceneDidMount`:

```typescript
this.eventSubscriber.on("newGame_click", async () =>
{
  let startButton = getState().startButton;
  startButton.state = ButtonState.Pressed;
  setState({startButton});
  await sleep(500);
  this.newGame();
  startButton.state = ButtonState.Normal;
  setState({startButton});
});
```

Modify `setState` in the `newGame` function to clear the other variables when the game restarts:

```typescript
setState({
  path: generatePath(),
  creeps: [],
  traps: [],
  score: {humanScore: 0, creepScore: 0},
});
```

And update the `render` function:

```typescript
<scene>
  {Button(getState().startButton)}
</scene>
```

<hr>

<br>

That’s it!  This is a bare-bones implementation of a game, obviously it needs more in order to be compelling.  Hope this helps you get started. 

Next steps:
 - Change the lever interactions to require more than one person to be involved.
 - Add more weapon types, instead of just the trap.
 - Track per-player scores (and maintain stats b/w games).
 - Add health, instead of one-shot kills.
 - Make the creeps spawn faster and walk faster as the game progresses, and/or randomize their movement.
