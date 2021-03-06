API
===

Implementation details for the gatecoin Http API

---

##API Accessibility

__*Public*__: Everyone can execute the API functions through http without authentication

__*Private*__: Only http request with required headers can execute the API function


##Required Headers

`API_USER_ID`: The unique id for the user, can be obtained after login

`API_REQUEST_SIGNATURE`: An valid API Signature signed with API key in HMAC

`API_REQUEST_DATE`: The datetime of request in UTC format


##API Key

__*Long term API Key*__: An API Key generated on demand by the user *(using the /GetAPIKey function)*, valid until expiry date or deleted by user. Will not be removed upon logout.

__*Short term API Key*__: An API Key generated at successful login by the user to the system through the API.

##System Login

The user can login to the system through the API in the path /Login by passing username and password in a POST request. For example: `http://api.gatecoin.com/v1/Login` with json data 
```json
{ UserName : "gtcuser", Password : "oassword" }
```
The json string below will be returned on successful login:
```json
{
  "isSuccess": true,
  "apiKey": "A19DC6BE417615645646216D8589D9818",
  "publicKey": "6967615654684689A7E8FD41E5AB3C6DD",
  "alias": "alias1",
  "defaultCurrency": "BTCUSD",
  "defaultLanguage": "en-us",
  "verifLevel": 3,
  "userHasUnreadTickets": false,
  "lastLogonTime": "1420791056",
  "isPendingUnlockSecret": false,
  "responseStatus": {
    "message": "OK"
  }
}
```

*isSuccess*: *true* if success, otherwise *false*

*apiKey*: The API Key will be expired after 15 mins. The expiry date of API Key will be refreshed after each successful private API function call for up to 24 hour after creation.

For example, for an API Key created on `2013-11-01 07:00:00`, it will be expired on `2013-11-01 07:15:00`. If there is a private API function call at `2013-11-01 07:02:00`, the expiry date will be extended to `2013-11-01 07:17:00`. But the API Key will expire on `2013-11-02 07:00:00` no matter how many times the API Key was refreshed.

##Requested Signature

A valid request signature must be signed using HMAC with an active API Key and converted to base64 string.

The javascripts below contains required library for HMAC and base64 string conversion:

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/rollups/hmac-sha256.js

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/components/enc-base64-min.js

The code below demonstrate how to send required header using ajax:

```javascript
$.ajaxSetup({
	beforeSend: function (jqXHR, settings) {
		var publicKey = $("#input_public_key").val();
		var key = $("#input_key").val();
		if (publicKey == "") {
			publicKey = gPublicKey; // gPublicKey is a variable stored the publicKey when login
		}
		if (key == "") {
			key = gApiKey; // gApiKey is a variable stored the apiKey when login
		}
		if (publicKey != null && key != null) {
			var now = (new Date(Date.now())).getTime() / 1000;
			var httpMethod = settings.type;
			var ct = settings.contentType;
			if (ct == false) {
				ct = "multipart/form-data";
			}
			var contentType = (httpMethod == "GET") ? "" : ct;
			var message = settings.type + settings.url + contentType + now;
			var hash = CryptoJS.HmacSHA256(message.toLowerCase(), key);
			var hashInBase64 = CryptoJS.enc.Base64.stringify(hash);

			jqXHR.setRequestHeader("API_PUBLIC_KEY", publicKey);
			jqXHR.setRequestHeader("API_REQUEST_SIGNATURE", hashInBase64);
			jqXHR.setRequestHeader("API_REQUEST_DATE", now);
		}
	}
});
```
##System Logout

The user can logout from the system through the API in the path /Logout. All short term API Key will be removed from the system.



