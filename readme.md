Auth0 Example
=============

In this example project we'll user Auth0 to allow user to log in to our site via a Google account. This requires some setup with an Auth0 account, a Google developer account, and our page itself.

High level overview
-------------------
Below is an example of how we'll be using Auth0 to enable login functionality on our site. It acts as an external server - similar to the API calls we've done in the past. Auth0 will connect our app to third party services so that our users can log in with their other accoutns. In this example we'll be using the user's Google account.

![conceptual stack](images/conceptualStackFull.png)

We'll add a button on the HTML that will run a function on the client. This function will send a login request to Auth0. Then, Auth0 will request authorization from Google. Google will respond as to whether or not the authentication was good to Auth0. Then Auth0 will tell our web app what's up. If the authentication was successful, we'll also receive an object with the user's info.

Account Setup
=============
Before we can start working with Auth0 and Google in our app we'll need to set them up. First we'll set up our Google account, then our Auth0 account, then we'll let them know how to talk with each other. At that point we can start using it in our app.

Needed Accounts
--------------
First, create an account with Auth0 an a google developer account:

* https://auth0.com/
* https://console.developers.google.com/


Initial Google setup
--------------------

![conceptual stack](images/conceptualStackGoogle.png)

First, we'll need to create a new project:

![step 0](images/00-google 0.png)

Once our new project is created, we'll want to create some credentials for this project and set it as an OAuth project.

![step 1](images/01-google 1.png)

Now that we've got a new OAuth project we'll need to configure the consent screen. This is the screen that tells the users what our app will be accessing of their profile. We'll be accessing only enough to prove they are real.

![step 2](images/02-google 2.png)

On the next screen, let's give our product a name and ill in any other info if you have it ready.

![step 3](images/03-google 3.png)

On the next screen we'll choose that this is a Web Application and give the client for our application a name.

![step 4](images/04-google 4.png)

Save your work and keep this page open in a tab. We'll be coming back to it later.

Auth0 setup
-----------
![conceptual stack](images/conceptualStackAuth0.png)

First, we'll want to create a "client" on Auth0. This will be what talks to Google to get out authentication working.

![step 5](images/05-auth0 0.png)

Let's give our client a name and choose "Single Page Web Application".

![step 6](images/06-auth0 1.png)

We'll be using Angular, of course.

![step 7](images/07-auth0 2.png)

Once our app is created, take note of the domain as listed here. Copy it.

Also, in the "Allowed Callback URLs" text area, input the domains from which this app will be receiving calls. For this example we'll be using 'http://localhost:3030'.
Note: if you deploy to something like Heroku, you'll need to add the URLs here.

![step 8](images/08-auth0 3.png)

Keep the Auth0 tab open as well. We'll bounce back/forth between both and our app until we've got all working.

Connecting Auth0 and Google
---------------------------
![conceptual stack](images/conceptualStackAuth0Google.png)

In your Google Tab, we'll need to add the domain from our Auth0 tab in the "Authorized Redirect URIs" field. Not that you may need to add "https://" a the front of the domain.

This tells Google that it is acceptable for this Auth0 domain to make authentication requests.

![step 12](images/12-google 6a.png)

Back in the Auth0 tab, Expand the "Connections" menu item and choose "social". From here you'll be able to choose two social authentication endpoints (in the free version). We'll be using Google so make sure that is turned on.

![step 10](images/10-auth0 4.png)

Next we need to use the "Client ID" and "Client Secret" from our Google project and tell Auth0 about it. Copy these from the Google tab and paste them into the appropriate fields in the Auth0 tab.

![step 11](images/11-auth0 5.png)

These connections should now be set up and we've got to add Auth0's functionality to our app.


Setting up our project
======================
![conceptual stack](images/conceptualStackClient.png)

Start with a basic Node/Express/Angular project. The example project simply spins up a server on port 3030 and serves an index.html file.

We'll source in some setup from Auth0 followed by angular and our client side script:

```javascript
<!-- auth0 setup -->
<script src="https://cdn.auth0.com/js/lock-8.1.min.js"></script>
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<script src="vendors/angular.min.js" charset="utf-8"></script>
<script src="scripts/authExample.js" charset="utf-8"></script>
```

Yes, I know... normally I'd avoid using the CDN for serving a vendor file, but in this case the user will need to be connected to the web to authenticate anyway and we'll need Auth0's servers up to check to it should be fine. The viewport is there for the overlay that we'll see when things are working.

Next, we'll add a button to the page and hook it up to an ng-click that will run the login functionality.

```html
<body ng-app='myApp'>
  <h1>Node Express Auth0 Template 9-2016</h1>
  <div ng-controller='authController'>
    Please <button ng-click='logIn()'>Log In</button>
  <div>
</body>
```

We'll have to put some setup at the top of our js file.

* Replace CLIENTID with the Client ID from your Auth0 tab in the web browser.
* Replace DOMAIN with the domain from the same page (should look like YOURNAME.auth0.com)
* For the logOutUrl use the logout URL from your Auto0 client page.

```javascript
var lock = new Auth0Lock( 'CLIENTID', 'DOMAIN');
// log out url, from Auth0
var logOutUrl = 'https://YOURAUTH0SUBDOMAIN.auth0.com/v2/logout';
```

That button on the HTML will, of course, also require an angular module and controller. These are kept as simple as possible to start:

```javascript
var myApp=angular.module( 'myApp', [] );

myApp.controller( 'authController', [ '$scope', '$http', function( $scope, $scope ){
  console.log( 'authController here!');
  $scope.logIn = function(){
    // call out logIn function from auth0.js
    console.log( 'in logIn' );
    lock.show( function( err, profile, token ) {
      if (err) {
        console.error( "auth error: ", err);
      } // end error
      else {
        // save token to localStorage
        localStorage.setItem( 'userToken', token );
        console.log( 'token:', token );
        // save user profile to localStorage
        localStorage.setItem( 'userProfile', JSON.stringify( profile ) );
        console.log( 'profile:', profile );
      } // end no error
    }); //end lock.show
  }; // end scope.logIn
}]); // end authController
```

This runs the "$scope.logIn" function on button click. For our first tests, this is all we should need in our js file as well as our HTML file.

Time to test!
-------------
* take a deep breath
* cross your fingers
* spin up your server
* open your page in a new tab
* click the "Log In" button

If things are set up correctly you should see the Auth0 popup login box

![step 13](images/13-signUp.png)

Choose "Sign Up" and click the Google logo. You should be taken to a page that shows the name of your app and the access to the user's google account that was set in your Google setup phase earlier.

![step 14](images/14-permissions.png)

Once you accept you should be logged in. In this example we're really just logging out the token for this user's logged in session as well as the user object.

![step 15](images/15-loggedIn.png)

Next Steps
----------
If you've made it this far you've got the ability to do some basic log in using Auto0. In pairs, take a look at the rest of the code in a peer programming environment. Reconstruct the code and deconstruct its meaning. I'll be patrolling the space and answering questions.

If you can recreate this project you've got a starting point from which to work with basic authentication.

https://www.youtube.com/watch?v=Q7_jbluF0qo
