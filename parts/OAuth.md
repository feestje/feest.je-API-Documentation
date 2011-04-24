Some calls from our Feest.Je API require authentication. In this document we'll explain how you can have a user give you access to their account.

The feest.je way of OAuth
-------------------------
The feest.je platform uses the OAuth protocol for authentication. If you want more information about OAuth, you should visit their [website](http://oauth.net).
We use this protocol to keep you from having a hard time creating login-scripts and accessing our user database, but we won't explain the details about how oauth works, writing an app is enough work as it is!
There are, however, some things you need to know. These are a few of our OAuth requirements:
 * **Authenticate using headers** - We use header-based OAuth because it's easier, less error-prone and (above all) neater! No OAuth data cluttering your parameters!
 * **Always use SSL for /oauth/* endpoints** - When sending requests during the request_token, access_token and authorize steps **always** use SSL!
 * **Always pass an OAuth callback** - Not only do we require you to do so, but it's also a nifty way of passing back extra information to yourself.

Signing a request
-----------------
All OAuth 1.0A requests use the same basic algorithm for creating a signature base string and a signature.
The signature base string is probably the most brain-bending part of OAuth. The signature base string is composed of the HTTP method being used, followed by an ampersand ("&") and then the URL-encoded base URL being accessed, complete with path (but not query parameters), followed by an ampersand ("&"). Then, you take all query parameters and POST body parameters (when the POST body is of the URL-encoded type, otherwise the POST body is ignored), including the OAuth parameters necessary for negotiation with the request at hand, and sort them in lexicographical order by first parameter name and then parameter value (for duplicate parameters), all the while ensuring that both the key and the value for each parameter are URL encoded in isolation. Instead of using the equals ("=") sign to mark the key/value relationship, you use the URL-encoded form of "%3D". Each parameter is then joined by the URL-escaped ampersand sign, "%26".
This algorithm is simply expressed in the pseudo-code:

    httpMethod + "&" +
    url_encode(  base_uri ) + "&" +
    sorted_query_params.each  { | k, v |
        url_encode ( k ) + "%3D" +
        url_encode ( v )
    }.join("%26")

Here at feest.je we require you to sign all your OAuth requests using the HMAC-SHA1 algorithm.
Did you get all of that? Then we've got some great news for you: no matter what kind of OAuth 1.0 request you are making, the rules for generating the signature base string remain constant, so you can use this knowledge on all sites that use OAuth!

Step 1: My kingdom for a token
------------------------------
You now know enough about OAuth to start with the first step of authentication! Good times!
The first step is to get a request token. This gives us a heads up on what you're planning to do and in return you get a temporary token that you can pass along with the url when you direct the user to the site.
The endpoint for the request token is **https://feest.je/oauth/request_token**. For security reasons you should always send this request as a **POST** request over SSL.
Lets pretend the consumer secret of our app is "6e05752ce1308eec2305e00f08b44a85381e579b", we'll be passing the following arguments with the call:

 * **oauth_consumer_key**
   - 2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90
 * **oauth_nonce**
   - 0004cbb55076f17524ec4e65d8701e67e4
 * **oauth_signature_method**
   - HMAC-SHA1
 * **oauth_timestamp**
   - 1302526985
 * **oauth_version**
   - 1.0

After we sort the parameters and our request generates the following signature base string:

    POST&http%3A%2F%2Fwww.feestje.local%2Foauth%2Frequest_token&oauth_consumer_key%3D2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90%26oauth_nonce%3D0004cbb55076f17524ec4e65d8701e67e4%26oauth_signature_method%3DHMAC-SHA1%26oauth_timestamp%3D1302526985%26oauth_token%3D%26oauth_version%3D1.0

Note that we're not using an oauth_token_secret when calculating the composite signing key, because we don't have one yet. You do have to include the ampersand, though (I know, it looks weird...)

    6e05752ce1308eec2305e00f08b44a85381e579b&

Next we use the composite signing key to create an oauth_signature from the signature base string by signing the base string using the composite signing key (take your time to read this line a couple of times). The resultant oauth_signature is:

    eXBdrfxcdMN9SEvrLJfZG8KCtX4=

Phew! That was a lot of work for a single request, but we're almost there. All we have to do now is put all the pieces together and make our Authorization header:

    Authorization: OAuth realm="http://www.feestje.local/oauth/request_token",
      oauth_consumer_key="2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90",
      oauth_token="",
      oauth_nonce="0004cbb55076f17524ec4e65d8701e67e4",
      oauth_timestamp="1302526985",
      oauth_signature_method="HMAC-SHA1",
      oauth_version="1.0",
      oauth_signature="eXBdrfxcdMN9SEvrLJfZG8KCtX4%3D"

Now you're finally ready to send the request and get your temporary oauth_token and oauth_token_secret. If everything went well you should get something back like this:

    oauth_token=73d5de2cb3e2563c3f0416433236d81f0c00303c&oauth_token_secret=1854955440fac040cbbf2678dee1e0c145c84ef1
    
Store these tokens, you'll need them in the next step and exchange them for the real deal in step three.

Step 2: Enter stage left; the user
----------------------------------
Finally, it's time for the user to do something! In this step we'll send the user to the feest.je site for authorization, passing along our temporary user_token and callback url, so the server knows who we are and we know when the user is done.
This really is the easiest part of OAuth. All you have to do is redirect the user to **https://feest.je/oauth/authorize** and pass the oauth_token we received in the previous step, and our callback url:

    https://feest.je/oauth/authorize?oauth_token=73d5de2cb3e2563c3f0416433236d81f0c00303c&oauth_callback=http:%2F%2Flocalhost%2Fcallback

**Don't forget to url-encode your callback url!**

The user will be asked to confirm acces to his account for your app by either pressing the approve button, or logging in. If the user grants you access your callback url will be called and your oauth_token will be passed along, together with an oauth_verifier.
In the case of this example the following url will be called after the user grants the app access:

    http://localhost/callback?oauth_token=8ldIZyxQeVrFZXFOZH5tAwj6vzJYuLQpl0WUEYtWc&oauth_verifier=pDNg57prOHapMbhv25RNf75lVRd6JDsni1AJJIDYoTY

**It is highly recommended to pass a https link as a callback url.**

Step 3: The grand finale
------------------------
Only one thing remains: to claim what is rightfully yours! In the final step you'll exchange your temporary oauth_token, oauth_token_secret and oauth_verifier for the actual user_token.
To do this we'll make one more OAuth call to **https://feest.je/oauth/access_token** with the following parameters:

 * **oauth_consumer_key**
   - 2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90
 * **oauth_nonce**
   - 8ldIZyxQeVrFZXFOZH5tAwj6vzJYuLQpl0WUEYtWc
 * **oauth_signature_method**
   - HMAC-SHA1
 * **oauth_token**
   - 73d5de2cb3e2563c3f0416433236d81f0c00303c
 * **oauth_timestamp**
   - 1302546985
 * **oauth_verifier**
   - pDNg57prOHapMbhv25RNf75lVRd6JDsni1AJJIDYoTY
 * **oauth_version**
   - 1.0

Even though it will be the last time we'll use it, our oauth_token_secret is still **1854955440fac040cbbf2678dee1e0c145c84ef1**.

As before, we set up our POST call over SSL, starting with the base string:

    POST&https%3A%2F%2Ffeest.je%2Foauth%2Faccess_token&oauth_consumer_key%3D2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90%26oauth_nonce%3D8ldIZyxQeVrFZXFOZH5tAwj6vzJYuLQpl0WUEYtWc%26oauth_signature_method%3DHMAC-SHA1%26oauth_timestamp%3D1302546985%26oauth_token%3D73d5de2cb3e2563c3f0416433236d81f0c00303c%26oauth_version%3D1.0

Next, we use our oauth_consumer_secret and our oauth_token_secret to create a composite signing key:

    6e05752ce1308eec2305e00f08b44a85381e579b&1854955440fac040cbbf2678dee1e0c145c84ef1

We use this key to sign our request, and end up with the following OAuth signature:

    WNV0SIP5Z3slzNfnqv9quGpLp4Q=
    
All we have to do now is send the request with the right Authentication header:

    Authorization: OAuth realm="https://feest.je/oauth/access_token",
      oauth_consumer_key="2c176a8e3e4bbcfbaf69fcb388d8841c495f4d90",
      oauth_token="73d5de2cb3e2563c3f0416433236d81f0c00303c",
      oauth_nonce="8ldIZyxQeVrFZXFOZH5tAwj6vzJYuLQpl0WUEYtWc",
      oauth_timestamp="1302546985",
      oauth_signature_method="HMAC-SHA1",
      oauth_version="1.0",
      oauth_signature="WNV0SIP5Z3slzNfnqv9quGpLp4Q%3D"

Make the call and you'll get your oauth_token and oauth_token_secret in the response data.
**Congratulations, you can now make API calls on the user's behalf!**
