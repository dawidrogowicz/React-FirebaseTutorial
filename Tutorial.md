# React Firebase chat app

##Introduction

If you are into front-end development I bet you heard about REACT JS.  
This unique framework is becoming an industry standard. If you want to know more on how to use it stick to this tutorial and we will unveil the basics, create a production ready application and integrate it with *real-time database* provided by __FIREBASE__.  
We will together build a messaging app with users authentication by __GOOGLE AUTH__.
![What We Will Build](WhatWeWillBuild.png)

For starters, let's explain what actually is __REACT__ and __FIREBASE__, how it works and how can we utilize it.
![React Logo](ReactLogo.png)  

__REACT__ is a JavaScript Library for building interfaces created by __FACEBOOK__ developers. In short it creates a virtual DOM that is updated whenever some state of your application changes.
It is optimized for performence as it re-renders only elements of the DOM affected by the change.  
  
![Firebase Logo](FirebaseLogo.png)  

__FIREBASE__ is a *BAAS* (backend as a service) that provides a lot of useful tools for mobile and web development. In it's offer we can find Hosting (even for node apps), real-time database (what we are interested in), authentication using E-mail, __Facebook__ or __Google__.


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


### Structuring the application 
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

### Creating a Message
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
Also we need to add some nice styling

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
