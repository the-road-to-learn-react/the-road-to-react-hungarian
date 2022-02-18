## Imperative React

React is inherently declarative. When you implement JSX, you tell React what elements you want to see, not how to create these elements. When you implement a hook for state, you tell React what you want to manage as a stateful value and not how to manage it. And when you implement an event handler, you do not have to assign a listener imperatively:

```javascript
// imperative JavaScript + DOM API
element.addEventListener('click', () => {
  // do something
});

// declarative React
const App = () => {
  const handleClick = () => {
    // do something
  };

  return (
    <button
      type="button"
      onClick={handleClick}
    >
      Click
    </button>
  );
};
```

However, there are cases when we will not want everything to be declarative. For example, sometimes you want to have imperative access to rendered elements, most often as a side-effect, in cases such as these:

* read/write access to elements via the DOM API:
  * measuring (read) an element's width or height
  * setting (write) an input field's focus state
* implementation of more complex animations:
  * setting transitions
  * orchestrating transitions
* integration of third-party libraries:
  * [D3](https://d3js.org) is a popular imperative chart library

Because imperative programming in React is often verbose and counterintuitive, we'll walk through only a small example for setting the focus of an input field imperatively. For the declarative way, simply set the input field's autofocus attribute:

{title="src/App.js",lang="javascript"}
~~~~~~~
const InputWithLabel = ({ ... }) => (
  <>
    <label htmlFor={id}>{children}</label>
    &nbsp;
    <input
      id={id}
      type={type}
      value={value}
# leanpub-start-insert
      autoFocus
# leanpub-end-insert
      onChange={onInputChange}
    />
  </>
);

# leanpub-start-insert
// note that `autoFocus` is a shorthand for `autoFocus={true}`
// every attribute that is set to `true` can use this shorthand
# leanpub-end-insert
~~~~~~~

This works, but only if one of the reusable components is rendered. For example, if the App component renders two InputWithLabel components, only the last rendered component receives the `autoFocus` on its render. However, since we have a reusable React component here, we can pass a dedicated prop which lets the developer decide whether the input field should have an active `autoFocus`:

{title="src/App.js",lang="javascript"}
~~~~~~~
const App = () => {
  ...

  return (
    <div>
      <h1>My Hacker Stories</h1>

      <InputWithLabel
        id="search"
        value={searchTerm}
# leanpub-start-insert
        isFocused
# leanpub-end-insert
        onInputChange={handleSearch}
      >
        <strong>Search:</strong>
      </InputWithLabel>

      ...
    </div>
  );
};
~~~~~~~

Again, using just `isFocused` as an attribute is equivalent to `isFocused={true}`. Within the component, use the new prop for the input field's `autoFocus` attribute:

{title="src/App.js",lang="javascript"}
~~~~~~~
const InputWithLabel = ({
  id,
  value,
  type = 'text',
  onInputChange,
# leanpub-start-insert
  isFocused,
# leanpub-end-insert
  children,
}) => (
  <>
    <label htmlFor={id}>{children}</label>
    &nbsp;
    <input
      id={id}
      type={type}
      value={value}
# leanpub-start-insert
      autoFocus={isFocused}
# leanpub-end-insert
      onChange={onInputChange}
    />
  </>
);
~~~~~~~

The feature works, yet it's still a declarative implementation. We are telling React *what* to do and not *how* to do it. Even though it's possible to do it with the declarative approach, let's refactor this scenario to an imperative approach. We want to execute the `focus()` method programmatically on the input field's element via the DOM API once it has been rendered:

{title="src/App.js",lang="javascript"}
~~~~~~~
const InputWithLabel = ({
  id,
  value,
  type = 'text',
  onInputChange,
  isFocused,
  children,
}) => {
# leanpub-start-insert
  // A
  const inputRef = React.useRef();
# leanpub-end-insert

# leanpub-start-insert
  // C
  React.useEffect(() => {
    if (isFocused && inputRef.current) {
      // D
      inputRef.current.focus();
    }
  }, [isFocused]);
# leanpub-end-insert

  return (
    <>
      <label htmlFor={id}>{children}</label>
      &nbsp;
# leanpub-start-insert
       {/* B */}
# leanpub-end-insert
      <input
# leanpub-start-insert
        ref={inputRef}
# leanpub-end-insert
        id={id}
        type={type}
        value={value}
        onChange={onInputChange}
      />
    </>
  );
};
~~~~~~~

All the essential steps are marked with comments that are explained step by step:

* (A) First, create a `ref` with **React's useRef Hook**. This `ref` object is a persistent value which stays intact over the lifetime of a React component. It comes with a property called `current`, which, in contrast to the `ref` object, can be changed.
* (B) Second, the `ref` is passed to the element's JSX-reserved `ref` attribute and thus element instance gets assigned to the changeable `current` property.
* (C) Third, opt into React's lifecycle with React's useEffect Hook, performing the focus on the element when the component renders (or its dependencies change).
* (D) And fourth, since the `ref` is passed to the element's `ref` attribute, its `current` property gives access to the element. Execute its focus programmatically as a side-effect, but only if `isFocused` is set and the `current` property is existent.

Essentially that's the whole example of how to move from declarative to imperative programming in React. In this case, it's possible to use either the declarative or imperative approach as you experienced first hand. However, it's not always possible to use the declarative approach, so the imperative approach can be performed whenever it's necessary.

### Exercises:

* Confirm your [source code](https://bit.ly/3B0qc2S).
  * Confirm the [changes](https://bit.ly/3aT3bEq).
* Read more about [refs in React](https://www.robinwieruch.de/react-ref/).
  * Optional: Learn [how to build a Slider component with imperative React](https://www.robinwieruch.de/react-slider/).
  * Optional: Learn [how to create a custom hook with a ref](https://www.robinwieruch.de/react-custom-hook-check-if-overflow/).
* Read more about [why frameworks matter](https://www.robinwieruch.de/why-frameworks-matter/).
* Optional: [Leave feedback for this section](https://forms.gle/nABoW2tKAPd1yVkv7).