# References and identifiers

## References and identifiers

[egghead.io lesson 13: Create Relationships in your Data with mobx-state-tree Using References and Identifiers](https://egghead.io/lessons/react-create-relationships-in-your-data-with-mobx-state-tree-using-references-and-identifiers)

References and identifiers are a first-class concept in MST. This makes it possible to declare references, and keep the data normalized in the background, while you interact with it in a denormalized manner.

Example:

```javascript
const Todo = types.model({
    id: types.identifier(),
    title: types.string
})

const TodoStore = types.model({
    todos: types.array(Todo),
    selectedTodo: types.reference(Todo)
})

// create a store with a normalized snapshot
const storeInstance = TodoStore.create({
    todos: [{
        id: "47",
        title: "Get coffee"
    }],
    selectedTodo: "47"
})

// because `selectedTodo` is declared to be a reference, it returns the actual Todo node with the matching identifier
console.log(storeInstance.selectedTodo.title)
// prints "Get coffee"
```

### **Identifiers**

* Each model can define zero or one `identifier()` properties
* The identifier property of an object cannot be modified after initialization
* Each identifier / type combination should be unique within the entire tree
* Identifiers are used to reconcile items inside arrays and maps - wherever possible - when applying snapshots
* The `map.put()` method can be used to simplify adding objects that have identifiers to [maps](https://github.com/mobxjs/mobx-state-tree/blob/master/API.md#typesmap)
* The primary goal of identifiers is not validation, but reconciliation and reference resolving. For this reason identifiers cannot be defined or updated after creation. If you want to check if some value just looks as an identifier, without providing the above semantics; use something like: `types.refinement(types.string, v => v.match(/someregex/))`

_Tip: If you know the format of the identifiers in your application, leverage `types.refinement` to actively check this, for example the following definition enforces that identifiers of `Car` always start with the string `Car_`:_

```javascript
const Car = types.model("Car", {
    id: types.identifier(types.refinement(types.string, identifier => identifier.indexOf("Car_") === 0))
})
```

### **References**

References are defined by mentioning the type they should resolve to. The targeted type should have exactly one attribute of the type `identifier()`. References are looked up through the entire tree, but per type. So identifiers need to be unique in the entire tree.

### **Customizable references**

The default implementation uses the `identifier` cache to resolve references \(See [`resolveIdentifier`](https://mobx-state-tree.gitbook.io/docs/api#resolveidentifier)\). However, it is also possible to override the resolve logic, and provide your own custom resolve logic. This also makes it possible to, for example, trigger a data fetch when trying to resolve the reference \([example](https://github.com/mobxjs/mobx-state-tree/blob/cdb3298a5621c3229b3856bb469327da6deb31ea/packages/mobx-state-tree/test/reference-custom.ts#L150)\).

Example:

```javascript
const User = types.model({
    id: types.identifier(),
    name: types.string
})

const UserByNameReference = types.maybe(
    types.reference(User, {
        // given an identifier, find the user
        get(identifier /* string */, parent: any /*Store*/) {
            return parent.users.find(u => u.name === identifier) || null
        },
        // given a user, produce the identifier that should be stored
        set(value /* User */) {
            return value.name
        }
    })
)

const Store = types.model({
    users: types.array(User),
    selection: UserByNameReference
})

const s = Store.create({
    users: [{ id: "1", name: "Michel" }, { id: "2", name: "Mattia" }],
    selection: "Mattia"
})
```

#### 

