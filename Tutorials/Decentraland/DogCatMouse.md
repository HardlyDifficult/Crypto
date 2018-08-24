# Dog, Cat, Mouse, Cheese.  Decentraland.

TODO intro.  

We'll be creating a scene in which a predator guards its home, attacking any prey in sight.  The prey is on a mission: sneak through the fence, get some cheese, and get out safely.

In this example, we will create a stack-based finite state machine (FSM) to manage AI for animals.

## Getting Started

We are starting with a scene and a collection of components already created.  For more information about components and how to get started with Decentraland, see one of our previous tutorials: TODO 

### Download the Starting Scene

Get the scene and art from:

 - https://github.com/hardlydifficult/DogCatMouseCheese/archive/Static_Scene.zip

And extract it anywhere.

### Start Decentraland

Open a command prompt, navigate to the project's directory, and run:

```
dcl start
```

This will open a new browser tab with the scene.

**Test**: Walk around and take a look. Nice, right?  Thanks James (the artist).

## Adding a Grid

For path finding and collision handling, we will be logically positioning objects into a grid in which each cell is 1m x 1m.

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

To ease testing, we'll add a click event which will restart the world.

In `scene.tsx` add:

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

Then update `sceneDidMount` to add:

```typescript
this.eventSubscriber.on("Exit_click", e => this.onExitClick());
```

**Test**: Click on the exit mount and the trees should re-spawn with new random positions.

### Add Static Scenery to the Grid

The `spawnTrees` algorithm above includes a loop to select a position with clearance / free space around it.  For this to work, we'll need to register the position of each of our static scenery objects with the grid.

Update `ts/SceneHelper` by adding the following method:

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

Then in `scene.tsx`, update `sceneDidMount` to update the grid:

```typescript
sceneDidMount()
{
  Grid.init(30, 30);
  SceneHelper.updateGridWithStaticScenery();
  Grid.set(this.state.baitProps.position);
  this.spawnTrees();
}
```

**Test**: Click the exit mount several times and confirm the trees are never overlapping scenery.

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
  this.eventSubscriber.on("Exit_click", e => this.onExitClick());
}
```

## Animals
```
	// Helper methods
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
				{ animation: Animation.Idle, weight: 1 },
				{ animation: Animation.Walk, weight: 0 },
				{ animation: Animation.Drink, weight: 0 },
				{ animation: Animation.Dead, weight: 0 },
				{ animation: Animation.Run, weight: 0 },
				{ animation: Animation.Sit, weight: 0 },
			],
			isDead: false,
			scale: 1,
		};
		this.setState({ animals: [...this.state.animals, animal] });
		this.eventSubscriber.on(animal.id + "_click", () =>
		{
			AnimalStateMachine.sendMessage(animal.id, "click");
		});

		return animal;
	}


onExitClick()
	{ 
		for (const animal of this.state.animals.slice())
		{
			EventManager.emit("despawn", animal.id);
		}
	
	onRenderAnimals()
	{
		this.setState({ animals: this.state.animals });
	}
	onEntranceClick()
	{ // Spawn prey
		const animalProps = this.spawnAnimal(
			config.prey.animalType,
			SceneHelper.entranceProps.position,
			add(SceneHelper.entranceProps.position, { x: 1, y: 0, z: 0 }),
			config.prey.sneakSpeed);
		if (animalProps)
		{
			AnimalStateMachine.pushStates([
				new StateDespawn(animalProps, {delay: 1000}),
				new StateGoTo(animalProps, SceneHelper.exitProps, config.prey.exitConfig, config.prey.blockedConfig),
				new StateEat(animalProps, this.state.baitProps, config.prey.eatConfig, config.prey.blockedConfig),
			]);
		}
	}
	onHouseClick()
	{ // Spawn predator
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

## Something cool

Changes:
ts\EventManager
ts\StateMachine\*
scene.tsx

scene.tsx Didmount:
  EventManager.init(this.eventSubscriber);

		this.eventSubscriber.on("Entrance_click", e => this.onEntranceClick());
		this.eventSubscriber.on("House_click", e => this.onHouseClick());

		this.eventSubscriber.on('renderAnimals', e => this.onRenderAnimals());
		this.eventSubscriber.on('captureCheese', e => this.onCaptureBait());
		this.eventSubscriber.on('despawn', (animalId, delay) => this.onDespawn(animalId, delay));
		this.eventSubscriber.on('gridCellSet', cell => this.onGridCellSet(cell));



scene.tsx Logic:
// Events  
	async onCaptureBait()
	{
		await sleep(750);
		this.state.baitProps.isVisible = false;
		this.setState({ baitProps: this.state.baitProps });
		await sleep(2000);
		this.state.baitProps.isVisible = true;
		this.setState({ baitProps: this.state.baitProps });
	}
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



todo in Grid.ts
		EventManager.emit("gridCellSet", position);

