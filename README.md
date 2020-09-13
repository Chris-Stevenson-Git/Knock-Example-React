# Guide to using Knock for React/Rails

This is Part 2 to the tutorial.

The other half to this tutorial can be found [here](https://github.com/Chris-Stevenson-Git/Knock-Example-Rails)

## Setup
Create your React app
```bash
create-react-app your_react_project
```

Navigate into your react project and we need to do a couple of npm installs.
```bash
npm install axios react-router-dom
```

We now have access to axios and the react-router-dom.

## Edit text time!
First thing we need to do is create a couple of components for our React app.

Create two files in a components folder inside the src folder. We will call these Login and MyProfile.
```bash
/src/components/Login.js
/src/components/MyProfile.js
```
Your file tree should now look like this.

![filetree](https://i.imgur.com/CpDsvi9.png)

## App.js

Now we can jump straight into your App.js file located
```bash
your_proj_name/src/App.js
```

Lots of garbage here, easier to start fresh so delete everything.

Now we should import what we are going to need. Add these lines at the top of your file.

```jsx
import React from 'react';
import axios from 'axios'
import {Route, Link, HashRouter as Router} from 'react-router-dom';

import Login from './components/Login'
import MyProfile from './components/MyProfile'
```

We should also declare the BASE_URL we are going to be using

Below your import lines set the base URL to your server. By default it will be..

```jsx
const BASE_URL = 'http://localhost:3000'
```
### The App.js class

Now we need to create the structure of our App.

```jsx
class App extends React.Component{

  //App state
  state = {
  }

  //function to run on component mounting
  componentDidMount(){
  }

  //function to set the state to the current logged in user
  setCurrentUser = () => {
  }

  //function to log the user out.
  handleLogout = () => {
  }

  render(){
    return (

    ); // return
  } // render
} //class App
```

This is the main outline and we'll go through each function below.

### State
We only want to store one thing here, and this is a reference to the current user. I'm going the lazy route and just storing the whole user as an object but obviously this is not the best way to do this as we are holding onto sensitive and unnecessary data like the email and password digest.

Your state just needs to hold the currentUser
```jsx
  state = {
    currentUser: undefined
  }
```

### Component Mount
This is a function that will load once when you load the website. We just want to check if the user is logged in when we visit so we'll pass in the setCurrentUser function.

```jsx
componentDidMount(){
    this.setCurrentUser();
  }
```

### Set Current User
This is a function to get the current user from your database if there is one.

We declare that there is a token which holds a json web token 'jwt' from your local storage. (We'll set this on the login page).

We then do an axios request to the back end and ask it if we're logged in already. We pass through this token as an auth header which will let our server validate us.

If our token is valid then we set the state to our current user. If not you'll see a warning in your console that you're unauthorized.

```jsx
  setCurrentUser = () => {
    let token = "Bearer " + localStorage.getItem("jwt");
    axios.get(`${BASE_URL}/users/current`, {
      headers: {
        'Authorization': token
      }
    })
    .then(res => {
      this.setState({currentUser: res.data})
    })
    .catch(err => console.warn(err))
  }
```

### Handle Logout
Function to log us out of the page.
Three things we need to do here.
1. Set our state of current user to undefined.
2. Remove the jwt token from our local storage
3. Set our axios default headers to undefined.

This will completely reset our logged in state.

```jsx
  handleLogout = () => {
    this.setState({currentUser: undefined})
    localStorage.removeItem("jwt");
    axios.defaults.headers.common['Authorization'] = undefined;
  }

```

### Render
This is where all our HTML goes.

Feel free to design your own HTML page here. Mine just has a simple nav bar which will show if you're logged in and give you appropriate links to click on as well as a couple of routes to Login and My Profile so we can load the correct components.

The only thing really of note here is that we do pass the ability to set the current user down to the Login component.

```html
  render(){
    return (
      <Router>
        <header>
          <nav>
            {/* Show one of two nav bars depending on if the user is logged in */}
              {
                this.state.currentUser !== undefined
                ?
                (
                  <ul>
                    <li>Welcome {this.state.currentUser.name} | </li>
                    <li><Link to='/my_profile'>My Profile</Link></li>
                    <li><Link onClick={this.handleLogout} to='/'>Logout</Link></li>
                  </ul>
                )
                :
                (
                  <ul>
                    <li><Link to='/login'>Login</Link></li>
                  </ul>
                )
              }
          </nav>
          <hr/>
        </header>
        <Route exact path='/my_profile' component={MyProfile}/>
        <Route
          exact path='/login'
          render={(props) => <Login setCurrentUser={this.setCurrentUser}{...props}/>}
          />
      </Router>
    );
  }
```

Now finally after the class we need to export. So add

```js
export default App;
```

It should look like this.

![export](https://i.imgur.com/8DgvSzd.png)

## Login.js Component

This is our component to handle logging in.

Set up the basic format as below.


```js
import React from 'react'
import axios from 'axios'

const BASE_URL = 'http://localhost:3000'

class Login extends React.Component{
  state = {
  }

  //handle typing in the form
  handleInput = (ev) => {
  } //handleInput

  //handle the submit of the login
  handleSubmit = (ev) => {
  }

  render(){
    return(
    ); // return
  }// render
} // class Login

export default Login

```

Now let's go through each function

### State
The state this time needs to hold the email and password we're going to submit.

```jsx
  state = {
    email: '',
    password: ''
  }
```

### Handle Input
This is a function to set the state to be whatever is being typed in the form.

I've set this function up to take an argument of an event. If we then make our form fields have a name of either 'password' or 'email' the switch case will be able to recognise which form is being typed in and set the correct state. This saves us from making a handleInputEmail and handleInputPassword function.

```
handleInput = (ev) => {
    switch(ev.target.name){
      case 'email':
        this.setState({email: ev.target.value})
        break;
      case 'password':
        this.setState({password: ev.target.value})
    }
  } //handleInput
```

### Handle Submit
We need to do a post request to our server and ask it to validate our details.

For this I'm going to show you the code and explain it below

```jsx
  handleSubmit = (ev) => {
    //create a request object we can pass through to knock
    const request = {'email': this.state.email, 'password': this.state.password}

    //do an axios post request where we can send through the user details to rails and login
    axios.post(`${BASE_URL}/user_token`, {auth: request})
    .then(result => {
      localStorage.setItem("jwt", result.data.jwt)
      axios.defaults.headers.common['Authorization'] = 'Bearer ' + result.data.jwt;
      this.props.setCurrentUser();
      this.props.history.push('/my_profile');
    })
    .catch(err => {
      console.warn(err)
    })
    ev.preventDefault();
  }
```

So for the basics. We create an object called request and set it to have two keys; email and password. Each key is set to its state.

We also take an argument as the event and prevent default behaviour of reloading the page.

Now our axios.post request goes to the url /user_token and sends through a parameter of 'auth' with our request object as its value.

This will send through our username and password to the back end.

If it's successful then we
1. Set our local storage to have a json web token validating our login.
2. Set axios default headers to have an authorization key.
3. Call the inherited function so that we can set the current user in App.js
4. Redirect the URL of the page to /my_profile so we can load the MyProfile component.


### Render
All we need to render is a simple form taking an email and password.

```
render(){
    return(
      <form onSubmit={this.handleSubmit}>
        <label>Login Form</label>
        <br/>
        <input
          onChange={this.handleInput}
          name="email"
          type="email"
          placeholder='Enter Email'
        />
        <br/>
        <input
          onChange={this.handleInput}
          name="password"
          type="password"
          placeholder='Enter Password'
        />
        <br/>
        <button>Login</button>
      </form>

    ); // return
  }// render
```

 ## My Profile Component

I won't go through this one in detail as everything inside has already been explained. This is basically a component to show some details about the user.

```
import React from 'react'
import axios from 'axios'

const BASE_URL = 'http://localhost:3000'

class MyProfile extends React.Component{
  state = {
    currentUser: {
      name: '',
      email: ''
    }
  }

  componentDidMount(){
    let token = "Bearer " + localStorage.getItem("jwt");
    axios.get(`${BASE_URL}/users/current`, {
      headers: {
        'Authorization': token
      }
    })
    .then(res => {
      this.setState({currentUser: res.data})
    })
    .catch(err => console.warn(err))
  }

  render(){
    return(
      <div>
        <h1>Hello {this.state.currentUser.name}</h1>
        <h4>Your email is {this.state.currentUser.email}</h4>
      </div>
    );
  }//render

}//class MyProfile


export default MyProfile

```

And that's it! You can test it out by logging in, closing the tab, and reopening it to see if your server remembers you!
