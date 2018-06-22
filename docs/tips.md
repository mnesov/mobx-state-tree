# Tips

### Building with production environment

MobX-state-tree provides a lot of dev-only checks. They check the correctness of function calls and perform runtime type-checks over your models. It is recommended to disable them in production builds. To do so, you should use webpack's DefinePlugin to set environment as production and remove them. More information could be found in the [official webpack guides](https://webpack.js.org/plugins/environment-plugin/#usage).

### Generate MST models from JSON

The following service can generate MST models based on JSON: [https://transform.now.sh/json-to-mobx-state-tree](https://transform.now.sh/json-to-mobx-state-tree)

### `optionals` and default value functions

`types.optional` can take an optional function parameter which will be invoked each time a default value is needed. This is useful to generate timestamps, identifiers or even complex objects, for example:

`createdDate: types.optional(types.Date, () => new Date())`

### `toJSON()` for debugging

For debugging you might want to use `getSnapshot(model, applyPostProcess)` to print the state of a model. But if you didn't import `getSnapshot` while debugging in some debugger; don't worry, `model.toJSON()` will produce the same snapshot. \(For API consistency, this feature is not part of the typed API\)

### Handle circular dependencies between files using `late`

On the exporting file:

```javascript
export function LateStore() {
    return types.model({
        title: types.string
    })
}
```

In the importing file

```javascript
import { LateStore } from "./circular-dep"

const Store = types.late(LateStore)
```

Thanks to function hoisting in combination with `types.late`, this lets you have circular dependencies between types, across files.

### Simulate inheritance by using type composition

There is no notion of inheritance in MST. The recommended approach is to keep references to the original configuration of a model in order to compose it into a new one, for example by using `types.compose` \(which combines two types\) or producing fresh types using `.props|.views|.actions`. An example of classical inheritance could be expressed using composition as follows:

```javascript
const Square = types
    .model(
        "Square",
        {
            width: types.number
        }
    )
    .views(self => ({
        surface() {
            return self.width * self.width
        }
    }))

// create a new type, based on Square
const Box = Square
    .named("Box")
    .views(self => {
        // save the base implementation of surface
        const superSurface = self.surface

        return {
            // super contrived override example!
            surface() {
                return superSurface() * 1
            },
            volume() {
                return self.surface * self.width
            }
        }
    }))

// no inheritance, but, union types and code reuse
const Shape = types.union(Box, Square)
```

Similarly, compose can be used to simply mixin types:

```javascript
const CreationLogger = types.model().actions(self => ({
    afterCreate() {
        console.log("Instantiated " + getType(self).name)
    }
}))

const BaseSquare = types
    .model({
        width: types.number
    })
    .views(self => ({
        surface() {
            return self.width * self.width
        }
    }))

export const LoggingSquare = types
    .compose(
        // combine a simple square model...
        BaseSquare,
        // ... with the logger type
        CreationLogger
    )
    // ..and give it a nice name
    .named("LoggingSquare")
```



