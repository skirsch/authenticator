# iPhone native app authenticator that can be used by multiple RPs (including Passli) to get additional assurance

# what the caller specifies
caller specifies the JSON for the message the app will display on the "logged in user's iPhone" (logged into the current browser). 

## example JSON request

From account: 213432343
To Bank: 2343243
To account number: 23432423
Amount: 324.23
Currency: USD
Button: Approve wire transfer

## User registration
User starts up the app
The app then is associated using the standard method because the browser was paired on the phone so it knows who the user is from the browser just like any other app.


user gets to see pre-canned button options like:
- approve 
- sign 
- authorize
- buy 

Field options in JSON include:
- AccountID
- From
- To
- Amount
- Currency
- ...

## callback
 rp's have pre-registered callbacks to prevent abuse.

## approval LoA options
- simple response (hit button)
- add faceid/PIN before responding (likely overkill)

## Signing key
crypto private key generated per RP on first use and stored as non-exportable.
It signs the JSON of: request, the LoA level, response selected, date/time, nonce (sent from the RP to the app)

ECDSA is the only algorithm supported by iPhones
