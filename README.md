![travis](https://travis-ci.org/brigand/react-mixin.svg)

Want to use ES6/CoffeeScript/TypeScript/{InsertNoun}Script classes, and mixins?

React doesn't have anything built in for this, but don't worry!  This package implements
react's mixin strategy for arbitrary objects.

Install with one of:

```sh
# recommended
npm install --save react-mixin@2

# will expose window.reactMixin or the reactMixin AMD module
curl 'wzrd.in/standalone/react-mixin@2' > vendor/react-mixin.js
```


Here's an example:

```js
var reactMixin = require('react-mixin');
var someMixin = require('some-mixin');
class Foo extends React.Component {
    render: function(){ return <div /> }    
}
reactMixin(Foo.prototype, someMixin);
reactMixin(Foo.prototype, someOtherMixin);
```

---

## Aside: Do I need mixins?

Less and less each week. I expect that by December mixin usage will be nearly 0 due to [high order components][hoc-article]. Most of the common packages have already moved away from mixins. This package is an *upgrade path*, not something you should use long term.

[hoc-article]: https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750

---


## Class level behavior

Many of the things that were regular properties in createClass are now static properties of the class.  To have things like getDefaultProps, propTypes, and getInitialState working correctly you need to apply react-mixin a level higher than the prototype: the class itself.

```js
var mixin = {
  getDefaultProps: function(){
    return {b: 2};
  }
};

class Foo {
  static defaultProps = {
    a: 1
  };
  render(){
    console.log(this.props); // {a: 1, b: 2}
  }
}

reactMixin.onClass(Foo, mixin);
```

## But it's at the end of the file!

For more readability, there is an es7 decorator proposal.  With the latest babel version and the stage config option set to 0 or 1, you can use decorators.

```js
@reactMixin.decorate(mixin)
class Foo {
  static defa...
}
```

This is a very thin wrapper.

```js
  reactMixin.decorate = function(mixin) {
    return function(reactClass) {
      return reactMixin.onClass(reactClass, mixin);
    };
  }
```

## Differences from createClass

@ndout resolved the differences by adding `reactMixin.onClass`.  If there are any more incompatibilites, **other than autobinding methods which is intentionally omitted**, please create an issue.

---

That's pretty much it.  You get errors instead of silently overwriting things, like in react,
with the exception of things whitelisted in index.js as type MANY, MANY_MERGED (getDefaultProps/getInitialState).

Autobinding is done by React.createClass, and there's no equivilent in ES6 classes.  This also has better performance (I think), but you do lose some convenience.  You can explicitly bind things in the constructor or componentWillMount.  On the main class, the constructor replaces componentWillMount.

```js
class Foo extends React.Component {
    constructor(props){
        super(props);
        this.handleChange = this.handleChange.bind(this);
    }
    ...
}
```

## But... autobinding!

If you need autobinding because a mixin depends on it, you can bind the needed methods in the constructor, or do something like this (haven't given it much thought, suggestions welcome).

```js
function autobind(methodNames){
    return {
        componentWillMount: function(){
            methodNames.forEach((name) => {
                this[name] = this[name].bind(this);
            });
        }
    };
}

@reactMixin.decorate(mixin)
@reactMixin.decorate(autobind(Object.keys(mixin)))
class Foo {
  ...
}
```

Like this but want to use it outside of react?  See [smart-mixin][1] and define your own mixin spec.

## Should I use this?

Avoid mixins if you can, but if you can't, this seems to be the best way to get them with es6 classes.

Should you use es6 classes for react components? It seems to be the future with createClass becoming legacy. It's best if everyone uses one pattern for better or worse.

That said, `createClass` isn't going anywhere in the near future.

[1]: https://github.com/brigand/smart-mixin

