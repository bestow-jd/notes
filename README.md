

# Notes, Patterns, and Best Practices

## React Stuff

### Render Props vs Rendering Children

Passing a render function to 

* Docs: https://reactjs.org/docs/render-props.html

* Ex: 

  * ```
    <Mouse render={mouse => (
              <Cat mouse={mouse} />
            )}/>
    ```

### Composite Components

* Composite components are HTML-style components that allow you to pass other specialized components in as children. 
* Much, much better pattern than using

### HOCs

* `WrappedComponent.name` should be avoided, if you need the name of a component, pass it in as an extra param to the HOC.

  * It is unsafe because if you pass an HOC into that HOC it wont be the name of the OG component which is probably the intended name so it presents a pitfall. Answer from: https://stackoverflow.com/questions/50967215/get-name-of-wrapped-component-from-its-higher-order-component

  * Bad:

    ```
    withSomeHOC = (WrappedComponent) => {
        const nameText = WrappedComponent.name;
        <WrappedComponent text={nameText} />
    }
    ```

  * Good:

    ```
    withSomeHOC = nameText => (WrappedComponent) => {
        <WrappedComponent text={nameText} />
    }
    ```

* Using it in a debug setting is fine (like in an error catching situation), but dont rely on it being what you want.

### Memos

* Memos are the same concept as `React.PureComponent` but for functional components
* Compatibility:
  * Breaks enzyme's `mount` so we are currently unable to use it

### Props Spreading on Components

* "With great power comes great responsibility" - calvin

* "Its like giving your devs a loaded gun. Can be an effective tool, but they also might shoot themselves with it"

* Can create to hard to track down bugs

* However, without props spreading it takes away power and flexibility from the component usage

* Dont do blanket object props spreading like:

  ```props => (<Component {...props}/>)```

* `...rest` spreading combined with destructuring it can be okay, so long as you are defining other props. This makes it clear to other devs that it was intentional and separates 

  ```({prop1, ...rest}) => (<Component someProp={prop1} {...rest}/>)```

* Order matters, so a blanket props spread can be used more conservatively at the beginning of a component rather than the end (so that it can be overwritten, rather than the spread doing the overwriting) 

* *Conclusion*: be very intentional with it, heavily weigh potential downsides

## Javascript Stuff

### Destructuring

Destructuring is an amazing tool to aid in clarity and readability of code. Recommended to use whenever possible, such as in functions, components and retrieving variables from an object.

* Benefits:

  * Clear API definition of a function parameters/component props rather than ambiguous objects
  * Easier to introduce Typescript to later
  * Forces dev to make a conscious decision about including additional props
  * Clearer code throughout the function, no need to throw `props` around everywhere
  * Less object interaction that can cause null property errors (like when props is null)

* Potential Downsides/Pitfalls:

  * Can have funky run-time interactions with arrays (especially if an array is passed and an object was expected). This is a pitfall because the function def is abstracted away, so someone ~~dumb~~ unwitting could accidentally pass in an array to a desctructured function and the function will run but have weird errors. This is greatly improved with Typescript since you wont be able to do it as easily, but still could happen on a dynamic run-time variable (like something interpreted from JSON).

    * Ex. `let [x, y] = ['a', 'b'];` results in `x='a'` and `y='b'` 

    * Example bad code that wouldnt be caught by compiler/IDEs (without Typescript):

    * ```
      const foo = ({text, someObj}) => (
      	{
          upperCasedText:text.upperCase(), 
          someValue: someObj.value
      	}
      )
      foo([{value: '=('}, "2"])
      ```

  * Easy to be lazy and continue to pass  `...rest` everywhere (but still better than just `props`)

* Usage:

  * Bad:

    * No destructuring

      * ```const foo = (props) => (<div> props.text </div>)```

    * Non-component/pure function usage with dynamic `...rest` params: 

      * ```const foo = ({text, ...rest}) => ({upperCasedText: text.upperCase()})```

        ​	NOTE: I forget why this is bad?

  * Good:

    *  props usage:
       * ```const foo = ({text}) => (<div> text </div>)```
    *  props usage With a  `...rest` param
       * ```const foo = ({text, ...rest}) => (<div {...rest}> text </div>)```

## Project Layout

### Constants

##### `constants.js` pattern

* Component specific constants should be named `constants.js` alongside whatever component they belong to like:

  * ```
    ComponentA/
    	ComponentA.js
    	constants.js
    ```

* JSON constants are fine, but less preferred

* Potential Downsides:

  *  naming will rely on the component directory to describe it, so if the directory above is unclear it becomes messy
  *  Annoying on your IDE since you can have plenty of `constants.js` files open at the same time

##### `{descriptive-name}.constant.js` pattern

* 