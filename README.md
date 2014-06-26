# Notifier

The service for sending notifications.

## Description

The `notifier` is a component with one responsibility of sending notifications. At the moment it provides HTTP API that recieves the event and turning that event into correponding notification.

### How it works?

Threre are 2 parts of `notifier`: HTTP [server](/source/server.js) and [jobs](/source/jobs.js). Both are using `respawn` to get up and be restarted in case of crash.

HTTP servers recieves events and turn those events into `actions`. Then `jobs` is scheduling two tasks [resolve](/source/jobs/resolve.js) and [execute](/source/jobs/execute.js). Resolvers are taking care of transformation of `action` to the form ready to be executed. Executors actually runs the `action`.

## API

To initialize the `notifier` you should create 3 entities - `actions`, `resovers` and `executors`.

The entry point of application responsible for initializing the `notifier`.

```js
var notifier = require('./source/notifier');

notifier.run();
```

## Recieving an action

`notifier` exposes `.action()` call to initialize particular action. The action `callback` is called then `server` recieves event with defined type.

```js
notifier.action('user-registered', function (event, actions, callback) {
	actions.create('send-welcome-email', {user: event.user}, callback);
});
```

You can define as many actions as you need for same event.

```js
notifier.actions('user-payment-recieved', function (event, actions, callback) {
	actions.create('send-invoice-email', {user: event.user, payment: event.amount}, callback);
});


notifier.actions('user-payment-recieved', function (event, actions, callback) {
	actions.create('notify-developers-sms', {user: event.user}, callback);
});
```

### Resolving an action

To resolve an action, `notifier` should define resolved. Usually resolve calls database or other service for additional data.

```js
notifier.resolve('user-registered', function (action, callback) {
	db.user.findOne({email: action.email}, function (err, user) {
		if (err) {
			return callback(err);
		}

		var data = {
			email: user.email,
			firstName: user.firstName,
			secondName: user.secondName,
			registered: user.registered
		};

		callback(null, action, data);
	});
});
```

## Executing action

Once action got resolve, it's ready to be executed.

```js
notifier.execute('user-registered', function (action, transport, callback) {
		var vars = [
			{name: 'FIRST_NAME', content: action.data.firstName}
			{name: 'SECOND_NAME', content: action.data.secondName}
			{name: 'REGISTERED_DATE', content: action.data.registered}
		];

		tranport.mandrill.sendTemplate(action.email, vars, 'welcome-email', callback);
});
```

The `callback` should recieve (err, action, data) - error, same action and resolved data.

## Transports

Madrill, SendGrid, MailGun, Twillio etc..

TDB.

## How to use?

TDB.

# Licence (MIT)

Copyright (c) 2014, Likeastore.com info@likeastore.com

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.