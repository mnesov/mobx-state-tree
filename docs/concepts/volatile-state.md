# Volatile state

## Volatile state

[egghead.io lesson 15: Use Volatile State and Lifecycle Methods to Manage Private State](https://egghead.io/lessons/react-use-volatile-state-and-lifecycle-methods-to-manage-private-state)

MST models primarily aid in storing _persistable_ state. State that can be persisted, serialized, transferred, patched, replaced etc. However, sometimes you need to keep track of temporary, non-persistable state. This is called _volatile_ state in MST. Examples include promises, sockets, DOM elements etc. - state which is needed for local purposes as long as the object is alive.

Volatile state \(which is also private\) can be introduced by creating variables inside any of the action initializer functions.

Volatile is preserved for the life-time of an object, and not reset when snapshots are applied etc. Note that the life time of an object depends on proper reconciliation, see the [how does reconciliation work?](https://mobx-state-tree.gitbook.io/docs/faq#how-does-reconciliation-work) section below.

The following is an example of an object with volatile state. Note that volatile state here is used to track a XHR request, and clean up resources when it is disposed. Without volatile state this kind of information would need to be stored in an external WeakMap or something similar.

```javascript
const Store = types.model({
        todos: types.array(Todo),
        state: types.enumeration("State", ["loading", "loaded", "error"])
    })
    .actions(self => {
        const pendingRequest = null // a Promise

        function afterCreate() {
            self.state = "loading"
            pendingRequest = someXhrLib.createRequest("someEndpoint")
        }

        function beforeDestroy() {
            // abort the request, no longer interested
            pendingRequest.abort()
        }

        return {
            afterCreate,
            beforeDestroy
        }
    })
```

Some tips:

1. Note that multiple `actions` calls can be chained. This makes it possible to create multiple closures with their own protected volatile state.
2. Although in the above example the `pendingRequest` could be initialized directly in the action initializer, it is recommended to do this in the `afterCreate` hook, which will only once the entire instance has been set up \(there might be many action and property initializers for a single type\).
3. The above example doesn't actually use the promise. For how to work with promises / asynchronous flows, see the [asynchronous actions](https://mobx-state-tree.gitbook.io/docs/concepts/actions#asynchronous-actions) section above.
4. It is possible to share volatile state between views and actions by using `extend`. `.extend` works like a combination of `.actions` and `.views` and should return an object with a `actions` and `views` field:

```javascript
const Todo =  types.model({}).extend(self => {
    let localState = 3

    return {
        views: {
            get x() {
                return localState
            }
        },
        actions: {
            setX(value) {
                localState = value
            }
        }
    }
})
```

### model.volatile

In many cases it is useful to have volatile state that is _observable_ \(in terms of Mobx observables\) and _readable_ from outside the instance. In that case, in the above example, `localState` could have been declared as `const localState = observable.box(3)`. Since this is such a common pattern, there is a shorthand to declare such properties, and the example above could be rewritten to:

```javascript
const Todo =  types.model({})
    .volatile(self => ({
        localState: 3
    }))
    .actions(self => ({
        setX(value) {
            self.localState = value
        }
    }))
```

The object that is returned from the `volatile` initializer function can contain any piece of data, and will result in an instance property with the same name. Volatile properties have the following characteristics:

1. The can be read from outside the model \(if you want hidden volatile state, keep the state in your closure as shown in the previous section\)
2. The volatile properties will be only observable be [observable _references_](https://mobx.js.org/refguide/modifiers.html). Values assigned to them will be unmodified and not automatically converted to deep observable structures.
3. Like normal properties, they can only be modified through actions
4. Volatile props will not show up in snapshots, and cannot be updated by applying snapshots
5. Volatile props are preserved during the lifecycle of an instance. See also [reconciliation](https://mobx-state-tree.gitbook.io/docs/faq#how-does-reconciliation-work)
6. Changes in volatile props won't show up in the patch or snapshot stream
7. It is currently not supported to define getters / setters in the object returned by `volatile`



