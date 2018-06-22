# Actions

[egghead.io lesson 2: Attach Behavior to mobx-state-tree Models Using Actions](https://egghead.io/lessons/react-attach-behavior-to-mobx-state-tree-models-using-actions)

By default, nodes can only be modified by one of their actions, or by actions higher up in the tree. Actions can be defined by returning an object from the action initializer function that was passed to `actions`. The initializer function is executed for each instance, so that `self` is always bound to the current instance. Also, the closure of that function can be used to store so called _volatile_ state for the instance, or to create private functions that can only be invoked from the actions, but not from the outside.

```javascript
const Todo = types.model({
        title: types.string
    })
    .actions(self => {
        function setTitle(newTitle) {
            self.title = newTitle
        }

        return {
            setTitle
        }
    })
```

Or, shorter if no local state or private functions are involved:

```javascript
const Todo = types.model({
        title: types.string
    })
    .actions(self => ({ // note the `({`, we are returning an object literal
        setTitle(newTitle) {
            self.title = newTitle
        }
    }))
```

Actions are replayable and are therefore constrained in several ways:

* Trying to modify a node without using an action will throw an exception.
* It's recommended to make sure action arguments are serializable. Some arguments can be serialized automatically, such as relative paths to other nodes
* Actions can only modify models that belong to the \(sub\)tree on which they are invoked
* You cannot use `this` inside actions, instead, use `self`. This makes it safe to pass actions around without binding them or wrapping them in arrow functions.

Useful methods:

* [`onAction`](https://github.com/mobxjs/mobx-state-tree/blob/master/API.md#onaction) listens to any action that is invoked on the model or any of its descendants.
* [`addMiddleware`](https://github.com/mobxjs/mobx-state-tree/blob/master/API.md#addmiddleware) listens to any action that is invoked on the model or any of its descendants.
* [`applyAction`](https://github.com/mobxjs/mobx-state-tree/blob/master/API.md#applyaction) invokes an action on the model according to the given action description

### **Asynchronous actions**

[egghead.io lesson 12: Defining Asynchronous Processes Using Flow](https://egghead.io/lessons/react-defining-asynchronous-processes-using-flow)

Asynchronous actions have first class support in MST and are described in more detail [here](https://mobx-state-tree.gitbook.io/docs/creating-asynchronous-actions). Asynchronous actions are written by using generators and always return a promise. For a real working example see the [bookshop sources](https://github.com/mobxjs/mobx-state-tree/blob/adba1943af263898678fe148a80d3d2b9f8dbe63/examples/bookshop/src/stores/BookStore.js#L25). A quick example to get the gist:

```javascript
import { types, flow } from "mobx-state-tree"

someModel.actions(self => {
    const fetchProjects = flow(function* () { // <- note the star, this a generator function!
        self.state = "pending"
        try {
            // ... yield can be used in async/await style
            self.githubProjects = yield fetchGithubProjectsSomehow()
            self.state = "done"
        } catch (error) {
            // ... including try/catch error handling
            console.error("Failed to fetch projects", error)
            self.state = "error"
        }
        // The action will return a promise that resolves to the returned value
        // (or rejects with anything thrown from the action)
        return self.githubProjects.length
    })

    return { fetchProjects }
})
```

### **Action listeners versus middleware**

The difference between action listeners and middleware is: Middleware can intercept the action that is about to be invoked, modify arguments, return types etc. Action listeners cannot intercept, and are only notified. Action listeners receive the action arguments in a serializable format, while middleware receives the raw arguments. \(`onAction` is actually just a built-in middleware\)

For more details on creating middleware, see the [docs](https://mobx-state-tree.gitbook.io/docs/middleware)

### **Disabling protected mode**

This may be desired if the default protection of `mobx-state-tree` doesn't fit your use case. For example, if you are not interested in replayable actions, or hate the effort of writing actions to modify any field; `unprotect(tree)` will disable the protected mode of a tree, allowing anyone to directly modify the tree.

