---
title: "The Initial State Gotcha"
categories:
  - "React"
  - "Quick Tip"
---

I ran into an issue recently while refactoring an existing React component in our application at work. Part of my changes to the component included wrapping it in a container component, so it could be updated by passing new props to it.

The use case was that of a form that offered a choice of available locations for visitors. When the form first rendered, the visitor was assumed to be checking in as a regular, and shown a set of a available locations opened to regular visitors. If the user decided they wanted to check in as a volunteer, however, we needed to fetch a different set of locations that they could check in to, thus re-rendering the child component with new props.

Everything seemed to be wired correctly. When the visitor changed the check in kind, an ajax request was fired to fetch the new locations, which were then passed to the child component as props. However, there was no change in what was rendered by the child component. It continued to show the initial set of locations. Why?

After examining the child component closely, I realized that it was setting its state from props in a `getInitialState()` method, kind of like this:

```javascript
getInitialState() {
  return {
    someStateValues: this.calculateValues(this.props.initialValues)
  }
}
```
Some other pieces of state where updated as the user interacted with the form to select a specific location to check in to, but the final filtered collection of opened locations that the form rendered was only set as a state variable inside that method call.

Problem is `getInitialState()` is only called when the component is first mounted, but not when it receives new props and should be updated, so the filtered collection of open locations for check in was never recalculated. That piece of state never changed.

The solution I found for this was to use the `componentWillReceiveProps()` lifecycle method to recalculate the state when the child component received new props. Like this:

```javascript
componentWillReceiveProps(nextProps) {
  if (this.props.initialValues !== nextProps.initialValues) {
    this.setState({
      someStateValues: this.calculateValues(nextProps.initialValues)
    })
  }
}
```

This is something good to keep in mind if you're setting state from props for some reason.
