# React Firebase chat app

## Introduction

If you are into front-end development I bet you heard about REACT JS.  
This unique framework is becoming an industry standard. If you want to know more on how to use it stick to this tutorial and we will unveil the basics, create a production ready application and integrate it with *real-time database* provided by __FIREBASE__.  
We will together build a messaging app with users authentication by __GOOGLE AUTH__.
![What We Will Build](WhatWeWillBuild.png)

For starters, let's explain what actually is __REACT__ and __FIREBASE__, how it works and how can we utilize it.
![React Logo](ReactLogo.png)  

__REACT__ is a JavaScript Library for building interfaces created by __FACEBOOK__ developers. In short it creates a virtual DOM that is updated whenever some state of your application changes.
It is optimized for performence as it re-renders only elements of the DOM affected by the change.  
  
![Firebase Logo](FirebaseLogo.png)  

__FIREBASE__ is a *BAAS* (backend as a service) that provides a lot of useful tools for mobile and web development. In it's offer we can find Hosting (even for node apps), real-time database (what we are interested in), authentication using E-mail, __Facebook__ or __Google__. You can find all information about __FIREBASE__ [here](https://firebase.google.com/docs/).


## Creating FIREBASE project.
For this little tutorial we will create new __FIREBASE__ project, you can do that [here](https://console.firebase.google.com/u/0/).
After that got to database Tab in your project and to the Rules. You need tochange the rules so we can actualy write something to the DB.
Your rules should look something like this:  
```json
 {
   "rules": {
     ".read": "auth == null",
     ".write": "auth == null"
   }
 }
 ```
We also need to set up an Authentication with __GOOGLE__.  
In Your __FIREBASE__ Console got to Authentication and enable Google auth provider.

## Creating React App
Ok, lets talk code.
What dependencies will we need?
We will use [*Create React App*](https://github.com/facebook/create-react-app/tree/master)
To install all dependencies run:  
```node
 npm install --global create-react-app
```

Then in directory where you want your project to live run:
```node
npx create-react-app react-intro
cd react-intro
```

Ok we need one more thing which is __FIREBASE__ npm module
```node
npm install --save firebase
```

Let's give it a try. Run the app:
```node
npm start
```

You should see standard *create-react-app* template.
![create-react-app template](createReactapp.png)


## Creating the application structure 
By now your folder structure should look like one below:
![FolderStructure](FolderStructure.png)  

That is a little mess, let's clean it up.
Our chat app will be created with use of 3 react components:  
Wrapper - App.js  
Form - Form.js  
Message - Message.js

We also need .css files for a little style and a separate file for __FIREBASE__ configuration. 

To understand what react component is please refer to reactJS [documentation](https://reactjs.org/)
Our Project should look like this to be more compact.
![FolderStructure Refractor](FolderStructureRefractor.png)

## Creating a Message
Our Message Will contain two properties: UserName and a Message itself.
##### Message.js
```javascript
import React, {Component} from 'react';
import './Message.css';

export default class Message extends Component {
  render() {
    return (
      <div className="message">
                <span className="message__author">
                    {this.props.message.userName}:
                </span>
        {this.props.message.message}
      </div>
    )
  }
}
```


This Message will be rendered for each record in the DataBase.

## Creating Form
Our form will handle the message rendering and adding a new message to database.
Lets see how it looks.

  
##### Form.js
```javascript 
import React, { Component } from 'react';
import './Form.css';
import Message from '../Message/Message';
import firebase from 'firebase';

export default class Form extends Component {
  constructor(props) {
    super(props);

    this.state = {
      userName: 'Sebastian',
      message: '',
      list: [],
    };

    this.messageRef = firebase.database().ref().child('messages');
    this.listenMessages();
  }

  componentWillReceiveProps(nextProps) {
    if(nextProps.user) {
      this.setState({'userName': nextProps.user.displayName});
    }
  }

  handleChange(event) {
    this.setState({message: event.target.value});
  }

  handleSend() {
    if (this.state.message) {
      var newItem = {
        userName: this.state.userName,
        message: this.state.message,
      }

      this.messageRef.push(newItem);
      this.setState({ message: '' });
    }
  }

  handleKeyPress(event) {
    if (event.key !== 'Enter') return;
    this.handleSend();
  }

  listenMessages() {
    this.messageRef
      .limitToLast(10)
      .on('value', message => {
        this.setState({
          list: Object.values(message.val()),
        });
      });
  }

  render() {
    return (
      <div className="form">
        <div className="form__message">
          { this.state.list.map((item, index) =>
            <Message key={index} message={item} />
          )}
        </div>
        <div className="form__row">
          <input
            className="form__input"
            type="text"
            placeholder="Type message"
            value={this.state.message}
            onChange={this.handleChange.bind(this)}
            onKeyPress={this.handleKeyPress.bind(this)}
          />
          <button
            className="form__button"
            onClick={this.handleSend.bind(this)}
          >
            send
          </button>
        </div>
      </div>
    );
  }
}
```

Ok that's quite some code.
Lets see whats going on here. 
This component is a perfect example of Class Component.
It has a state that we set up with initial values.
```javascript
constructor(props) {
    super(props);
    
    this.state = {
      userName: 'Sebastian',
      message: '',
      list: [],
    };
    
    this.messageRef = firebase.database().ref().child('messages');
    this.listenMessages();
}
```
There are also methods to work with events.
```javascript
  handleChange(event) {
    this.setState({message: event.target.value});
  }

  handleSend() {
    if (this.state.message) {
      var newItem = {
        userName: this.state.userName,
        message: this.state.message,
      }

      this.messageRef.push(newItem);
      this.setState({ message: '' });
    }
  }

  handleKeyPress(event) {
    if (event.key !== 'Enter') return;
    this.handleSend();
  }

  listenMessages() {
    this.messageRef
      .limitToLast(10)
      .on('value', message => {
        this.setState({
          list: Object.values(message.val()),
        });
      });
  }
```
And Component Life Cycle Methods
```javascript
  componentWillReceiveProps(nextProps) {
    if(nextProps.user) {
      this.setState({'userName': nextProps.user.displayName});
    }
  }
```
This last one handles events specific for React Components. 
Lets Head to the last but not least component.
## App Component 
 
This component will be a wrapper on our form and message.
It will also handle __FIREBASE__ authentication.
##### App.js
```javascript
import React, { Component } from 'react';
import logo from '../../images/logo.svg';
import './App.css';
import Form from '../Form/Form.js';
import firebase from 'firebase';
import firebaseConfig from '../../config';

firebase.initializeApp(firebaseConfig);

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      user: null,
    }
  }

  componentDidMount() {
    firebase.auth().onAuthStateChanged(user => {
      this.setState({ user });
    });
  }

  handleSignIn() {
    const provider = new firebase.auth.GoogleAuthProvider();
    firebase.auth().signInWithPopup(provider);
  }

  handleLogOut() {
    firebase.auth().signOut();
  }

  render() {
    return (
      <div className="app">
        <div className="app__header">
          <img src={logo} className="app__logo" alt="logo" />
          <h2>
            SIMPLE APP WITH REACT
          </h2>

          { !this.state.user ? (
            <button
              className="app__button"
              onClick={this.handleSignIn.bind(this)}
            >
              Sign in
            </button>
          ) : (
            <button
              className="app__button"
              onClick={this.handleLogOut.bind(this)}
            >
              Logout
            </button>
          )}
        </div>
        <div className="app__list">
          <Form user={this.state.user} />
        </div>
      </div>
    );
  }
}

export default App;
```

## Final Touch
We need to add some styling 

##### Message.css
```css
.message {
    position: relative;
    padding: 10px 60px 10px 20px;
    font-size: 12px;
    color: rgba(0, 0, 0, 0.6);
    background-color: rgba(0, 0, 0, .05);
    text-align: left;
    line-height: 1.7;
}

.message:nth-child(even) {
    background-color: rgba(0, 0, 0, .025);
}

.message__author {
    font-weight: 600;
    margin-right: 10px;
    color: rgba(0, 0, 0, 1);
}
```

##### Form.css
```css
.form {
    display: flex;
    flex-wrap: nowrap;
    flex-direction: column;
    width: 100%;
}

.form__row {
    display: flex;
    flex-wrap: nowrap;
    width: 100%;
    padding: 20px;
}

.form__input {
    width: 100%;
    padding: 15px 20px;
    border-radius: 5px 0 0 5px;
    border: 1px solid #ddd;
    box-sizing: border-box;
    outline: 0;
}

.form__input:focus {
    background-color: rgba(245, 243, 243, 0.5);
}

.form__button {
    padding: 0 30px;
    background-color: #000;
    color: #fff;
    border: 0;
    border-radius: 0 5px 5px 0;
    outline: 0;
    cursor: pointer;
}

.form__button:hover {
    background-color: rgba(0, 0, 0, .8);
}

.form__message {
    width: 100%;
    padding: 0 20px;
}
```
##### App.css
```css
.app {
  text-align: center;
}

.app__logo {
  animation: App-logo-spin infinite 20s linear;
  height: 80px;
}

.app__header {
  background-color: #222;
  padding: 20px;
  color: white;
}

.app__intro {
  font-size: large;
}

.app__list {
  width: 100%;
  max-width: 600px;
  padding: 20px;
  margin: 0 auto;
}

.app__button {
  border: 0;
  border-radius: 0;
  padding: 10px 20px;
  color: black;
  background-color: white;
  cursor: pointer;
  opacity: .8;
  outline: 0;
}

.app__button:hover {
  opacity: 1;
}

@keyframes App-logo-spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

We need also add __FIREBASE__ config someware.
Lets add it to the config.js

##### config.js
```javascript
export default {
  apiKey: "AIzaSyAaODb7bCoZPvV4gdVyG_sV_Lc1_GuVdwg",
  authDomain: "react-intro-37cd1.firebaseapp.com",
  databaseURL: "https://react-intro-37cd1.firebaseio.com",
  projectId: "react-intro-37cd1",
  storageBucket: "react-intro-37cd1.appspot.com",
  messagingSenderId: "181293593583"
};
```

## Running our App
Hopefully that will work, let's test that.
```bash
npm start
```