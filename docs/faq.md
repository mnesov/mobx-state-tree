# FAQ

## When not to use MST?

MST provides access to snapshots, patches and interceptable actions. Also, it fixes the `this` problem. All these features have a downside as they incur a little runtime overhead. Although in many places the MST core can still be optimized significantly, there will always be a constant overhead. If you have a performance critical application that handles a huge amount of mutable data, you will probably be better off by using 'raw' MobX, which has a predictable and well-known performance and much less overhead.

Likewise, if your application mainly processes stateless information \(such as a logging system\), MST won't add much value.

## Where is the `any` type?

MST doesn't offer an any type because it can't reason about it. For example, given a snapshot and a field with `any`, how should MST know how to deserialize it? Or apply patches to it? Etc. etc. If you need `any` there are following options

1. Use `types.frozen`. Frozen values need to be immutable and serializable \(so MST can treat them verbatim\)
2. Use volatile state. Volatile state can store anything, but won't appear in snapshots, patches etc.
3. If your type is regular, and you just are too lazy to type the model, you could also consider generating the type at runtime once \(after all, MST types are just JS...\). But you will loose static typing and any confusion it causes is up to you to handle :-\).

## How does reconciliation work?

* When applying snapshots, MST will always try to reuse existing object instances for snapshots with the same identifier \(see `types.identifier()`\).
* If no identifier is specified, but the type of the snapshot is correct, MST will reconcile objects as well if they are stored in a specific model property or under the same map key.
* In arrays, items without an identifier are never reconciled.

If an object is reconciled, the consequence is that localState is preserved and `postCreate` / `attach` life-cycle hooks are not fired because applying a snapshot results just in an existing tree node being updated.

## Creating async flows

See [creating asynchronous flow](https://mobx-state-tree.gitbook.io/docs/creating-asynchronous-actions).

## Using mobx and mobx-state-tree together

[egghead.io lesson 5: Render mobx-state-tree Models in React](https://egghead.io/lessons/react-render-mobx-state-tree-models-in-react)

Yep, perfectly fine. No problem. Go on. `observer`, `autorun` etc. will work as expected.

## Should all state of my app be stored in `mobx-state-tree`?

No, or, not necessarily. An application can use both state trees and vanilla MobX observables at the same time. State trees are primarily designed to store your domain data, as this kind of state is often distributed and not very local. For local component state, for example, vanilla MobX observables might often be simpler to use.

## Can I use Hot Module Reloading?

[egghead.io lesson 10: Restore the Model Tree State using Hot Module Reloading when Model Definitions Change](https://egghead.io/lessons/react-restore-the-model-tree-state-using-hot-module-reloading-when-model-definitions-change)

Yes, with MST it is pretty straight forward to setup hot reloading for your store definitions, while preserving state. See the [todomvc example](https://github.com/mobxjs/mobx-state-tree/blob/745904101fdaeb51f16f40ebb80cd7fecf742572/packages/mst-example-todomvc/src/index.js#L60-L64)

## TypeScript & MST

TypeScript support is best-effort, as not all patterns can be expressed in TypeScript. But except for assigning snapshots to properties we get pretty close! As MST uses the latest fancy Typescript features it is recommended to use TypeScript 2.3 or higher, with `noImplicitThis` and `strictNullChecks` enabled.

We recommend using TypeScript together with MST, but since the type system of MST is more dynamic than the TypeScript system, there are cases that cannot be expressed neatly and occassionally you will need to fallback to `any` or manually adding type annotations.

### **Using a MST type at design time**

When using models, you write an interface, along with its property types, that will be used to perform type checks at runtime. What about compile time? You can use TypeScript interfaces to perform those checks, but that would require writing again all the properties and their actions!

Good news! You don't need to write it twice! Using the `typeof` operator of TypeScript over the `.Type` property of a MST Type will result in a valid TypeScript Type!

```javascript
const Todo = types.model({
        title: types.string
    })
    .actions(self => ({
        setTitle(v: string) {
            self.title = v
        }
    }))

type ITodo = typeof Todo.Type // => ITodo is now a valid TypeScript type with { title: string; setTitle: (v: string) => void }
```

Due to the way typeof operator works, when working with big and deep models trees, it might make your IDE/ts server takes a lot of CPU time and freeze vscode \(or others\) A partial solution for this is to turn the `.Type` into an interface.

```javascript
type ITodoType = typeof Todo.Type;
interface ITodo extends ITodoType {};
```

### **Snapshot types are limited**

Until conditionally mapped types are available \(scheduled for TS 2.8\), the types of snapshots cannot be inferred correctly. But you will get some type assistence when using `getSnaphot` with types, like this:

```javascript
const snapshot = getSnapshot<typeof Car.SnapshotType>(car)
```

Tip: recycle the interface

```javascript
type ICarSnapshot = typeof Car.SnapshotType
const snapshot = getSnapshot<ICarSnapshot>(car)
```

For lazy folks:

```javascript
const snapshot = getSnapshot<any>(car)
```

_note: Even when typing snapshots, they will still not be as accurate as they could be until TS 2.8_

### **Typing self in actions and views**

The type of `self` is what `self` was **before the action or views blocks starts**, and only after that part finishes, the actions will be added to the type of `self`.

Sometimes you'll need to take into account where your typings are available and where they aren't. The code below will not compile: TypeScript will complain that `self.upperProp` is not a known property. Computed properties are only available after `.views` is evaluated.

For example:

```javascript
const Example = types
  .model('Example', {
    prop: types.string,
  })
  .views(self => ({
    get upperProp(): string {
      return self.prop.toUpperCase();
    },
    get twiceUpperProp(): string {
      return self.upperProp + self.upperProp; // Compile error: `self.upperProp` is not yet defined
    },
  }));
```

You can circumvent this situation by declaring the views in two steps:

```javascript
const Example = types
  .model('Example', { prop: types.string })
  .views((self: typeof Example.Type) => { // override the inferred type of self
      const views = {
        get upperProp(): string {
            return self.prop.toUpperCase();
        },
        get twiceUpperProp(): string {
            return views.upperProp + views.upperProp;
        }
      }
      return views
  }))
```

Note that you can also declare multiple `.views` block, in which case the `self` parameter gets extended after each block

```javascript
const Example = types
  .model('Example', { prop: types.string })
  .views(self => {
    get upperProp(): string {
      return self.prop.toUpperCase();
    },
  }))
  .views(self => ({
    get twiceUpperProp(): string {
      return self.upperProp + self.upperProp;
    },
  }));
```

Similarly, when writing actions or views one can use helper functions:

```javascript
import { types, flow } from "mobx-state-tree"

const Example = types
  .model('Example', { prop: types.string })
  .actions(self => {
    // Don't forget that async operations HAVE
    // to use `flow( ... )`.
    const fetchData = flow(function *fetchData() {
      yield doSomething()
    })

    return {
      fetchData,
      afterCreate() {
        // Notice that we call the function directly
        // instead of using `self.fetchData()`. This is
        // because Typescript doesn't know yet about `fetchData()`
        // being part of `self` in this context.
        fetchData()
      }
    }
  });
```

### **Snapshots can be used to write values**

Everywhere where you can modify your state tree and assign a model instance, you can also just assign a snapshot, and MST will convert it to a model instance for you. However, that is simply not expressible in static type systems atm \(as the type written to a value differs to the type read from it\). So the only work around is to upcast to `any`:

```javascript
const Task = types.model({
    done: false
})
const Store = types.model({
    tasks: types.array(Task),
    selection: types.maybe(Task)
})

const s = Store.create({ tasks: [] })
// `{}` is a valid snapshot of Task, and hence a valid task, MST allows this, but TS doesn't, so need any cast
s.tasks.push({} as any)
s.selection = {} as any
```

### **Known Typescript Issue 5938**

Theres a known issue with typescript and interfaces as described by: [https://github.com/Microsoft/TypeScript/issues/5938](https://github.com/Microsoft/TypeScript/issues/5938)

This rears its ugly head if you try to define a model such as:

```javascript
import { types } from "mobx-state-tree"

export const Todo = types.model({
    title: types.string
});

export type ITodo = typeof Todo.Type
```

And you have your tsconfig.json settings set to:

```javascript
{
  "compilerOptions": {
    ...
    "declaration": true,
    "noUnusedLocals": true
    ...
  }
}
```

Then you will get errors such as:

> error TS4023: Exported variable 'Todo' has or is using name 'IModelType' from external module "..." but cannot be named.

Until Microsoft fixes this issue the solution is to re-export IModelType:

```text
import { types, IModelType } from "mobx-state-tree"

export type __IModelType = IModelType<any,any>;

export const Todo = types.model({
    title: types.string
});

export type ITodo = typeof Todo.Type
```

It ain't pretty, but it works.

### **Optional/empty maps**

Optional parameters, including "empty" maps, should be either a valid snapshot or a MST instance. To fix type errors such as `Error while converting {} to map`, define your type as such:

```javascript
  map: types.optional(types.map(OtherType), {})
```

## How does MST compare to Redux

So far this might look a lot like an immutable state tree as found for example in Redux apps, but there're are only so many reasons to use redux as per [article linked at the very top of redux guide](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367) that MST covers too, meanwhile:

* Like Redux, and unlike MobX, MST prescribes a very specific state architecture.
* mobx-state-tree allows direct modification of any value in the tree; it is not necessary to construct a new tree in your actions.
* mobx-state-tree allows for fine-grained and efficient observation of any point in the state tree.
* mobx-state-tree generates JSON patches for any modification that is made.
* mobx-state-tree provides utilities to turn any MST tree into a valid Redux store.
* Having multiple MSTs in a single application is perfectly fine.

