# iPhone native app authenticator that can be used by multiple RPs (including Passli) to get additional assurance

caller specifies JSON for the message we will display
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

# 

# rp's have pre-registered callbacks to prevent abuse.

approval LoA options
- simple response (hit button)
- add faceid/PIN before responding (likely overkill)

## Signing key
crypto private key generated per RP on first use and stored as non-exportable.
It signs the JSON of: request, the LoA level, response selected, date/time, nonce (sent from the RP to the app)

ECDSA is the only algorithm supported by iPhones
