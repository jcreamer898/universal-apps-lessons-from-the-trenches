name: intro
class: center, middle

# Universal Apps Lessons from the Trenches
#### http://bit.ly/universal-app-lessons

---
class: center, middle

# whoami

### Jonathan Creamer

<img src="images/family.jpg" style="width: 80%;" />

---

# whoami

* Currently Senior Front End Engineer at [Lonely Planet](http://lonelyplanet.com)
* Past JavaScript Engineer appendTo
* Nashville, TN

<img src="images/lonelyplanet_bw.png" style="width: 10em" />

* Love JavaScript, tweet at [@jcreamer898](http://twitter.com/jcreamer898), blog at [jonathancreamer.com](http://jonathancreamer.com)
* [Microsoft MVP](https://mvp.microsoft.com/en-us/MyProfile/Preview?previewAs=Public)

???

class: center, middle

---

### Agenda

1. What is this Universal App thing all about?
1. How to get started
1. Back End
1. Front End
1. Tips n' Tricks

???

- Middleware for react router
- Can you even universal app bro?
- Unit test EVERY SINGLE state change
- Be careful passing anonymous functions
- Use Enzyme
- Memoize selectors to be sure you don’t change more state than you realize
- Passing in state through props is an anti-pattern
- Don't over use route params with react router because you can rightly couple logic to your business logic
- Use flow
- How to fetch data on route changes
- React Developer Tools
- Dotenv

---
class: center, middle

# What is this Universal App thing all about?

---
class: center, middle

### Universal React Apps

![](images/jsall.jpg)

???

node
then react

---

### Can you even universal app bro?

* node.js
* Share client/server, Backbone, etc...
* react renderToString
* ~~Isomorphic~~
* One code base
* Same components on client and server
* JavaScript everywhere

???

Who's heard of "Isomorphic"? Universal?
Who's actually USING it?

---

### Progressive enhancement

* SEO
* User feedback
* Not just about "no javascript"

---

### Progressive enhancement

<img src="images/progressive.png" style="width:60%" />

* < 2s
* Give users quick feedback

---

### SPA

* Everybody loves a good spa
* Why not get a bit of both?
* Render on the server
* Then go to the SPA

---
class: center, middle

# Getting Started

---

class: center, middle

### When you google for React Universal Starters...

![](images/loud.gif)

---

### Getting started

* Lots and lots of starters

--
* lots...
--

* https://github.com/kriasoft/react-starter-kit
--

* and lots...
--

* https://github.com/davezuko/react-redux-starter-kit
--

* and lots...
--

* https://github.com/facebookincubator/create-react-app
--

* and lots...
--

* https://github.com/erikras/react-redux-universal-hot-example
--

* ...
---
class: center, middle

### How it feels to look at React Starters...

![](images/firehose.gif)

---

### Learn the basics

<blockquote data-conversation="none" data-lang="en"><p lang="en" dir="ltr">Learn React in the right order: <a href="https://t.co/RhzRGzEIe0">https://t.co/RhzRGzEIe0</a> <a href="https://t.co/uVdrYW2dbo">pic.twitter.com/uVdrYW2dbo</a></p>&mdash; Dan Abramov (@dan_abramov) <a href="https://twitter.com/dan_abramov/status/703214489387327488">February 26, 2016</a></blockquote>

???

Avoid unnecessary complexity

---
class: center, middle

# Frontend

---

### Start simple...ish

* What do you "need"
* babel, babel-preset-es2015, babel-preset-react
* react, react-dom
* react-router
* webpack, babel-loader
* redux, react-redux, helmet (optional)

---

### Start with an App component

```js
import React from "react";
import { connect } from "react-redux";

export default class App extends React.Component {
  static propTypes = {
    title: React.PropTypes.string,
    children: React.PropTypes.node,
  };

  render() {
    // ...
  }
}

const mapStateToProps = state => ({ title: state.title });
const connected = connect(mapStateToProps)(App);
export { connected };
```

* Default export for testing, connected for render

---

### App Render

```js
import Helmet from "react-helmet";

render() {
  const { children, title } = this.props;

  return (
    <div className="App">
        <Helmet title={title} />
        {children}
    </div>
  );
}
```

* Use React Helmet for meta

---

### Containers

```js
export default function Home() {
  return (
    <div className="Home">
      // ...
    </div>
  );
}
```

---

### Routing

```js
import { connected as AppComponent } from "./components/app"
import Home from "./components/home";
import Users from "./components/users";

const routes = {
  path: "",
  component: AppComponent,
  childRoutes: [
    {
      path: "/",
      component: Home,
    },
    {
      path: `/users`,
      component: Users,
    },
  ],
};

export default routes;
```

* Use a JS Object for routes

---

### Mount the App

```js
import ReactDOM from "react-dom";
import { Router, browserHistory } from "react-router";
import { Provider } from "react-redux";
import configureStore from "./store";
import routes from "./routes";

export default function configureClient(initialState, element = "app") {
  const store = configureStore(initialState);

  ReactDOM.render(
    <Provider store={store}>
      <Router routes={routes} history={browserHistory} />
    </Provider>,
    document.getElementById(element)
  );
}
```

* Export as a function
* Provider for redux
* Router for react-router
* browserHistory uses the history npm lib

---

### configureStore

```js
import { createStore } = "redux";
import reducers from "./reducers";
import { browserHistory } from "react-router";
import { routerMiddleware } from "react-router-redux";

export default function configureStore(initialState) {
  const store = createStore(
    reducers,
    initialState,
    applyMiddleware(
      // ...,
      routerMiddleware(browserHistory),
    ),
  );

  // Optional, but nice; need typeof check for back end
  typeof window !== "undefined" && window.$$store = store;

  return store;
}
```

* Use routerMiddleware for pushing history into redux
* Add $$store for quick debugging

---

### React HAWT AMIRITE?

```js
if (process.env.NODE_ENV === "development" && module.hot) {
  module.hot.accept("./reducers", () => {
    /* eslint-disable */
    store.replaceReducer(require("./reducers").default);
    /* eslint-enable */
  });
}
```

* Hot Load w/ webpack for dev
* Requires a pretty advanced webpack setup
* Phenomenal way to dev

---
class: center, middle

# Backend

---

### Express

```shell
npm install -g express-generator
```

* Great place to start
* Move all server code into a server folder
* Use babel to transpile

```json
"scripts": {
  "build": "npm run build:server && npm run build:client",
  "build:client": "webpack && babel -d dist/app app",
  "build:server": "babel -d dist/server app"
}
```

???

Update ./bin/www to point to server/app in dev, dist/server/app in prod
Allows you to use ALL latest features regardless of node version

---
### Super basic backend

```js
import { renderToString } from "react-dom/server";
import App from "../public/javascripts/components/app";
import React from "react";

/* GET home page. */
router.get("/", function(req, res) {
  // This is the magic sauce
  const markup = renderToString(<App />);

  res.render("index", {
    title: "Express",
    markup
  });
});
```

* Express? Hapi? ...?
* ReactDOM/renderToString

---


### react-router on the server

```js
import { match as reactRouterMatch } from "react-router";

export default function match(routes) {
  return (req, res, next) => {
    const request = req;

    if (!routes) next(new Error("No routes passed to the matchMiddle middleware."));

    reactRouterMatch({ routes, location: req.url }, (err, redirectLocation, props) => {
      if (err) {
        next(err);
      } else if (redirectLocation) {
        res.redirect(302, redirectLocation.pathname + redirectLocation.search);
      } else if (props) {
        request.props = props;
        next();
      } else {
        next("route");
      }
    });
  };
}
```

* Need to match back end routes with clientside routes
* This enables the progressive enhancement/spa combo
* Expose `match` middleware
* Adds `props` to `request`

---

### State

```js
/* imports ... */

router.get("/", match, function(req, res) {
  const initialState = { /* ... */};

  const store = createStore(
    reducers,
    initialState,
    applyMiddleware(/*...*/)
  );

  const markup = renderToString(
  <Provider store={store}>
    <RouterContext {...req.props} />
  </Provider>
  );

  res.render("index", {
    title: "Express",
    markup,
    initialState
  });
});
```

* Call match middleware
* Setup initial state
* RouterContext allows for server rendering

---

### state

```html
<script type="text/javascript">
  window.__initialState = {{{initialState}}};
</script>
```

* HBS view engine, or even JSX, meh
* Pass initialState to redux store

---

### Universal App in Production

* http://www.lonelyplanet.com/usa/nashville/restaurants/a/poi-eat/362228
* http://www.lonelyplanet.com/usa/nashville/restaurants/hattie-bs/a/poi-eat/1513072/362228

---
class: center, middle

# Lessons learned

---

### Lessons learned

* Take it slow, Rome wasn't built in a day
* It's hard
* BUT fun and totes worth it

---

### TEST ALL TEH THINGS

```js
describe("<Button />", () => {
  it("should have an onclick", () => {
    const onClick = sinon.spy();
    const wrapper = shallow(<Button onClick={onClick} />);

    wrapper.simulate("click");
    expect(onClick.calledOnce).to.be.ok;
  });
})
```

* Use airbnb's [Enzyme](https://github.com/airbnb/enzyme)
* Sinon for mocking
* Mocha, chai, NYC + Istanbul for testing and code coverage

---

### Test EVERY state change

```js
describe("bookingReducer", () => {
  it("should check availability", () => {
    const state = someReducer({
      fetching: false,
    }, {
      type: CHECK_AVAILABILITY,
      payload: { startDate, endDate }
    });

    expect(state.fetching).to.be.ok;
  });
});
```

* Testing ALL state changes is huge
* Greatly reduces risks of bugs
---

### Test all actions

```js
describe("bookingActions", () => {
  it("should validate dates", () => {
    const startDate = moment().subtract(10, days);
    const dispatch = sinon.stub();

    fetchAvailability({ startDate, endDate })(dispatch);

    expect(dispatch.getCall(0).args[0].type).to.equal(INVALID_DATE);
  });
});
```

* Use w/ `redux-thunk`

---

### Load up that app state

```js
// DON'T
constructor(props) {
  super(props);

  this.state = { foo: props.foo };
}
render() {
  return (<h1>{this.props.foo}</h1>);
}
```
```js
// DO
render() {
  return (<h1>{this.props.foo}</h1>);
}

const mapStateToProps = state => state.foo;
const connected = connect(mapStateToProps)(Foo);
export { connected };
```

* Don't be shy with your state
* Don't pass in state through props, use react-redux

```js
import { connected as Foo } from "./foo";
```

---

### Be careful w/ mapStateToProps

```js
const getLoggedInUsers = state => state.users.map(u => u.loggedIn);
const getUserMetrics = state => ({
  loggedIn: state.users.reduce((i, u) => u.loggedIn ? i + 1 : i, 0).length,
});

const mapStateToProps = state => ({
  users: getLoggedInUsers(state),
  metrics: getUserMetrics(state),
});
```

* Will render every time!

---

### Reselect and memoize FTW

```js
const getUsers = state => state.users;
const getLoggedInUsers = createSelector([
  getUsers
], (users) => users.map(u => u.loggedIn));

const getUserMetrics = createSelector([
  getLoggedInUsers,
], (users) => ({
  loggedIn: users.length,
});

const mapStateToProps = state => ({
  users: getLoggedInUsers(state),
  metrics: getUserMetrics(state),
});
```

* Word of caution on the server

---

### Same with functions

```js
render() {
  return (
    <DatePicker
      onSelect={(dates) => dispatch(checkAvailability(dates))}
    />
  )
}
```

* This will re-render the DatePicker

---

### Same with functions

```js
constructor() {
  this.onSelectDates = this.onSelectDates.bind(this);
}
onSelectDates(dates) {
  this.props.dispatch(checkAvailability(dates));
}
render() {
  return (
    <DatePicker
      onSelect={this.onSelectDates}
    />
  )
}
```

* All better

---

### SRP Reducers

```js
// Don't
export default function ListReducer(state = {
  items = [],
  filters = {},
}, action) {
  switch (action.type) {
    case FETCH_DONE: {
      return {
        items: [...action.payload],
        filters: { ...state.filters },
      };
    }
    case FILTER_DONE: {
      return {
        items: [...state.items],
        filters: { ...action.payload.filters },
      };
    }
  }
}
```

* Too much in one

---

### SRP Reducers

```js
// Don't
export function ListReducer(state = [], action) {
  switch (action.type) {
    case FETCH_DONE: {
      return [...action.payload];
    }
  }
}

export function FilterReducer(state = {}, action) {
  switch (action.type) {
    case FILTER_DONE: {
      return { ...action.payload.filters };
    }
  }
}
```

* Easier to test
* Update independently
* Manage state separately


---

# Thanks!

<img style="width:50%" src="images/clap.gif" />

### [@jcreamer898](http://twitter.com/jcreamer898)
### [jonathancreamer.com](http://jonathancreamer.com)
