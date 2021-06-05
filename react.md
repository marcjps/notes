
# React

### Scaffolding a new app

Run these commands to create a new app and start it running.

```text
npm install -g create-react-app
create-react-app my-app
cd my-app/
npm start
```

To make a production build:

```text
npm run build
```

Note: if you are hosting the final build in a subdirectory, you need to set the "homepage" parameter in the package.json so it points to the directory that the app is hosted in.  Here's an example package.json with the "homepage" set:

```json
{
	"name": "react-spanish-shop",
	"version": "0.1.0",
	"private": true,
	"homepage": "/shop",
	"dependencies": {
		"react": "^16.2.0",
		"react-delay": "^0.1.0",
		"react-dom": "^16.2.0",
		"react-scripts": "1.0.17",
		"react-typist": "^2.0.4"
	},
	"scripts": {
		"start": "react-scripts start",
		"build": "react-scripts build",
		"test": "react-scripts test --env=jsdom",
		"eject": "react-scripts eject"
	}
}
```

### A skeleton component

This is a skeleton component to start with.

```javascript
import React from 'react';

class ServiceSettingsForm extends React.Component {

	render = () => {
		return (
			<p>Hello world</p>
		)
	}

}

export default ServiceSettingsForm;
```


Once you have a component you can reference it like this:

```javascript
import ServiceSettingsForm from '../ServiceSettingsForm';
```

..then in render()..

```html
<ServiceSettingsForm />
```

### Lifecycle hooks

To be completed.

### Using properties (props) to send data in and out of a component

The "properties" of a component are its parameters, you can use them to pass values into the component, and pass events out.

In the parent class, pass data to a child component like this:

```html
<Component someValue="foo" />
<Component someValue={this.foo} />
```

And in the child component you can access the value like this:

```javascript
this.props.someValue
```

To raise an event, in the parent class, link the prop to a method like this:
    
```javascript
onDelete() {
...
}

<Component callback={this.onDelete} />
```

Then in the child class you can call the method like this:

```javascript
this.props.callback();
```

### Validating props using PropTypes (optional)

React provides an optional method of validating strongly typing props.

Add the prop-types package to the project:

```text
npm i prop-types --save
```

In a component which has props, import PropTypes:

```javascript
import PropTypes from 'prop-types';
```

Add propTypes as a static class member.   It defines the type of each prop used by the class.

```javascript
class MyComponent extends React.Component {
	static propTypes = {
		Request: PropTypes.shape({
			english: PropTypes.string,
			spanish: PropTypes.string
		}),
		Responses: PropTypes.array,
		onMultiCompleted: PropTypes.func
	};
}
```

For examples of validators for different types, check here: https://reactjs.org/docs/typechecking-with-proptypes.html

### Using state to store data in a component

The component has a "state" object which contains whatever data we want to put into it.

In the class we can set an initial state for the component.  

```javascript
class MyComponent extends React.Component {
    state = {
            foo: 'foo'
    };
}
```

You can also use a class constructor, if you wish to use a value from props to set the initial state.

```javascript
constructor(props)
{
	super(props);
	this.state = {
		query: props.query
	};
}
```

In subsequent functions, we can change the state.  We should do it like this:

```javascript
this.setState({foo: "bar"});
```

Calling this.setState will rerender the component for us after changing the state.  Code should not try to set the state variable directly apart from the first initialisation.

this.setState is asynchronous.  Any code you place after the setState function will run before the state is changed.  But you can use a call back:

```javascript
this.setState({status: value}, () => { this.search(1, this.state.pagesize) });
```

You can read from the state like this.  

```javascript
this.state.foo
```


### Writing a render function in a component

The render function is responsible for returning the content that we want the component to display.  

Render is called when:

* This components props are changed by the parent component
* This components state is changed by calling setState

Note that a rerender of a parent *does not* re-render a child unless the child props were changed.

A pattern for writing render functions is to break them down a bit.  For example:

```javascript
render = () => {
	return (
		<div>
		{this.renderThingOne()}
		{this.renderThingTwo()}
		</div>
	)
}

renderThingOne = () => {
	return (
		<p>Thing One</p>
	)
}

renderThingTwo = () => {
	return (
		<p>Thing Two</p>
	)		
}
```

This is kinda a half way step before breaking the item out into an actual separate component.

### Functional components

A functional component has no class or state.  It accepts props and returns a react element.  This is good to use as a simple efficient component when no state is necessary.

```javascript
const HelloComponent = (props) => {
    return (
        <p>Hello {props.username}</p>
    );
};
export default HelloComponent;
```

A common code pattern for UIs where subcomponents display something related to each other is to "lift state up", putting the state for everything in a container component, then passing state values down to different functional components using props.


### Context

Context provides a way to pass props to descendent components without explicitly passing them through each child level.  E.g. in the following structure, Component A could pass some data to Component C without us making any change to Component B (which may be a third party component that is out of our control)

```text
Component A -> Component B -> Component C
```

Context is implemented by creating a Provider that specifies which values to pass, and a Consumer that receives them.  Refer to here for implementation details: https://reactjs.org/docs/context.html


### URI Routing

Use this command to install react-router.

```text
npm install --save react-router-dom 
```

The react-router-dom library provides common react-router components (which are shared with react-native) plus components specific for web sites (such as BrowserRouter).

Import the requred classes:

```javascript
import {BrowserRouter as Router, Route} from 'react-router-dom';
```

In the render function use the router.  This route says the / URI should render the HomePage component.

```html
<Router>
	<React.Fragment>
		<Route exact path={config.site_root + "/"} component={HomePage} />
	</React.Fragment>
</Router>
```

Note: the router needs a tag child which isn't a route.  You can put HTML or components there.  I'm using React.Fragment, which is a component that outputs no content.

We can match parameters in the URI and use them.

```html
<Route exact path={config.site_root + "/service/:service/jobs/:status"} render={(props) => ( <ServicePage Service={props.match.params.service} CurrentTab={props.match.params.status} /> )} />
```

Reference: https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/guides/quick-start.md



### Redirecting

There's a redirect component we can use if we need to redirect.

Import it using:

```javascript
import { Redirect } from 'react-router-dom';
```

Then call it using:

```html
<Redirect to={"/some_generated_url"} />
```

There's a pattern we can use if we need a component to redirect in certain cases.  We would set a state variable, and have the render function conditionally render a redirect based on the state.  E.g.

```javascript
that.setState({redirectUrl: '/someurl'});
```

..and in the render function..

```javascript
if (this.state.redirectUrl !== null)
	return (
		<Redirect to={this.state.redirectUrl} />
	)
```


### Forms

Initiate the state with the items on the form.

```javascript
state = {
	Name: ''
};
```

Set the name on the form items to match the name of the variables in state.  Bind the value to the variable.  Bind the onChange handler to a function.

```html
<TextField floatingLabelText="Name" name="name" value={this.state.Name} onChange={this.handleChange} /><br />
```

Use this generic function to handle all changes.

```javascript
handleChange = (e) => {
	var change = {};
	change[e.target.name] = e.target.value;
	this.setState(change);
}
```

Ref: https://hjnilsson.com/2016/12/11/generic-form-handleChange-in-react/

#### Controlled vs Uncontrolled form components

A controlled component is one where the value of the form control is bound to a variable, using a "value" attribute, and the "onChange" handler needs to update the value in state on every change so that the form control is showing the users changes.  

An uncontrolled component is one where the value is not directly bound to a variable.  It may be set to an initial value using the "defaultValue" attribute, and we'd probably have a "submit" type handler which submits the final value when the user has done editing.  I use this when the render function is too slow for us to call it on every change.


### HTTP requests using fetch

fetch() is provided by ES6 support in browsers.  

Here is how to implement a GET request.   Fetch calls are asyncronous, they return a promise.  The then() code runs when the promise completes.

```javascript
fetch(url)
	.then(funcstion (response) {
		// check the response, get json if OK, otherwise throw error
		if (response.ok)
			return response.json();
		else 
			throw Error(`HTTP ${response.status}`);
	})
	.then(function (data) {
		// it was ok, put the json data in the state
		that.setState({services: data});
	})
	.catch(function (exception) {
		// handle error
		console.log("caught an exception");
	});
```

Here is a POST.  As you can see, we pass the posted data in as a 'body' parameter to fetch.

```javascript
fetch(url, {
	method: 'POST',
	headers: {'content-type': 'application/json'},
	body: JSON.stringify(this.state.executor)
})
.then(function(response) {
	if (response.status == 204) {
		// we have success
	} else 
		throw Error(`HTTP ${response.status}`);
})
.catch(function(exception) {
	// handle error
	console.log("caught an exception");
});
```

Note: adding credentials: 'include' to the options enables sending of any basic auth details that were used to authenticate against the web site.

I think the fetch code is quite ugly,.  It would be better if we didn't need the first then() where we have to get the json data from the response.  Also, it'd be better if any error went to the error handler instead of requiring us to check for errors and then throw an error.  So how do we fix that?  Use the axios package.

### HTTP requests using axios

Install axios:

```text
npm install axios --save 
```

Here is how to implement a GET request.

```javascript
axios.get(url)
	.then( (response) => {
		// do something with the successful response
		this.setState({ databases: response.data });
	})
	.catch( (error) => {
		// handle the error
		this.setState({ errorMessage: error.message });
	});
```

That's a lot better.




### Semantic UI

Include the CSS:

```html
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/semantic-ui/2.2.12/semantic.min.css"></link>
```

(Note: other ways to include the CSS are described here: https://react.semantic-ui.com/usage)

Add the react package:

```text
npm install semantic-ui-react --save
```

Then import and use components.  E.g. 

```javascript 
import { Dropdown, Form, Button } from 'semantic-ui-react';
```

...

```javascript 
<Form>
    <Form.Group inline>
        <Form.Field>
            <label>Paragraph style</label>
            <Dropdown
                selection
                options={BlockTypes}
                value={props.selectedBlockType}
                onChange={props.onSelectBlockType}
            />
        </Form.Field>
    </Form.Group>
</Form>
```

View the available components at: https://react.semantic-ui.com/

### Material UI

Material UI is a CSS framework using Google's Material styles.

Add the roboto font:
	
```html
<link href='https://fonts.googleapis.com/css?family=Roboto:400,300,500' rel='stylesheet' type='text/css'>
```

```css    
font-family: 'Roboto', sans-serif;
```

Install module:

```text
mpm install --save material-ui
```

Include theme provider in the root of the project:

```javascript
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';

ReactDOM.render((
	<MuiThemeProvider>
		<Router>
	...
```

Then add matierial UI components.  Refer to demo components: http://www.material-ui.com/#/components/app-bar

Icons are listed here: https://material.io/icons/

An example of how to import and use them:

```javascript
import BalanceIcon from 'material-ui/svg-icons/action/account-balance';
import CheckBox from 'material-ui/svg-icons/toggle/check-box';
...
<BalanceIcon />
```



### Using inline styles

Inline styles are frequently employed in react code.  I think the main reason for this is to keep style definitions inside the components they are used by, rather than in an external CSS.

The pattern is usually:

```javascript
const styles = {
	paginationContainer: {
		display: 'flex',
		flexDirection: 'row',
		justifyContent: 'flex-end',
		padding: '.5em',
		fontSize: '.75em'
	}
}
```

Then pass this into a style attribute.

```html
<div style={styles.paginationContainer}>
```


### Configuration settings

We can pass configuration settings for the application into react's "start" and "build".

In package.json, define configuration settings as follows.  They must be prefixed with REACT_APP_.

```text
"start": "REACT_APP_API_URI=http://localhost:8066/ react-scripts start",
"build": "REACT_APP_API_URI=/ react-scripts build",
```

In the code, read the settings as follows:

```javascript
fetch(process.env.REACT_APP_API_URI + "xq/api-documents.xq?query=" + query, {
	credentials: 'include'
})	
```

These can also be set on a per-environment basis, e.g. in package.json:

```text
"build:test": "REACT_APP_API_URI=foo react-scripts build",
"build:staging": "REACT_APP_API_URI=bar react-scripts build",
```

Then produce a separate build for each environment as follows:

```text
npm run build:staging
```

### Query string 

Here's some tools that are useful for working with query strings.

Import withRouter:

```javascript
import {withRouter} from 'react-router-dom';
```

Then addd it as a Higher Order Component by adding it to the export line in your component.

```javascript
export default withRouter(DocumentListPage);
```

This will give your class access to new props, this.props.search and this.props.history, which are connected to the browser's location and history.

Also add the package qs (a "query string" parser):

```text
npm install --save qs
```

```javascript
import queryString from "qs";
```

Then we can parse the query string and retrieve parameters from it:

```javascript
const currentQueryString = queryString.parse(props.location.search.slice(1));
console.log(currentQueryString.foo);
```

To make a link which includes a query string:

* I actually don't know how to successfully do this :-)

To programmatically push parameters to the query string:

```javascript
const currentQueryString = new URLSearchParams(this.props.history.location.search);

// change some uri parameters
currentQueryString.set('foo', 'bar');
currentQueryString.delete('baz');

this.props.history.replace({...this.props.history.location, search: currentQueryString.toString()});
```

### Code splitting

By default create-react-app creates a webpack configuration that minifies all of the application code into a single JavaScript file. (nicknamed a chunk)

Very large javascript files haven't been terribly common in the past so some environments can have trouble with them.  if we need we can use code splitting to create multiple smaller chunks.  The separate chunks need to be imported during run-time.  Unfortunately our code needs to tolerate chunks being unavailable while they are loading.

The current main way of code splitting is to use the "import" statement, which imports javascript code and returns a promise.  

A useful library called react-loadable (https://github.com/jamiebuilds/react-loadable) makes this a bit easier, creating loadable components from import statements.  React-loadable returns a "loading" component while the component is unloaded, and the required component when it is finally available.  This lets us render something temporarily while a component is still loading.

The code to use react-loadable looks like this:

```javascript
const Toolbar = Loadable({
    loader: () =>
        import (/* webpackChunkName: "ChunkA" */ '../Components/Toolbar').then(e => e.default ),
    loading: () => {
        return (
            <p>Loading......</p>
        )
    }
});
```

This code returns a Toolbar component that is loaded from the Toolbar.js file (which is a normal React.Component).  The component will be loaded at runtime by the import() statement.  While the component is unloaded, the "loading..." paragraph is returned in place of the Toolbar component (so will render, if the <Toolbar /> component is used in the render function).  Then when loading completes, the real component is returned.  

Child components used by Toolbar will also be loaded from the chunk.

The webpackChunkName is a magic hint to webpack asking it to use a particular filename for this javascript chunk file.  You can provide the same filename to each component import, thereby grouping related components in a single chunk.  If a component ends up being used in two different chunks, it will be duplicated in both (which seems to work the same but is inefficient as we download duplicated code).  

### Higher order components

A higher order component is called by a child component and wraps around the child component, injecting additional props.

This is a trick to avoid duplicating code between components, if we need several components to use a value or function then we get them all to wrap themselves in the same higher order component which provides the required feature as a prop.

Refer to here for implementation details: https://reactjs.org/docs/higher-order-components.html




