---
date: 2018-01-05
title: Event Handling
description: Learn the different events that can occur in a scene and how to catch them.
redirect_from:
  - /sdk-reference/event-handling/
categories:
  - development-guide
type: Document
set: development-guide
set_order: 3
---

When users interact with the entities in your scene, these generate several types of events. These events can have an effect on the scene [state]({{ site.baseurl }}{% post_url /development-guide/2018-01-04-scene-state %}), which triggers a new rendering of the scene.

![](/images/media/events_state_diagram.jpeg)

Generally, a good way of having your scene respond to events is to set up a listener in the `sceneDidMount()` method. See [scriptable scene]({{ site.baseurl }}{% post_url /development-guide/2018-01-05-scriptable-scene %}) for more context about when this method is executed.

{% raw %}

```tsx
async sceneDidMount() {
  this.eventSubscriber.on(`pointerDown`, () => console.log("pointer down"))
}
```

{% endraw %}

To debug a scene, you can use `console.log()` to keep track of the occurrence of events or to verify that the event's parameters are what you expected.

## Clicking

Clicks can be done either with a mouse, a touch screen, a VR controller or some other device, the events that are generated from this don't make any distinction. When the ray that the avatar casts forward points at a valid entity and the user clicks, this creates a `click` event.

> Note: Clicks can be made from a maximum distance of 10 meters away from the entity.

#### onClick

The easiest way to handle click events is to add an `onClick` component to the entity itself. With this in place, there's no need to add an event subscriber for click events from this entity, that's already implicitly handled.

You can declare what to do in the event of a click by writing a lambda in the `onClick` itself, or you can call a separate function to keep the render method more legible.

{% raw %}

```tsx
<box
  onClick={() => console.log("Clicked!")}
  position={{ x: 5, y: 1, z: 5 }}
  scale={{ x: 2, y: 2, z: 1 }}
/>
```

{% endraw %}

If you call a function from onClick, any uses of the `this` operator refer to the function itself, not to the [scriptable scene object]({{ site.baseurl }}{% post_url /development-guide/2018-01-05-scriptable-scene %}). It can sometimes be a problem if you need to refer to the scene state or to other functions in the scene. To avoid this problem, you can either define the function as a lambda or call the function through a lambda defined in the `onClick` value (as in the example above). See [TypeScript Tips]({{ site.baseurl }}{% post_url /development-guide/2018-01-08-typescript-tips %}) for more complete examples of how to work around this.

The click event object is passed as a parameter of the function you call in the `onClick`. This event object contains the following parameters that can be accessed by your function:

- `elementId`: the ID of the entity that was clicked (if the entity has an id).
- `pointerId`: the ID for the user who performed the click.

{% raw %}

```tsx
import { ScriptableScene, createElement } from "decentraland-api/src"

export default class Scene extends ScriptableScene {
  async render() {
    return (
      <scene>
        <box
          position={{ x: 5, y: 0, z: 5 }}
          id="myBox"
          onClick={e => {
            console.log(`elementId: ` + e.elementId)
            console.log(`pointerId: ` + e.pointerId)
          }}
        />
      </scene>
    )
  }
}
```

{% endraw %}

This example logs both parameters of the click event each time the box entity is clicked.

#### The generic click event

The generic `click` event represents all clicks done on valid entities. Only entities that have an id are considered valid for generating click events. The click event object contains the following parameters:

- `elementId`: the Id of the entity that was clicked.
- `pointerId`: the id for the user who performed the click.

{% raw %}

```tsx
import { createElement, ScriptableScene } from "decentraland-api"

export default class LastClicked extends ScriptableScene {
  state = {
    lastClicked: "none"
  }

  async sceneDidMount() {
    this.subscribeTo("click", e => {
      this.setState({ lastClicked: e.elementId })
      console.log(this.state.lastClicked)
    })
  }

  async render() {
    return (
      <scene>
        <box id="box1" position={{ x: 2, y: 1, z: 0 }} color="ff0000" />
        <box id="box2" position={{ x: 4, y: 1, z: 0 }} color="00ff00" />
      </scene>
    )
  }
}
```

{% endraw %}

The example above uses the `subscribeTo` to initiate a listener that checks for all click events. When the user clicks on either of the two boxes, the scene stores the id of the clicked entity in the `lastClicked` state variable and prints it to console.

#### Entity-specific click events

A simpler way to deal with clicks that are made on a single entity is to listen for click events that are specific for that entity. The names of entity-specific click events are as follows: the id of the entity, an underscore and then _click_. For example, the event created from clicking an entity called `redButton` is named `redButton_click`.

> Note: Entity-specific click events have no properties, so you can't access the user's id from this event.

{% raw %}

```tsx
import { createElement, ScriptableScene } from "decentraland-api"

export default class RedButton extends ScriptableScene {
  state = {
    buttonState: false
  }

  async sceneDidMount() {
    this.eventSubscriber.on("redButton_click", () => {
      this.setState({ buttonState: !this.state.buttonState })
      console.log(this.state.buttonState)
    })
  }

  async render() {
    return (
      <scene>
        <box id="redButton" color="ff0000" />
      </scene>
    )
  }
}
```

{% endraw %}

The scene above uses an `eventSubscriber` to initiate a listener that checks for click events done on the `redButton` entity. Whenever this occurs, the state of `buttonState` is toggled.

## Pointer down and pointer up

The pointer down and pointer up events are fired whenever the user presses or releases an input controller. This could be a mouse, a touch screen, a VR controller or another kind of controller. It doesn't matter where the user's avatar is pointing at, the event is triggered every time.

{% raw %}

```tsx
import { createElement, ScriptableScene } from "decentraland-api"

export default class BigButton extends ScriptableScene {
  state = {
    buttonState: 0
  }

  async sceneDidMount() {
    this.subscribeTo("pointerDown", e => {
      this.setState({ buttonState: 0 })
    })
    this.subscribeTo("pointerUp", e => {
      this.setState({ buttonState: 1 })
    })
  }

  async render() {
    return (
      <scene>
        <box
          id="button"
          position={{ x: 3, y: this.state.buttonState, z: 3 }}
          transition={{ position: { duration: 200, timing: "linear" } }}
        />
      </scene>
    )
  }
}
```

{% endraw %}

The scene above uses two `subscribeTo` functions to initiate listeners that check both when the user clicks or releases a pointer button. Both listener functions alter the `buttonState` state variable in the scene. This variable is then used to set the height of a box that mimics the pressing of the user's button.

## Position change

The `positionChanged` event sends the position of the user each time it changes.

The `positionChanged` event has the following properties:

- `position`: a Vector3Component with the user's position relative to the base parcel of the scene.
- `cameraPosition`: a Vector3Component with the user's absolute position relative to the world.
- `playerHeight`: the eye height of the user, in meters.

{% raw %}

```tsx
import { createElement, ScriptableScene } from "decentraland-api"

export default class BoxFollower extends ScriptableScene {
  state = {
    boxPosition: { x: 0, y: 0, z: 0 }
  }

  async sceneDidMount() {
    this.subscribeTo("positionChanged", e => {
      this.setState({ boxPosition: e.position })
    })
  }

  async render() {
    return (
      <scene>
        <box position={this.state.boxPosition} />
      </scene>
    )
  }
}
```

{% endraw %}

The scene above uses a `subscribeTo` function to initiate a listener to track when the position of the user changes. When the user moves, the scene stores the current position in the state variable `boxPosition`, which is used to set the position of a box that follows the user around.

## Rotation change

The `rotationChanged` event sends the angle in which the user is looking each time it changes.

The `rotationChanged` event has the following properties:

- `rotation`: a Vector3Component with the user's rotation.

- `quaternion`: the rotation of the user expressed as a quaternion.

{% raw %}

```tsx
import { createElement, ScriptableScene } from "decentraland-api"

export default class ConeHead extends ScriptableScene {
  state = {
    rotation: { x: 0, y: 0, z: 0 }
  }

  async sceneDidMount() {
    this.subscribeTo("rotationChanged", e => {
      e.rotation.x += 90
      this.setState({ rotation: e.rotation })
    })
  }

  async render() {
    return (
      <scene>
        <cone position={{ x: 0, y: 1, z: 2 }} rotation={this.state.rotation} />
      </scene>
    )
  }
}
```

{% endraw %}

The scene above uses a `subscribeTo` function to initiate a listener to track the user's rotation. When the user looks in a different direction, the scene stores the current angle in the state variable `rotation`. This example adds another 90 degrees to the X axis of this angle just to make the output more fun to play with. This angle is used to orient a cone that faces and mimics the user.

## Create custom events

For scenes that include complex logic like games, it makes sense to create custom events. For example, in a game you might define a custom event called _lose_ and send it whenever a player loses, you can then subscribe different functions to whenever this event occurs.

To enable custom events, you must first create a custom event subscriber and import it into your scene, as described below.

#### Create an event manager

To keep your scene's code cleaner, place the event manager in its own file, we recommend `ts\EventManager.ts`.

{% raw %}

```tsx
import { EventSubscriber } from "decentraland-api"

export namespace EventManager {
  let eventSubscriber: EventSubscriber

  export function init(_eventSubscriber: EventSubscriber) {
    eventSubscriber = _eventSubscriber
  }

  export function emit(eventType: string, ...params: any[]) {
    eventSubscriber.emit(eventType, ...params)
  }
}
```

{% endraw %}

The event manager has two simple functions, one to subscribe to initiate the subscriber and the other to emit instances of an event.

#### Subscribe to events

In your scene, you need to initiate the event subscriber before you can use it. We recommend doing that in the `sceneDidMount()` function so that it starts working as soon as the scene is loaded. Don't forget to also import the event subscriber you defined in `EventManager.ts` to the `scene.tsx` file.

{% raw %}

```tsx
 sceneDidMount() {
    EventManager.init(this.eventSubscriber);
    ...
  }
```

{% endraw %}

The example below initiates the event subscriber and then subscribes to the custom event _lose_, that is emitted when the user loses a game. Whenever the _lose_ event occurs, the function `userLost()` is called.

{% raw %}

```tsx
 sceneDidMount() {
    EventManager.init(this.eventSubscriber);
    this.eventSubscriber.on('lose', e => this.userLost());
  }
```

{% endraw %}

#### Emit custom events

You can trigger the emission of custom events at any part of your scen's code. To do so, reach out to the `emit()` function that the event subscriber exposes. If other parts of your scene's code are subscribed to the event you're emitting, they will execute the corresponding code when the event is emitted.

{% raw %}

```tsx
 harmUser(){
    this.setState({health: this.state.health -= 1})
    if (this.state.health <= 0)
    {
      EventManager.emit("lose")
    }
  }
```

{% endraw %}
