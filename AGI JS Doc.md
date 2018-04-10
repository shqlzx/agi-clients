<img src="https://files.armorgames.com/misc/ag_logo_horizontal_black.png" height="75" alt="Armor Games">

# Armor Games Javascript AGI

*Version 1.1*

- [Usage](#usage)
  - [Notes](#notes)
  - [Errors](#errors)
- [User Methods](#user-methods)
  - [authenticateUser](#authenticateuser()) 
  - [getUser](#getuser(uid|username,-isusername))
  - [getFriends](#getfriends({-uid,-limit,-offset,-page-}))
- [Storage Methods](#storage-methods)
  - [saveGame](#savegame(key,-value))
  - [retrieveGame](#retrievegame(key))
- [Premium Content Methods](#premium-content-methods)
  - [showStorefront](#showstorefront(sku))
  - [setPurchaseHandler](#setpurchasehandler(callback))
  - [retrievePurchases](#retrievepurchases())
  - [retrievePurchase](#retrievepurchase(sku))
  - [consume](#consume(sku,-quantity))

## Usage

Instantiate the Armor Games API object:
You need to set the document.domain to "armorgames.com" in order for your iframe to have access to our AGI object created on the webpage.

Here is some example code for waiting on the AGI to load and setting the domain correctly.

```javascript
document.domain = "armorgames.com";
(function(){
	var ag = null;
	document.addEventListener("DOMContentLoaded", function(event) {
	    var agiChecks = 0;
	    function checkForAGI() {
	        if (agiChecks > 1000) return;

	        try {
	            if (typeof parent.agi !== 'undefined') {
	                ag = new ArmorGames({
	                    user_id: parent.apiAuth.user_id,
	                    auth_token: parent.apiAuth.auth_token,
	                    game_id: parent.apiAuth.game_id,
	                    api_key: 'E7AC1751-0F3A-4E55-874B-A4C59B1A79D0',
	                    agi: parent.agi
	                    //env: 'stage'
	                });

	                // ... you can start doing AG requests
	            } else {
	                agiChecks++;
	                window.setTimeout(checkForAGI, 250);
	            }
	        } catch(err) {
	            agiChecks++;
	            window.setTimeout(checkForAGI, 250);
	        }
	    }
	    checkForAGI();
	});
());

```

Parameter details:

 * `user_id`: Provided by the game page outside of the containing iframe
 * `auth_token`: Provided by the game page outside of the containing iframe
 * `game_id`: Provided by the game page outside of the containing iframe
 * `api_key`: Visit the developer portal to obtain this key from the game's settings page
 * `env`: (Optional) Specify 'stage' if you would like to test against the Armor Games staging server. _Default: 'production'_

### Notes

When calling certain functions, such as save game, you may notice the first HTTP request fails, and a second one is made that succeeds.
This is because for certain types of requests like game save, or giving quests, we require signed requests.  In order to get the key and
session needed for signing the request, an initial request must be made that fails but returns the key.  So, in short, it is by design.

### Errors
Attach .catch(function(error) {}) to any of the API calls to capture error
conditions. The API uses [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) which should provide a familiar interface.

```javascript
ag.saveGame('stats_level_1', {
    time: 501,
    score: 32,
    duration: 72
}).then(function(response) {
    var key = response.key;
}).catch(function(error) {
    console.error('Uh oh: ', error);
});
```

## User Methods
Methods for accessing information about Armor Games users.

### authenticateUser()
Returns information for the current user.
```javascript
ag.authenticateUser().then(function(user) {
    // user (ArmorGamesUser object)
});
```

### getUser(uid|username, isUsername)
Returns information for the specified user, either by UID or username.
```javascript
ag.getUser(uid).then(function(user) {
    // user (ArmorGamesUser object)
});
```

```javascript
ag.getUser(username, true).then(function(user) {
    // user (ArmorGamesUser object)
});
```

### getFriends({ uid, limit, offset, page })

Returns an array of users who are connected to the specified user.
```javascript
ag.getFriends({
    uid: ...,
    limit: ...,
    offset: ...,
    page:...
}).then(function(users) {
    // users (array of ArmorGamesUser objects)
});
```

## Storage Methods
The storage service provides developers the ability to store and retrieve game data.

### saveGame(key, value)
Stores or updates the object for the given key.

Note: Make sure you pass an object to the second parameter.
```javascript
ag.saveGame('stats_level_1', {
    time: 501,
    score: 32,
    duration: 72
}).then(function(response) {
    var key = response.key;
});
```

### retrieveGame(key)
```javascript
ag.retrieveGame('stats_level_1').then(function(response) {
    var data = response.stats_level_1;
    //...
});
```

## Premium Content Methods
The Premium Content service provides the tools needed for a game developer to sell 
products and verify purchases within their games. Armor Games currently supports 
two product types: _​Unlockables_​ and _Consumables​_.

**Unlockable products**​ - player buys item once, can access forever.
**Consumable products​** - player buys finite quantity which are consumed during game play.

The Armor Games team does the initial product setup for developers. This includes 
creating new products, associating marketing assets, and setting prices. Once setup, 
developers can use the following methods to interact with the Premium Content service.

To get started selling products within your game, please contact ​tasselfoot@armorgames.com​.

### showStorefront(sku)
Displays the HTML Storefront modal window atop the game. This window displays product 
information and purchase options to the user. The user can choose to purchase a 
product or simply close the Storefront window.

The developer has basic control over what is displayed in the HTML Storefront modal 
window depending on the params object provided. For example, if a product SKU is 
provided then the Storefront will only list the product for the given SKU. If the 
SKU is omitted, then all active products for the game will be displayed.

```javascript
ag.showStorefront('2wa-product');
ag.showStorefront();
```

### setPurchaseHandler(callback)
Method will be invoked when storefront is closed.
```javascript
ag.setPurchaseHandler(function(payload) {
    // payload = {sku: "2wa-product", is_consumable: false}
});
```

### retrievePurchases()
Retrieve all purchases made by the user. The return is an array of all products 
available to the game, and each element of the array has two objects, "product" 
and "purchase".  "Product" contains details about the product itself, and "purchase" 
has details about the user's purchase or lack of purchase.  For instance, you can check:
`response.purchase.success (boolean)` to see if the user has purchased the product.
If it's a consumable, you can check `response.purchase.data (int)` to see how many
they currently have of that consumable.
```javascript
ag.retrievePurchases().then(function(response) {
    for(var index in response) {
        if(response[index].purchase.success) {
            console.log('You have purchased '+response[index].product.name+'!');
        }
    }
});
```

### retrievePurchase(sku)
Retrieve a single purchase made by the user.
```javascript
ag.retrievePurchase('2wa-cookies').then(function(response) {
    // var remaining = response.purchase.data;
});
```

### consume(sku, quantity)
The consume method will decrease a user’s item quantity by the given amount for 
the provided product SKU. 
```javascript
ag.consume('2wa-cookies:20', 1).then(function(response) {
    // Product "2wa-cookies:20" should be decremented by 1
    // var remaining = response.purchase.data;
});
```