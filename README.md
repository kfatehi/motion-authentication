# motion-authentication

`motion-authentication` aims to provide a simple, standardized authentication helper for common authentication strategies.

Currently, this library only supports iOS, but could easily support other platforms. Please submit an issue (or PR :D) for the platform you would like to support.

Need authorization? Use [`motion-authorization`](https://github.com/andrewhavens/motion-authorization)!

## Installation

Add this line to your application's `Gemfile`, then run `bundle install`:

    gem 'motion-authentication'

Next, run `rake pod:install` to install the CocoaPod dependencies.

## Usage

Start by subclassing `Motion::Authentication` to create your own `Auth` class. Specify your authentication strategy and your sign in URL:

```ruby
class Auth < Motion::Authentication
  strategy DeviseTokenAuth
  sign_in_url "http://localhost:3000/api/v1/users/sign_in"
end
```

Available strategies:

* `DeviseTokenAuth` - This authentication strategy takes `email` and `password`, makes a POST request to the `sign_in_url`, and expects the response to include `email` and `token` keys in the JSON response object.

### `.sign_in`

Using your `Auth` class, call `.sign_in` and pass a hash of credentials:

```ruby
Auth.sign_in(email: email, password: password) do |result|
  if result.success?
    # authentication successful!
  else
    app.alert "Invalid login?"
  end
end
```

A successful sign in will securely store the current user's auth token.

### `.signed_in?`

You can check if an auth token has previously been stored by using `signed_in?`. For example, in your App Delegate, you might want to open your sign in screen when the app is opened:

```ruby
def on_load(options)
  if Auth.signed_in?
    open DashboardScreen
  else
    open SignInScreen
  end
end
```

### `.authorization_header`

After signing in, and receiving the auth token, you will probably want to configure your API client to use your auth token in your API requests in an authorization header. Call `.authorization_header` to return the header value specific to the strategy that you are using. Two common places would be upon sign in, and when your app is launched.

```ruby
# app_delegate.rb
def on_load(options)
  if Auth.signed_in?
    MyApiClient.update_auth_header(Auth.authorization_header)
    # ...
  end
end

# sign_in_screen.rb
def on_load(options)
  Auth.sign_in(data) do |result|
    if result.success?
      MyApiClient.update_auth_header(Auth.authorization_header)
      # ...
    end
  end
end
```

### `.sign_out`

At some point, you're going to need to sign out. This method will clear the stored auth token, but also allows you to pass a block to be called after the token has been cleared.

```ruby
Auth.sign_out do
  open HomeScreen
end
```

### `.current_user`

`motion-authentication` provides a `current_user` attribute. It has no effect on authentication, so you can do whatever you want with it.

```ruby
Auth.sign_in(data) do |result|
  if result.success?
    Auth.current_user = User.new(result.object)
  end
end
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
