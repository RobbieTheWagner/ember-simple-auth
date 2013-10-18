[![Build Status](https://travis-ci.org/simplabs/ember-simple-auth.png?branch=master)](https://travis-ci.org/simplabs/ember-simple-auth)

#  Ember.SimpleAuth

Ember.SimpleAuth is a lightweight and unobtrusive library for implementing
token based authentication with [Ember.js](http://emberjs.com) applications. It
has minimal requirements with respect to the application structure, routes etc.
as well as the server interface.

## Token Based Authentication

The general idea behind token based authentication for Ember.js applications is
that instead of e.g. using a session cookie that keeps the authenticated user's
id as it would maybe be done in old-school web applications, the client injects
an `Authorization` header containing the user's unique and secret token in
every AJAX request it makes. The server uses that token to identify the
corresponding user. The header looks something like this on the server side:

```
Authorization: Token token="<secret token>"
```

The client obtains the secret token from the server when the user logs in. This
can happen either via a regular login form where the user enters credentials
and which Ember.SimpleAuth supports out of the box or via an external OpenId or
OAuth provider like
[Facebook](https://developers.facebook.com/docs/facebook-login/getting-started-web/),
[Twitter](https://dev.twitter.com/docs/auth) or
[Google](https://developers.google.com/accounts/docs/OpenID) which can easily
be integrated with Ember.SimpleAuth.

## Usage

Ember.SimpleAuth only requires one route and one template/controller. To enable
it it's best to use a custom initializer:

```js
Ember.Application.initializer({
  name: 'authentication',
  initialize: function(container, application) {
    Ember.SimpleAuth.setup(application);
  }
});
```

The route for logging in can be named anything you want:

```js
App.Router.map(function() {
  this.route('login');
});
```

To wire everything up, the generated `App.LoginController` needs to implement
the respective mixin provided by Ember.SimpleAuth:

```js
App.LoginController = Ember.Controller.extend(Ember.SimpleAuth.LoginControllerMixin);
```

Also, you need to mixin the `Ember.SimpleAuth.LogoutActionableMixin` in the
`App.ApplicationRoute`:

```js
App.ApplicationRoute = Ember.Route.extend(Ember.SimpleAuth.LogoutActionableMixin);
```

The last step is to add a template that renders the login form:

```html
<form {{action login on='submit'}}>
  <label for="identification">Login</label>
  {{view Ember.TextField id='identification' valueBinding='identification' placeholder='Enter Login'}}
  <label for="password">Password</label>
  {{view Ember.TextField id='password' type='password' valueBinding='password' placeholder='Enter Password'}}
  <button type="submit">Login</button>
</form>
```

and to add a logout button somewhere in your layout:

```html
<button {{ action 'logout' }}>Logout</button>
```

At this point the infrastructure for users to log in and out is set up. Also
every AJAX request (unless it's a cross domain request) will include
[the authentication header](#token-based-authentication).

To actually make a route in the application protected and inaccessible when no
user is authenticated, simply implement the
`Ember.SimpleAuth.AuthenticatedRouteMixin` in the respective route:

```js
App.Router.map(function() {
  this.route('protected');
});
App.ProtectedRoute = Ember.Route.extend(Ember.SimpleAuth.AuthenticatedRouteMixin);
```

This will make the route redirect to `/login` (or a different URL if
configured) when no user is authenticated.

The current session can always be accessed as `session`. To display
login/logout buttons depending on whether the user is currently authenticated
or not, simple add something like this to the respective template:

```html
{{#if session.isAuthenticated}}
  {{#link-to 'logout'}}Logout{{/link-to}}
  <p class="navbar-text pull-right">Your are currently signed in</p>
{{else}}
  {{#link-to 'login'}}Login{{/link-to}}
{{/if}}
```

For more examples including custom URLs, JSON payloads, error handling,
handling of the session user etc., see the [examples](#examples).

## The Server side

The only requirement on the server side is that there is an endpoint for
authenticating users that accepts the credentials as JSON via POST and an
endpoint that invalidates the secret token via DELETE. By default these
endpoints are expected as `POST /session` and `DELETE /session` but the
exact URLs can be customized.

The default request JSON sent to `POST /session` is as follows:

```json
{
  session: {
    identification: "<identification of the user - user name, email or whatever your server expects>",
    password:       "<secret!>"
  }
}
```

The response JSON expected by default is:

```json
{
  session: {
    authToken: "<secret token>"
  }
}
```

Both the request as well as the response JSON can be different than these
defaults and customization only needs a minimal amount of code (see
_"Full-fledged example"_ in the examples).

In the case of `DELETE /session` no JSON is sent with the request and none
is expected in the response.

### The `Authorization` header

Once the session has successfully been established via the `POST /session`
endpoint and the server responded with the authentication token, all subsequent
request the client sends will includes the `Authorization` header. That header
includes the authentication token which the server can use to identify the
user. The header looks something like this (roughly based on
[this](http://tools.ietf.org/html/draft-hammer-http-token-auth-01)
Internet-Draft that never became an RFC though):

```
Authorization: Token token="<secret token>"
```

If your server is a [Ruby on Rails](http://rubyonrails.org) application, it
already has
[support for token based authentication built in](http://api.rubyonrails.org/classes/ActionController/HttpAuthentication/Token.html).

**In any case of course you should use `HTTPS` for all communication between
client and server so the authentication token can't be intercepted.**

## Examples

To run the examples you need to have Ruby (at least version 1.9.3) and the
[bundler gem](http://bundler.io) installed. If you have that, you can run:

```bash
git clone https://github.com/simplabs/ember-simple-auth.git
cd ember-simple-auth/examples
bundle
./runner
```

Open [http://localhost:4567](http://localhost:4567) to access the examples.

## Installation

To install Ember.SimpleAuth in you Ember.js application you have several
options:

* If you're using [Bower](http://bower.io), just add it to your
  `bower.json` file:

```js
{
  "dependencies": {
    "ember-simple-auth": "https://github.com/simplabs/ember-simple-auth-component.git"
  }
}
```

* Download a prebuilt version from
  [the releases page](https://github.com/simplabs/ember-simple-auth/releases)
* [Build it yourself](#building)

## Building

To build Ember.SimpleAuth yourself you need to have Ruby (at least version
1.9.3) and the [bundler gem](http://bundler.io) installed. If you have that,
building is as easy as running:

```bash
git clone https://github.com/simplabs/ember-simple-auth.git
cd ember-simple-auth
bundle
bundle exec rake dist
```

After running that you will find the compiled source file (including a minified
version) in the `dist` directory.

If you want to run the tests as well you also need
[PhantomJS](http://phantomjs.org). You can run the tests with:

```bash
bundle exec rake test
```