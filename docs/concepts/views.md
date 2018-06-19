# Views

[egghead.io lesson 4: Derive Information from Models Using Views](https://egghead.io/lessons/react-derive-information-from-models-using-views)

Any fact that can be derived from your state is called a "view" or "derivation". See the [Mobx concepts & principles](https://mobx.js.org/intro/concepts.html) for some background.

Views come in two flavors. Views with arguments and views without arguments. The latter are called computed values, based on the [computed](https://mobx.js.org/refguide/computed-decorator.html) concept in mobx. The main difference between the two is that computed properties create an explicit caching point, but further they work the same and any other computed value or Mobx based reaction like [`@observer`](https://mobx.js.org/refguide/observer-component.html) components can react to them. Computed values are defined using _getter_ functions.

Example:

```javascript
import { autorun } from "mobx"

const UserStore = types
    .model({
        users: types.array(User)
    })
    .views(self => ({
        get numberOfChildren() {
            return self.users.filter(user => user.age < 18).length
        },
        numberOfPeopleOlderThan(age) {
            return self.users.filter(user => user.age > age).length
        }
    }))

const userStore = UserStore.create(/* */)

// Every time the userStore is updated in a relevant way, log messages will be printed
autorun(() => {
    console.log("There are now ", userStore.numberOfChildren, " children")
})
autorun(() => {
    console.log("There are now ", userStore.numberOfPeopleOlderThan(75), " pretty old people")
})
```

If you want to share volatile state between views and actions, use `.extend` instead of `.views` + `.actions`, see the [volatile state](https://mobx-state-tree.gitbook.io/docs/concepts/volatile-state) section.

