# Default image
Create a new Login/Post login custom action and deploy it.
Then attach it to the login flow.

```javascript
exports.onExecutePostLogin = async (event, api) => {
  const user = event.user
  const color = 'ffaf38'
  if (!!!event.user.identities.find(i => i.isSocial && i.profileData?.picture)) {
    const url = `https://ui-avatars.com/api/?background=${color}&name=${user.given_name}+${user.family_name}&rounded=true` 
    api.idToken.setCustomClaim("picture", url);
  }
};
```
