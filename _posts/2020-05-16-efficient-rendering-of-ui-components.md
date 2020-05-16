---
layout: post
title:  "Efficient rendering of UI components"
date:   2020-05-16
---

Recently, I started working on an idle game with a friend of mine. But to challange ourselves we decided that we will not use any framework or library and implement everything we need by ourselves. 

As an additional challenge I decided to use some coding practices from [ClojureScript](https://clojurescript.org/) as I started coding in it for work. Some of the practices I decided to use is that everything but the state of the game will be immutable and that rendering will work as a function of the state.

We started coding the rendering functions, and it looked something like this:

```javascript
function renderPlayerHP(store) {
    const player = getPlayerFromState(store.getState());
    document.getElementById("player-hp-value").innerText = player.hp;
}

function renderPlayerLevel(store) {
    const player = getPlayerFromState(store.getState());
    document.getElementById("player-level").innerText = player.level;
}

export function renderGame(store) {
    renderPlayerHP(store);
    renderPlayerLevel(store);
}
```

and the `renderGame` function is called every time the state changes.

Seems pretty simple, but there are several problems with this approach. First of all, this is very not performant. Accessing the DOM is one of the slowest operations to do in Javascript and generating the entire game DOM everytime something changes in the state is a waste of processing power and god knows that state changes *a lot* in idle games. The second problem with this approach is that the elements in the DOM can be recreated in the middle of user interaction. Imagine a draggable object that the user is trying to drag and at the same time player's HP changes, the draggable object will be recreated in the original location and the user will have to try and drag it (and pray state doesn't change again while he's dragging it).

[React](https://reactjs.org/) has a good aproach to solving these problems, it uses a virtual DOM, changes it when state changes and checks of something in the real DOM needs to be updated accordingly. This solution is very good, but not that simple to implement without using libraries or frameworks so it's not good for us. 

One of the most popular UI frameworks in ClojureScript is called [Reagent](https://reagent-project.github.io/), it's a framework built on top of react and it has a little bit different approach to checking whether the DOM should be rerendered or not. It checkes if the state has changed, and only if indeed changed it will rerender the needed components. But, how does it know which components to rereneder? We don't want to render player's level every time the HP changes. 

Like I said before, in ClojureScript all data structures are immutable, but there is one data structure called [Atoms](https://clojure.org/reference/atoms) that can be mutated (I told several lies in the last sentence, but it's just to simplify things). So Reagent created their own data structure on top of Atoms (and called them Atoms too, so it's not confusing at all) and it can monitor which properties of the atom were accessed whithin a certain rendering function. This way, Reagent knows to run the rendering function only when the relevant part of the state changes. For example, in our example above, if only the player's HP changes, the function that renders the level will not run.

How do we implement this behavior in Javascript? Actually, it's not that complicated - with [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy).  Proxies wrap Javascript objects, and allows calling functions whenever a property of that object is being access (either for getting it or setting it). With some help from [StackOverflow](https://stackoverflow.com/questions/43177855/how-to-create-a-deep-proxy), I created the following function:

```javascript
function  deepProxy(onAccess, target) {
    return  new  Proxy(target, {
        get(target, property) {
            onAccess(property);
            const item = target[property];
            if (item && typeof item === 'object') {
                return  deepProxy(onAccess.bind(null, property), item);
            }
			return  item;
		}

	});
}
```
This function creates a proxy object and calls a callback every time one if its properties are accessed. In addition, if the value of that property is also an object, it wraps it the same way, so if one if its properties is accessed the callback will run too.

Let's see how it works:

```javascript
const someObj = {someProp: "some value", anotherProp: {innerProp: "inner value"}}; // We create an object
const proxiedObj = deepProxy(function() { console.log(Array.from(arguments)); }, someObj); // Wrap it in a proxy with a callback that will print all arguments that are passed to it
```

Now, let's try accessing the properties on the proxied object:

```javascript
proxiedObj.someProp // -> "some value", but also ["someProp"] is printed to the console
proxiedObj.anotherProp.innerProp // -> "inner value", but also ["anotherProp"] and ["anotherProp", "innerProp"] are printed to the console
```

Cool! Now we can spy on which attributes are being accessed on the state in order to render a speicific component. To do that, we can write the following code:

```javascript
// to store the dependencies of each rendering function
const renderersDependencies = {};

export function renderWhenNeeded(name, func, prevState, newState) {
    const fDeps = renderersDependencies[name] || [];

    // Wrap the state in a proxy 
    // and store properties that were used in renderersDependencies
    const proxiedNewState = deepProxy(function() {
        const currentDeps = renderersDependencies[name] || [];
        const path = Array.from(arguments).join(".");
        currentDeps.push(path);
        const newDeps = Array.from(new Set(removeSubstrings(currentDeps)));
        renderersDependencies[name] = newDeps;
    }, newState);

	// if it's the first time we run the rendering function 
	// or the relevant state had changed, redraw the component
    if(fDeps.length === 0 || isStateChanged(prevState, newState, fDeps)) {
        func(proxiedNewState);
    }
}


function removeSubstrings(strings) {
    return strings.filter(str => {
        return !strings.some(biggerString => {
            return biggerString.startsWith(str + ".")
        });
    });
}

function getProp(obj, propPath) {
    return propPath.split(".").reduce((o, prop) => o[prop], obj);
}

function isStateChanged(prevState, newState, props) {
    return props.some(prop => {
        return getProp(prevState, prop) !== getProp(newState, prop)
    });
}
```
And that's it! We can use the `renderWhenNeeded` function like so:

```javascript
export function renderGame(store) {
    const renderers = {
        renderPlayerHP,
        renderPlayerXP,
        renderPlayerGold,
        renderPlayerStrength,
        renderPlayerAgility,
        renderPlayerDefense,
        renderInventory
    }

    Object.keys(renderers).forEach(renderer => {
        renderWhenNeeded(renderer, renderers[renderer], store.getPrevState(), store.getState());
    });
}
```

