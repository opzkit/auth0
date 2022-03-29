# README
1. Install the [Auth0 Account Link](https://auth0.com/docs/customize/extensions/account-link-extension) extension
2. Customize the extension if necessary according to the [docs](https://auth0.com/docs/customize/extensions/account-link-extension)
3. Edit the created `auth0-account-link-extension` rule under `Auth Pipeline -> Rules` as below:

Locate the following in the code:
````javascript
function promptUser() {
return searchUsersWithSameEmail().then(function transformUsers(users) {
````
and replace it with:
```javascript
function promptUser() {
  function disableSocialLogin() {
    {
      // initialize app_metadata
      user.app_metadata = user.app_metadata || {};

      const is_social = context.connectionStrategy === context.connection;
      // if it is the first login (hence the `signup`) and it is a social login
      if (context.stats.loginsCount === 1 && is_social) {
        // turn on the flag
        user.app_metadata.is_signup = true;

        // store the app_metadata
        return auth0.users
          .updateAppMetadata(user.user_id, user.app_metadata)
          .then(function () {
            // throw error
            throw new Error('Signup disabled');
          });
      }

      // if flag is enabled, throw error
      if (user.app_metadata.is_signup) {
        throw new Error('Signup disabled');
      }
    }
  }
  return searchUsersWithSameEmail().then(function transformUsers(users) {
```

Then locate the code below and add an else clause calling the `disableSocialLogin` function:

```javascript
}).then(function redirectToExtension(targetUsers) {
  if (targetUsers.length > 0) {
    context.redirect = {
      url: buildRedirectUrl(createToken(config.token), context.request.query)
    };
  }
});
```
resulting code:
```javascript
}).then(function redirectToExtension(targetUsers) {
  if (targetUsers.length > 0) {
    context.redirect = {
      url: buildRedirectUrl(createToken(config.token), context.request.query)
    };
  } else return disableSocialLogin();
});
```


