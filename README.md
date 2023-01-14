# passport-wallet

Passport extension for web3 wallet. Use this if signing in via signwithwallet.com


## Install

This module is installed directly using `npm`:

```sh
$ npm install @incubiq/passport-wallet
```

## Usage

The passport-wallet authentication strategy authenticates users using a browser plugin wallet and a wallet address. The strategy requires a verify callback, which accepts the following credentials. Create a authenticate/passport_siww.js file and copy the code below.

```js

const passport = require('passport');
const SIWWStrategy = require("@incubiq/passport-wallet").Strategy;

    // register this SIWW strategy ; make use of a cSIWW const which carries your SIWW app data (ID, secret, callback...)
    passport.use(new SIWWStrategy({
        clientID: cSIWW.clientID,                                    // our app_id
        clientSecret: cSIWW.clientSecret,                            // our app_secret (only used if enableProof is true)
        callbackURL: cSIWW.callbackURL,                              // our callback URL
        enableProof: true,                                           // set to true to make use of the app_secret to secure calls (false = unsecured calls)
        authorizationURL: cSIWW.host+ "oauth/dialog/authorize",      // where we call SIWW for authorization (fixed URL)
        tokenURL: cSIWW.host+"oauth/token",                          // where we call SIWW for getting oAuth token (fixed URL)
        profileURL: cSIWW.host+"oauth/resources/profile"             // where we call SIWW for receiving end use's profile (fixed URL)
    },
    function(accessToken, refreshToken, profile, done) {
        try {
            if(!profile) {
                return done(null, false, {});       // got an error...
            }

            // call your register user here, and make use of the profile data received
            registerUser({
                // profile.connector        => the SIWW wallet connector (eg SIWC for "Sign-in with Cardano")
                // profile.blockchain       => the blockchain used by SIWW for authentication (eg "cardano")
                // profile.username         => the user's SIWW unique username
                // profile.wallet_id,       => the signing wallet's name (eg "nami")
                // profile.wallet_address,  => the signing wallet's address (eg "addr123....")
                // profile.authorizations,  => array of authorization levels received from SIWW 
            })
            .then(function(obj){
                    process.nextTick(function () {
                        return done(null, obj);
                    });
                })
                .catch(function(obj){
                    return done(null, false, "failed to register user");
                });
        }          
        catch (err) {
            return done(null, false, err);
        }
    }));

```

Then to authenticate requests, use passport.authenticate(), specifying the strategy defined in the above file passport_siww.js. 

Setup the route as follows: 

```js
    const passportSIWW = require("../authenticate/passport_siww");    

    router.get('/siww',passport.authenticate('SIWW', {session: false}));        // will call SIWW /oauth/dialog/authorize
    router.get('/siww/callback', passportSIWW.authenticate('SIWW', {
        failureRedirect: '/auth/unauthorized',
        session: false
    }), function (req, res) {
        // completeLogin(req, res);
    });
```

## Sample

See a full working sample here: https://github.com/incubiq/sign_in_with_wallet/tree/main/backend_sample

## License

[MIT](LICENSE)

[node-url]: https://nodejs.org/en/download/
