# Types overview

[egghead.io lesson 11: More mobx-state-tree Types: map, literal, union, and enumeration](https://egghead.io/lessons/react-more-mobx-state-tree-types-map-literal-union-and-enumeration) [egghead.io lesson 17: Create Dynamic Types and use Type Composition to Extract Common Functionality](https://egghead.io/lessons/react-create-dynamic-types-and-use-type-composition-to-extract-common-functionality)

These are the types available in MST. All types can be found in the `types` namespace, e.g. `types.string`. See [Api Docs](https://mobx-state-tree.gitbook.io/docs/api) for examples.

### Complex types

* `types.model(properties, actions)` Defines a "class like" type, with properties and actions to operate on the object.
* `types.array(type)` Declares an array of the specified type.
* `types.map(type)` Declares a map of the specified type.

### Primitive types

* `types.string`
* `types.number`
* `types.boolean`
* `types.Date`
* `types.custom` creates a custom primitive type. This is useful to define your own types that map a serialized form one-to-one to an immutable object like a Decimal or Date.

### Utility types

* `types.union(dispatcher?, types...)` create a union of multiple types. If the correct type cannot be inferred unambiguously from a snapshot, provide a dispatcher function of the form `(snapshot) => Type`.
* `types.optional(type, defaultValue)` marks an value as being optional \(in e.g. a model\). If a value is not provided the `defaultValue` will be used instead. If `defaultValue` is a function, it will be evaluated. This can be used to generate, for example, IDs or timestamps upon creation.
* `types.literal(value)` can be used to create a literal type, where the only possible value is specifically that value. This is very powerful in combination with `union`s. E.g. `temperature: types.union(types.literal("hot"), types.literal("cold"))`.
* `types.enumeration(name?, options: string[])` creates an enumeration. This method is a shorthand for a union of string literals.
* `types.refinement(name?, baseType, (snapshot) => boolean)` creates a type that is more specific than the base type, e.g. `types.refinement(types.string, value => value.length > 5)` to create a type of strings that can only be longer then 5.
* `types.maybe(type)` makes a type optional and nullable, shorthand for `types.optional(types.union(type, types.literal(null)), null)`.
* `types.null` the type of `null`
* `types.undefined` the type of `undefined`
* `types.late(() => type)` can be used to create recursive or circular types, or types that are spread over files in such a way that circular dependencies between files would be an issue otherwise.
* `types.frozen` Accepts any kind of serializable value \(both primitive and complex\), but assumes that the value itself is **immutable** and **serializable**.
* `types.compose(name?, type1...typeX)`, creates a new model type by taking a bunch of existing types and combining them into a new one

### Property types

Property types can only be used as a direct member of a `types.model` type and not further composed \(for now\).

* `types.identifier(subType?)` Only one such member can exist in a `types.model` and should uniquely identify the object. See [identifiers](https://mobx-state-tree.gitbook.io/docs/concepts/references-and-identifiers#identifiers) for more details. `subType` should be either `types.string` or `types.number`, defaulting to the first if not specified.
* `types.reference(targetType)` creates a property that is a reference to another item of the given `targetType`somewhere in the same tree. See [references](https://mobx-state-tree.gitbook.io/docs/concepts/references-and-identifiers#references) for more details.

### LifeCycle hooks for `types.model`

[egghead.io lesson 14: Loading Data from the Server after model creation](https://egghead.io/lessons/react-loading-data-from-the-server)

All of the below hooks can be created by returning an action with the given name, like:

```javascript
const Todo = types
    .model("Todo", { done: true })
    .actions(self => ({
        afterCreate() {
            console.log("Created a new todo!")
        }
    }))
```

The exception to this rule is the `preProcessSnapshot` hook. Because it is needed before instantiating model elements, it needs to be defined on the type itself:

```javascript
types
    .model("Todo", { done: true })
    .preProcessSnapshot(snapshot => ({
        // auto convert strings to booleans as part of preprocessing
        done: snapshot.done === "true" ? true : snapshot.done === "false" ? false : snapshot.done
    }))
    .actions(self => ({
        afterCreate() {
            console.log("Created a new todo!")
        }
    }))
```

| Hook | Meaning |
| --- | --- | --- | --- | --- | --- | --- |
| `preProcessSnapshot` | Before creating an instance or applying a snapshot to an existing instance, this hook is called to give the option to transform the snapshot before it is applied. The hook should be a _pure_function that returns a new snapshot. This can be useful to do some data conversion, enrichment, property renames etc. This hook is not called for individual property updates. _**Note 1: Unlike the other hooks, this one is not**_ **created as part of the `actions`initializer, but directly on the type!** _**Note 2: The `preProcessSnapshot` transformation must be pure; it should not modify its original input argument!**_ |
| `afterCreate` | Immediately after an instance is created and initial values are applied. Children will fire this event before parents. You can't make assumptions about the parent safely, use `afterAttach` if you need to. |
| `afterAttach` | As soon as the _direct_ parent is assigned \(this node is attached to another node\). If an element is created as part of a parent, `afterAttach` is also fired. Unlike `afterCreate`, `afterAttach` will fire breadth first. So, in `afterAttach` one can safely make assumptions about the parent, but in `afterCreate` not |
| `postProcessSnapshot` | This hook is called every time a new snapshot is being generated. Typically it is the inverse function of `preProcessSnapshot`. This function should be a pure function that returns a new snapshot. |
| `beforeDetach` | As soon as the node is removed from the _direct_ parent, but only if the node is _not_ destroyed. In other words, when `detach(node)` is used |
| `beforeDestroy` | Called before the node is destroyed, as a result of calling `destroy`, or by removing or replacing the node from the tree. Child destructors will fire before parents |

Note, except for `preProcessSnapshot`, all hooks should be defined as actions.

All hooks can be defined multiple times and can be composed automatically.

## 



