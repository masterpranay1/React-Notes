# 7. Master Project : FireBase

Created: January 29, 2022 10:07 PM

# FireBase + User Authentication

Reading **Documentation** is very important.

**Firebase** is created by **Google.** And it provides features like **Google Login** ( Authorization ), **storing** ( database ).

It helps us to use backend features without coding much in backend. 

**Note:-**  Using Latest version of **Firebase** is recommended. 

**Add** a new project on firebase and copy the config object which have the **`api key`** and some other important thing.

Also In the authentication section turn on the **Google SignIn** .

Now, we created a new folder `firebase` inside the **src** folder and create a **firebase.utils.js** inside the **firebase** folder.

First we will import the firebase and then `firestore` and `auth` .

```jsx
import firebase from "firebase/compat/app";
import "firebase/compat/firestore";
import "firebase/compat/auth";
```

Then add our **config object**.

```jsx
const config = {
    apiKey: "AIzaSyBp1xPb9SMwn4ZCGjt3uqxxS2fRW8zuKR8",
    authDomain: "crwn-db-2fe10.firebaseapp.com",
    projectId: "crwn-db-2fe10",
    storageBucket: "crwn-db-2fe10.appspot.com",
    messagingSenderId: "967933534587",
    appId: "1:967933534587:web:51db92d11041017a369dde"
};
```

**Note** :- We can share this key ( However there will be a warning from **git guardian** ).

Now we will initialise the firebase and export our **auth** and **firestore.** 

```jsx
firebase.initializeApp(config);
export const auth = firebase.auth();
export const firestore = firebase.firestore();
```

## Google SignIn

```jsx
const provider = new firebase.auth.GoogleAuthProvider();
provider.setCustomParameters({ prompt: 'select_account' });
export const signInWithGoogle = () => auth.signInWithPopup(provider);
```

First we added the **Google auth Provider** and then in the custom parameters passed the **prompt.**

In Firebase, Under the Authentication tab we can the user email and user id of those who logged in our application.

![Screenshot from 2022-01-30 11-52-44.png](./images/Screenshot_from_2022-01-30_11-52-44.png)

Now we integrate the data we got from here to our app.

In our **App.js**, convert the App to a class component because we now need to store a state that is our **current user**

```jsx
this.state = {
  currentUser : null
}
```

Now add **componentDidMount and componentWillUnmount** Life cycle methods and one variable, **unsubscribeFromAuth** and set it null initially.

```jsx
componentDidMount() {
  this.unsubscribeFromAuth = auth.onAuthStateChanged(user => {
    this.setState({ currentUser : user});
    console.log(user);
  })
}
```

The `auth` is the function we exported from firebase utils. We called the **onAuthStateChanged** method which is a kind of **event** that is called whenever state changed ( It will change on App first load ). And we log the **user** parameter we get from firebase in this method.

This auth returns a function which will **unsubscribe the auth**. ~~(~~ However this is still not very clear, also instructor told that this is a very conceptual concept so no need to worry 🥲 )

And this returned function is called using **componentWillUnmount()** life cycle.

```jsx
componentWillUnmount() {
  this.unsubscribeFromAuth();
}
```

## Adding a Sign Out And Sign In

We will one link and one Special Option to our **Header Component.**

```jsx
{
    currentUser ?
    <div className="option" onClick={() => auth.signOut()}>SIGN OUT</div>
    :
    <Link className="option" to='/signin'>SIGN IN</Link>
}
```

`auth` is imported and it returns a special `signOut` method.

Apart from this we added some styling to our Google Button ( to look like a Google Button ) and added a `isGoogleSignin` bool kind parameter to **custom button** component.

## FireStore

We can set up a firestore database in two way, one in **production mode** and one is **test mode.** In test mode anybody has access to the data and it is valid only for **30 days.** 

And the **production mode** requires authentication.

[ Currently we are going with the **test mode** ]

FireStore store the data in **Collection-Document** Format.

For example, We created a new collection, User ( path is the root, `/` ). This `User` is the collection Id and the user can have multiple document with different **document id .** 

Now every individual document can have **multiple fields** or a **collection** and this cycle can continue.

**Note→** Multiple Fields means different **key:value** pair and one key can have more one value or If we want we have to create a new collection of that document ( **not field** ).

We can access these using, `firestore.collection('users')` and `firestore.collection('users').doc('docId')` .

We can even concat many collection or users like, `firestore.collection().doc().collection().doc()` and so on. An even alternative way is providing direct path , `firestore.doc('/users/docId/cartItems/docId2')`

Firestore either return **queryReference** or **querySnapshot** objects and either of these object can be **Document** or a **Collection**.

A **Document Reference** or a **Collection Reference** are two types of **References**  possible.

We use **documentRef** objects to perform our **CRUD** methods (create,retrieve, update, delete). The documentRef methods are `.set()`, `.get()`, `.update()` and `.delete()` respectively.

![Screenshot from 2022-01-31 22-12-11.png](./images/Screenshot_from_2022-01-31_22-12-11.png)

![Screenshot from 2022-01-31 22-13-34.png](./images/Screenshot_from_2022-01-31_22-13-34.png)

![Screenshot from 2022-01-31 22-13-45.png](./images/Screenshot_from_2022-01-31_22-13-45.png)

Now we create a function to Create / check where the User Data exist on **firestore** or not.

In firebase.utils.js ,

```jsx
export const createUserProfileDocument = async (userAuth, additionalData) => {
    if(!userAuth) return;
    const userRef = firestore.doc(`User/${userAuth.uid}`);
    const snapShot = await userRef.get();
    console.log(snapShot);
    if(!snapShot.exists) {
        const { displayName, email } = userAuth;
        const createdAt = new Date();
        try {
            await userRef.set({
                displayName,
                email,
                createdAt,
                ...additionalData
            })
        } catch (error) {
            console.log('error creating user', error.message);
        }
    }
    return userRef;
}
```

Notice we first queried using **document reference** and took the **document snapshot** . Thereafter checked whether the **snapshot** exists or not. 

If not we created the User using `.set` method.

Notice → this function is asynchronous and we are returning the **userRef** 

And inside the App.js, we updated the **componentDidMount** life cycle method.

```jsx
componentDidMount() {
    this.unsubscribeFromAuth = auth.onAuthStateChanged(async (userAuth) => {
      if(userAuth) {
        const userRef = await createUserProfileDocument(userAuth);
        userRef.onSnapshot(snapShot => {
          this.setState({
            currentUser : {
              id: snapShot.id,
              ...snapShot.data()
            }
          })
        })
      } else {
        this.setState({currentUser: userAuth})
      }
    });
  }
```

Note here userRef or snapshot or userAuth , all have have same id ( userAuth has uid ).

we used a special method `onSnapshot` which get fired when we query for a snapShot from userRef. 

## Sign Up Component

We will create a sign up class component.

```jsx
this.state = {
  displayName: "",
  email: "",
  password: "",
  confirmPassword: "",
};
```

Two event handling methods, handle submit and handle change

```jsx
handleSubmit = async event => {
    event.preventDefault();
    const {displayName, email, password, confirmPassword } = this.state;
    if(password !== confirmPassword) {
        alert("passwords don't match");
        return;
    }
    try {
      const { user } = await auth.createUserWithEmailAndPassword(email, password);
      await createUserProfileDocument(user, { displayName });
      this.setState({
          displayName: "",
          email: "",
          password: "",
          confirmPassword: "",
      })
    } catch(error) {
      console.log(error);
    }
}
handleChange = event => {
    const { name, value } = event.target;
    this.setState({ [name]: value });
}
```

Here we are `createUserWithEmailAndPassword` firebase function which create a user taking the email and password. Next is We passed the **user** object to the `createUserProfileDocument` which is function created by us ( which basically create a new user document in firestore of id equal to the **uid of user** ).

Now, when form is submitted we call **this.handleSubmit** and when Input field are changed we call **this.handleChange**.

Also, we updated the **handleSubmit** method in **Signin Component** in the same way we did in the **Sign Up**

```jsx
handleSubmit = async (e) => {
  e.preventDefault();
  const { email, password } = this.state;
  try {
    await auth.signInWithEmailAndPassword(email, password);
    this.setState({email: "",
    password: ""})
  } catch(error) {
    console.log(error.message);
  }
  this.setState({ email: "", password: "" });
};
```

Now, let us look at this **createUserProfileDocument.**

```jsx
export const createUserProfileDocument = async (userAuth, additionalData) => {
    if(!userAuth) return;
    const userRef = firestore.doc(`User/${userAuth.uid}`);
    const snapShot = await userRef.get();
    if(!snapShot.exists) {
        const { displayName, email } = userAuth;
        const createdAt = new Date();
        try {
            await userRef.set({
                displayName,
                email,
                createdAt,
                ...additionalData
            })
        } catch (error) {
            console.log('error creating user', error.message);
        }
    }
    return userRef;
}
```

Now just call and add the **Sign Up** Component to our **Sign in SIgn up page**.

---

# This section ends here. 😀