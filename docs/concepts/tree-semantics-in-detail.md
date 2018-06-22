# Tree semantics in detail

MST trees have very specific semantics. These semantics purposefully constrain what you can do with MST. The reward for that is all kinds of generic features out of the box like snapshots, replayability, etc... If these constraints don't suit your app, you are probably better of using plain mobx with your own model classes. Which is perfectly fine as well.

1. Each object in a MST tree is considered a _node_. Each primitive \(and frozen\) value is considered a _leaf_.
2. MST has only three types of nodes; _model_, _array_, and _map_.
3. Every _node_ tree in a MST tree is a tree in itself. Any operation that can be invoked on the complete tree can also be applied to a sub tree.
4. A node can only exist exactly _once_ in a tree. This ensures it has a unique, identifiable position.
5. It is however possible to refer to another object in the _same_ tree by using _references_
6. There is no limit to the number of MST trees that live in an application. However, each node can only live in exactly one tree.
7. All _leaves_ in the tree must be serializable; it is not possible to store, for example, functions in a MST.
8. The only free-form type in MST is frozen; with the requirement that frozen values are immutable and serializable so that the MST semantics can still be upheld.
9. At any point in the tree it is possible to assign a snapshot to the tree instead of a concrete instance of the expected type. In that case an instance of the correct type, based on the snapshot, will be automatically created for you.
10. Nodes in the MST tree will be reconciled \(the exact same instance will be reused\) when updating the tree by any means, based on their _identifier_ property. If there is no identifier property, instances won't be reconciled.
11. If a node in the tree is replaced by another node, the original node will die and become unusable. This makes sure you are not accidentally holding on to stale objects anywhere in your application.
12. If you want to create a new node based on an existing node in a tree, you can either `detach` that node, or `clone` it.

These egghead.io lessons nicely leverage the specific semantics of MST trees:

[egghead.io lesson 6: Build Forms with React to Edit mobx-state-tree Models](https://egghead.io/lessons/react-build-forms-with-react-to-edit-mobx-state-tree-models) 

[egghead.io lesson 7: Remove Model Instances from the Tree](https://egghead.io/lessons/react-remove-model-instances-from-the-tree) [egghead.io lesson 8: Create an Entry Form to Add Models to the State Tree](https://egghead.io/lessons/react-create-an-entry-form-to-add-models-to-the-state-tree)

