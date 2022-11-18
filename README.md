# DevConnector

> Social network for developers

This is a MERN stack application from the "MERN Stack Front To Back" course. It is a small social network app that includes authentication, profiles and forum posts.


**DO NOT SHARE ANY TOKENS THAT HAVE PERMISSIONS**
This would leave your account or repositories vulnerable, depending on permissions set.

It would also be worth adding your `default.json` config file to `.gitignore`
If git has been previously tracking your `default.json` file then...

```bash
git rm --cached config/default.json
```

Then add your token to the config file and confirm that the file is untracked with `git status` before pushing to GitHub.
GitHub does have your back here though. If you accidentally push code to a repository that contains a valid access token, GitHub will revoke that token. Thanks GitHub 🙏

You'll also need to change the options object in `routes/api/profile.js` where we make the request to the GitHub API to...

```javascript
const options = {
  uri: encodeURI(
    `https://api.github.com/users/${req.params.username}/repos?per_page=5&sort=created:asc`
  ),
  method: 'GET',
  headers: {
    'user-agent': 'node.js',
    Authorization: `token ${config.get('githubToken')}`
  }
};
```

### npm package request deprecated

As of 11th February 2020 [request](https://www.npmjs.com/package/request) has been deprecated and is no longer maintained.
We already use [axios](https://www.npmjs.com/package/axios) in the client so we can easily change the above fetching of a users GitHub repositories to use axios.

Install axios in the root of the project

```bash
npm i axios
```

We can then remove the client installation of axios.

```bash
cd client
npm uninstall axios
```

Client use of the axios module will be resolved in the root, so we can still use it in client.

Change the above GitHub API request to..

```javascript
const uri = encodeURI(
  `https://api.github.com/users/${req.params.username}/repos?per_page=5&sort=created:asc`
);
const headers = {
  'user-agent': 'node.js',
  Authorization: `token ${config.get('githubToken')}`
};

const gitHubResponse = await axios.get(uri, { headers });
```

```javascript
import uuid from 'uuid';
```

to

```javascript
import { v4 as uuidv4 } from 'uuid';
```

And where we use it from

```javascript
const id = uuid();
```

to

```javascript
const id = uuidv4();
```

## Addition of normalize-url package 🌎

Depending on what a user enters as their website or social links, we may not get a valid clickable url.
To solve this we brought in [normalize-url](https://www.npmjs.com/package/normalize-url) to well.. normalize the url.

Regardless of what the user enters it will ammend the url accordingly to make it valid (assuming the site exists).
## Fix broken links in gravatar 🔗

There is an unresolved [issue](https://github.com/emerleite/node-gravatar/issues/47) with the [node-gravatar](https://github.com/emerleite/node-gravatar#readme) package, whereby the url is not valid. Fortunately we added normalize-url so we can use that to easily fix the issue. If you're not seeing Gravatar avatars showing in your app then most likely you need to implement this change.

## Redux subscription to manage local storage 📥

The rules of redux say that our [reducers should be pure](https://redux.js.org/basics/reducers#handling-actions) and do just one thing.

If you're not familiar with the concept of pure functions, they must do the following..

1. Return the same output given the same input.
2. Have no side effects.

So our reducers are not the best place to manage local storage of our auth token.
Ideally our action creators should also just dispatch actions, nothing else. So using these for additional side effects like setting authentication headers is not the best solution here.

Redux provides us with a [`store.subscribe`](https://redux.js.org/api/store#subscribelistener) listener that runs every time a state change occurs.

We can use this listener to **_watch_** our store and set our auth token in local storage and axios headers accordingly.

- if there is a token - store it in local storage and set the headers.
- if there is no token - token is null - remove it from storage and delete the headers.

## Log user out on token expiration 🔐

If the Json Web Token expires then it should log the user out and end the authentication of their session.

We can do this using a [axios interceptor](https://github.com/axios/axios#interceptors) together paired with creating an instance of axios.  
The interceptor, well... intercepts any response and checks the response from our api for a `401` status in the response.  
ie. the token has now expired and is no longer valid, or no valid token was sent.  
If such a status exists then we log out the user and clear the profile from redux state.

## Remove Moment 🗑️

As some of you may be aware, [Moment.js](https://www.npmjs.com/package/moment) which [ react-moment ](https://www.npmjs.com/package/react-moment) depends on has since become _legacy code_.\
The maintainers of Moment.js now recommend finding an alternative to their package.

> Moment.js is a legacy project, now in maintenance mode.\
>  In most cases, you should choose a different library.\
>  For more details and recommendations, please see Project Status in the docs.\
>  Thank you.

Some of you in the course have been having problems installing both packages and meeting peer dependencies.\
 We can instead use the browsers built in [Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) API.\

```javascript
function formatDate(date) {
  return new Intl.DateTimeFormat().format(new Date(date));
}

export default formatDate;
```

```javascript
import formatDate from '../../utils/formatDate';
```

And use it instead of Moment...

```jsx
<td>
  {formatDate(edu.from)} - {edu.to ? formatDate(edu.to) : 'Now'}
</td>
```

## React Router V6 🧭

Since the course was released [React Router](https://reactrouter.com) has been updated to version 6
which includes some breaking changes.
You can see the official migration guide from version 5 [ here ](https://reactrouter.com/docs/en/v6/upgrading/v5).

### To summarize the changes to the course code

Instead of a `<Switch />` we now use a [ `<Routes />` ](https://reactrouter.com/docs/en/v6/api#routes-and-route) component.

The [ `<Route />` ](https://reactrouter.com/docs/en/v6/api#routes-and-route) component no longer receives a **_component_** prop, instead we
pass a **_element_** prop which should be a React element i.e. JSX. Routing is
also now relative to the component.

For redirection and Private routing we can no longer use `<Redirect />`, we now
have available a [ `<Navigate />` ](https://reactrouter.com/docs/en/v6/api#navigate) component.

We no longer have access to the **_match_** and **_history_** objects in our
component props. Instead of the match object for routing parameters we can use
the [**useParams**](https://reactrouter.com/docs/en/v6/api#useparams) hook, and in place of using the history object to _push_
onto the router we can use the [**useNavigate**](https://reactrouter.com/docs/en/v6/api#usenavigate) hook.

The above changes do actually clean up the routing considerably with all
application routing in one place in [App.js](client/src/App.js).
Our [PrivateRoute](client/src/components/routing/PrivateRoute.js) is a good deal
simpler now and no longer needs to use a render prop.

With moving all of the routing to App.js this did affect the styling as all
routes needed to be inside the original `<section className="container">`.
To solve this each page component in App.js (any child of a `<Route />`) gets
wrapped in it's own `<section className="container">`, So we no longer need that
in App.js. In most cases this just replaces the outer `<Fragment />` in the
component.

---

# Quick Start 🚀

### Add a default.json file in config folder with the following

```json
{
  "mongoURI": "<your_mongoDB_Atlas_uri_with_credentials>",
  "jwtSecret": "secret",
  "githubToken": "<yoursecrectaccesstoken>"
}
```

### Install server dependencies

```bash
npm install
```

### Install client dependencies

```bash
cd client
npm install
```

### Run both Express & React from root

```bash
npm run dev
```

### Build for production

```bash
cd client
npm run build
```

### Test production before deploy

After running a build in the client 👆, cd into the root of the project.  
And run...

Linux/Unix

```bash
NODE_ENV=production node server.js
```

Windows Cmd Prompt or Powershell

```bash
$env:NODE_ENV="production"
node server.js
```

Check in browser on [http://localhost:5000/](http://localhost:5000/)

### Deploy to Heroku

If you followed the sensible advice above and included `config/default.json` and `config/production.json` in your .gitignore file, then pushing to Heroku will omit your config files from the push.  
However, Heroku needs these files for a successful build.  
So how to get them to Heroku without commiting them to GitHub?

What I suggest you do is create a local only branch, lets call it _production_.

```bash
git checkout -b production
```

We can use this branch to deploy from, with our config files.

Add the config file...

```bash
git add -f config/production.json
```

This will track the file in git on this branch only. **DON'T PUSH THE PRODUCTION BRANCH TO GITHUB**

Commit...

```bash
git commit -m 'ready to deploy'
```

Create your Heroku project

```bash
heroku create
```

And push the local production branch to the remote heroku main branch.

```bash
git push heroku production:main
```

Now Heroku will have the config it needs to build the project.

> **Don't forget to make sure your production database is not whitelisted in MongoDB Atlas, otherwise the database connection will fail and your app will crash.**

After deployment you can delete the production branch if you like.

```bash
git checkout main
git branch -D production
```

Or you can leave it to merge and push updates from another branch.  
Make any changes you need on your main branch and merge those into your production branch.

```bash
git checkout production
git merge main
```

### License

This project is licensed under the MIT License
