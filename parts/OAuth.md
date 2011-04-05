Some calls from our Feest.Je API require authentication. This basically means that an user needs to be logged in with your app to the Feest.Je server.
Right now this might sound a tad weird, but that's okay. We will explain everything in this document in easy to understand steps.


OAuth Authentication
--------------------
The Feest.Je platform uses the OAuth protocol for authentication. If you want more information about OAuth, you should visit their [website](http://oauth.net).
We use this protocol to keep you from having a hard time creating login-scripts and accessing our user database for it.

Getting started
---------------
Alright, let's get to it. To use all the features that the Feest.Je API has to offer, you first need to follow a few simple steps.

The first step we need to go trough is to authenticate the user. This however, is only necessary if the user is not already logged in. How do you check this?
Well, you do not. Whether the user is logged in or not, it does not really matter. You just redirect the user to the following URL and we take care of everything
there.

	http://api.feest.je/oauthspul/?parameters
	
We will check here if the user is already logged in by using cookies that are stored on the users computer. Now, if the user was not logged in yet, we will show 
the user the following screen:

	inlogschermhier
	
At this screen, the user has to fill in its user credentials so we can verify if the user is legit. This all happens when the user presses the 'Allow' button.
As the name of the button suggests, this immediately gives your app the permission to use all the information of the user that gave the permission.

Now, if the user was already logged in, it does not need to log in again. But we can also not just give the permissions to your app. That is why we offer the user
the following screen. Here, the user can choose to give your app permission to use the user information or not.

	permissionschermhier
	
If the user desided to not give your app permission, we will redirect the user to the URL that you filled in at the **redirect_uri** parameter with additional information.

	redirectURL
	
If the user desided to give you permission however, we will also redirect the user to the URL that your filled in at the **redirect_uri** parameter with an authorization code.

	authorizationURL
	

Now that the user is authenticated and your app is authorized, the only thing left to do is authenticating your app. To do this you need to send the authorization code
and your app's secret to the Feest.Je API endpoint. You do this at (endpoint_url){theurl}. Below is an example of the call you will need to do to authenticate your app.

	authenticateappexample
	
	
If you send the right authentication code from your app and the right authorization code from the user, we will return an acces token for you.

    accescodeplaatje

Besides this acces token, the response contains the number of seconds that it will take for the token to expire. This number is stuffed in the expires parameter. Once this value reaches zero, you will have to go through the whole step of app authentication again. Remember however, that once the user authorizes your app, the user will not have to do this again.

If the authenication of your app somehow failed, we will return the following error:

    authenticationerrorplaatje




	
	
