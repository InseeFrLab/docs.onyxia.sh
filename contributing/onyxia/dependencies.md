---
description: Technologies at play in Onyxia-web
---

# ⚙ Technical stack

To find your way in Onyxia, the best approach is to start by getting a surface-level understanding of the libraries that are leveraged in the project.

{% hint style="info" %}
Modules marked by 🐔 are our own.
{% endhint %}

## Typescript

We are fully committed on keeping everything type safe. If you are a seasoned developer but not fully comfortable with TypeScript yet a good way to get you quickly up to speed is to go through the [What's new section](https://www.typescriptlang.org/docs/handbook/release-notes/overview.html) of the official website.

{% hint style="info" %}
You can skip anything related to `class` we don't do OOP in the project.
{% endhint %}

### tsafe 🐔

{% embed url="https://www.tsafe.dev" %}

We also heavily rely on [tsafe](https://github.com/garronej/tsafe). It's a collection of utilities that help write cleaner TypeScript code. It is crutial to understand at least [`assert`](https://docs.tsafe.dev/assert), [id](https://docs.tsafe.dev/id), [Equals](https://docs.tsafe.dev/equals) and [symToStr](https://docs.tsafe.dev/symtostr) to be able to contribute on the codebase.

### TS-CI 🐔

{% embed url="https://github.com/garronej/ts-ci" %}

We try, whenever we see an opportunity for it, to publish as standalone NPM module chunks of the code we write for Onyxia-web. It help keep the complexity in check. We use TS-CI as a starter for everything we publish on NPM.

## For working on what the end user 👁

Anything contained in the [src/ui](https://github.com/InseeFrLab/onyxia-web/tree/main/src/ui) directory.

### Onyxia-UI 🐔

{% embed url="https://github.com/InseeFrLab/onyxia-ui" %}

The UI toolkit used in the project, you can find the setup of [onyxia-UI](https://github.com/InseeFrLab/onyxia-ui) in onyxia-web here: [src/ui/theme.tsx](https://github.com/InseeFrLab/onyxia/blob/main/web/src/ui/theme.tsx).

#### [MUI](https://mui.com) integration

[Onyxia-UI](https://github.com/InseeFrLab/onyxia-ui) is fully compatible with [MUI](https://mui.com).

Onyxia-UI offers [a library of reusable components](https://inseefrlab.github.io/onyxia-ui) but you can also use [MUI](https://mui.com) components in the project, their aspect will automatically be adapted to blend in with the theme.

#### 🎨 Palettes

We currently offers builtin support for [four color palettes](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/web/src/ui/theme.tsx#L76-L88):

* France: [datalab.sspcloud.fr?**THEME\_ID=france**](https://datalab.sspcloud.fr/?THEME\_ID=france)
* Ultraviolet: [datalab.sspcloud.fr?**THEME\_ID=ultraviolet**](https://datalab.sspcloud.fr/?THEME\_ID=ultraviolet)
* Verdant: [datalab.sspcloud?**THEME\_ID=verdant**](https://datalab.sspcloud.fr/?THEME\_ID=verdant)
* Onyxia (default): [datalab.sspcloud.fr?**THEME\_ID=onyxia**](https://datalab.sspcloud.fr)

You can also [provide your own palette](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/web/.env#L50-L71).

#### 🔡 Fonts

The fonts are loaded in the [public/index.html](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/public/index.html#L6-L115). It's important to keep it that way for Keycloakify.

#### 🔡 Linking onyxia-ui in onyxia-web

To release a new version of [Onyxia-UI](dependencies.md#typescript). You just need to bump the [package.json's version](https://github.com/InseeFrLab/onyxia-ui/blob/470fdb4e54e2b16051ff8b7442ea4d765d76ba92/package.json#L3) and push. [The CI](https://github.com/garronej/ts-ci) will automate publish [a new version on NPM](dependencies.md#typescript).

If you want to test some changes made to onyxia-ui in onyxia-web before releasing a new version of onyxia-ui to NPM you can link locally onyxia-ui in onyxia-web.

```bash
cd ~/github
git clone https//github.com/InseeFrLab/onyxia
cd onyxia/web
yarn install

cd ~/github/onyxia #This is just a suggestion, clone wherever you see fit.
git clone https://github.com/InseeFrLab/onyxia-ui ui
cd ui
yarn install
yarn build
yarn link-in-web
npx tsc -w

# Open a new terminal
cd ~/github/onyxia/web
yarn start

```

Now you can make changes in `~/github/onyxia/ui/`and see the live updates. &#x20;

If you want to install/update some dependencies, you must remove the node\_modules, do you updates, then link again. &#x20;

### tss-react 🐔

{% embed url="https://github.com/garronej/tss-react" %}

The library we use for styling.

Rules of thumbs when it comes to styling:

* Every component should accept[ an optional `className`](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/components/App/Footer.tsx#L9)prop it should always [overwrite the internal styles](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/components/App/Footer.tsx#L55).
* A component should not size or position itself. It should always be the responsibility of the parent component to do it. In other words, you should never have `height`, `width`, `top`, `left`, `right`, `bottom` or `margin` in [the root styles](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/components/App/Footer.tsx#L16-L23) of your components.
* You should never have a color or a dimension hardcoded elsewhere than in [the theme configuration](https://github.com/InseeFrLab/onyxia-web/blob/main/src/app/theme.tsx). Use `theme.spacing()` ([ex1](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/components/pages/MyServices/MyServicesCards/MyServicesCard/MyServicesCard.tsx#L24), [ex2](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/components/pages/MyServices/MyServicesCards/MyServicesCard/MyServicesCard.tsx#L31), [ex3](https://github.com/InseeFrLab/onyxia-web/blob/95667d66cc6ee835ede8d9d6a9bca5299d11bc1a/src/app/components/pages/MyServices/MyServicesSavedConfigs/MyServicesSavedConfig/MyServicesSavedConfig.tsx#L30)) and [`theme.colors.useCases.xxx`](https://github.com/InseeFrLab/onyxia-web/blob/08addbc60c820b8306cf8b0ccbe4793bd2f85661/src/app/components/pages/MyServices/MyServicesSavedConfigs/MyServicesSavedConfig/MyServicesSavedConfigOptions.tsx#L23-L32).

### screen-scaler 🐔

{% embed url="https://github.com/garronej/screen-scaler" %}

Onyxia is mostly used on desktop computer screens. It's not worth the effort to create a fully flege responsive design for the UI.  \
screen-scaler enables us to design for a sigle canonical screen size. The library take charge of scaling/shrinking the image. depending on the real size of the screen.  \
It also asks to rotate the screen when the app is rendered in protrait mode. &#x20;

### Storybook

{% embed url="https://storybook.js.org" %}

It enables us to test the graphical components in isolation. [See sources](https://github.com/InseeFrLab/onyxia/tree/main/web/src/stories).

To launch Storybook locally run the following command:

```bash
yarn storybook
```

{% embed url="https://youtu.be/2L7rtAOlqtc" %}
Setting up a new story
{% endembed %}

### cra-envs 🐔

{% embed url="https://github.com/garronej/cra-envs" %}

We need to be able to do:

```bash
docker run --env OIDC_URL="https://url-of-our-keycloak.fr/auth" InseeFrLab/onyxia-web
```

Then, somehow, access `OIDC_URL` in the code like `process.env["OIDC_URL"]`.

In theory it shouldn't be possible, onyxia-web is an SPA, it is just static JS/CSS/HTML. If we want to bundle values in the code, we should have to recompile. But this is where [`cra-envs`](https://github.com/garronej/cra-envs) comes into play.

It enables to run onyxia-web again a specific infrastructure while keeping the app docker image generic.

Checkout [the helm chart](https://github.com/InseeFrLab/paris-sspcloud/blob/812b12e00e8c24f031083ab41949335bd24b9f4b/apps/onyxia/values.yaml#L18-L33):

```
  web:
    replicaCount: 2
    env:
      MINIO_URL: https://minio.lab.sspcloud.fr
      VAULT_URL: https://vault.lab.sspcloud.fr
      OIDC_URL: https://auth.lab.sspcloud.fr/auth
      OIDC_REALM: sspcloud
      TITLE: SSP Cloud
```

* All the accepted environment variables are defined here: [.env](https://github.com/InseeFrLab/onyxia-web/blob/main/.env). They are all prefixed with `REACT_APP_` to be compatible [with create-react-app](https://create-react-app.dev/docs/adding-custom-environment-variables/#adding-development-environment-variables-in-env). Default values are defined in this file.
* Only in development (`yarn start`) [`.env.local`](https://github.com/InseeFrLab/onyxia-web/blob/main/.env.local.sample) is also loaded and have priority over `.env`
* Then, in the code the variable can be accessed [like this](https://github.com/InseeFrLab/onyxia-web/blob/f6e2907e43eea825d39f350207705d564360eb23/src/app/libApi/LibProvider.tsx#L32).

{% hint style="warning" %}
Please try not to access the environment variable to liberally through out the code. In principle they should only be accessed [here](https://github.com/InseeFrLab/onyxia-web/blob/main/src/app/libApi/LibProvider.tsx). We try to keep things [pure](https://en.wikipedia.org/wiki/Pure\_function) as much as possible.
{% endhint %}

{% embed url="https://youtu.be/JaX14cborxE" %}

### powerhooks 🐔

{% embed url="https://github.com/garronej/powerhooks" %}

It's a collection general purpose react hooks. Let's document the few use cases **you absolutely need to understand**:

#### Avoiding useless re-render of Components

For the sake of performance we enforce that every component be wrapped into [`React.memo()`](https://reactjs.org/docs/react-api.html#reactmemo). It makes that a component only re-render if one of their prop has changed.

However if you use inline functions or [`useCallback`](https://reactjs.org/docs/hooks-reference.html#usecallback) as callbacks props your components will re-render every time anyway:

{% embed url="https://stackblitz.com/edit/react-ts-fyrwng?embed=1&file=index.tsx" %}
Playground to explain the usefulness of useConstCallback
{% endembed %}

We always use [useConstCallback](https://github.com/garronej/powerhooks#useconstcallback) for callback props. And [`useCallbackFactory`](https://github.com/garronej/powerhooks#usecallbackfactory) for callback prop in lists.

#### Measuring Components

It is very handy to be able to get the height and the width of components dynamically. It prevents from having to hardcode dimension when we don’t need to. For that we use [`useDomRect`](https://github.com/garronej/powerhooks#usedomrect)\`\`

### Keycloakify 🐔

{% embed url="https://github.com/InseeFrLab/keycloakify" %}

It's a build tool that enables to implement the login and register pages that users see when they are redirected to Keycloak for authentication.

If the app is being run on Keycloak the [`kcContext`](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/web/src/index.tsx#L3) isn't `undefined` and it means shat we should render the login/register pages.

If you want to test, uncomment [this line](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/web/src/keycloak-theme/login/kcContext.ts#L53) and run `yarn start`. You can also test the login pages in a local keycloak container by running `yarn keycloak`. All the instructions will be printed on the console.

The `keycloak-theme.jar` file is automatically [build](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/.github/workflows/ci.yml#L90-L93) and [uploaded as a GitHub release asset](https://github.com/InseeFrLab/onyxia/blob/9ced438bf6bad76a85049d52220617070f6daa79/.github/workflows/ci.yml#L113) by the CI.&#x20;

### type-routes

{% embed url="https://github.com/typehero/type-route" %}

The library we use for routing. It's like [react-router](https://reactrouter.com) but type safe.

### i18nifty 🐔

{% embed url="https://www.i18nifty.dev" %}

For internalization and translation.

### create-react-app

{% embed url="https://create-react-app.dev" %}

{% hint style="warning" %}
We plane to move to Vite when [Keycloakify](https://keycloakify.dev) will support it.
{% endhint %}

The project is a non-ejected [create-react-app](https://reactjs.org/docs/create-a-new-react-app.html) using [typescript template](https://create-react-app.dev/docs/getting-started#creating-a-typescript-app) (you can find [here](https://github.com/garronej/keycloakify-demo-app) the template repo that was used as a base for this project).

We use [react-app-rewired](https://github.com/timarney/react-app-rewired) instead of the default `react-scripts` to be able to use custom Webpack plugins without having to eject the App. The custom webpack plugins that we use are defined here [/config-overrides.json](dependencies.md#setting-up-the-development-environnement). Currently we only one we use is [circular-dependency-plugins](https://www.npmjs.com/package/circular-dependency-plugin).

## For working on 🧠 of the App

Anything contained in the [src/core](https://github.com/InseeFrLab/onyxia/tree/main/src/core) directory.

### redux-clean-architecture 🐔

{% embed url="https://github.com/garronej/clean-redux" %}

The framework used to implement strict separation of concern betwen the UI and the Core and high modularity of the code. &#x20;

### EVT 🐔

{% embed url="https://www.evt.land" %}

EVT is an event management library (like [RxJS ](https://rxjs.dev)is).

A lot of the things we do is powered under the hood by EVT. You don't need to know EVT to work on onyxia-web however, in order to demystify the parts of the codes that involve it, here are the key ideas to take away:

* If we need to perform particular actions when a value gets changed, we use[`StatefullEvt`](https://docs.evt.land/api/statefulevt).
* We use `Ctx`to detaches event handlers when we no longer need them. (See line 108 on [this playground](https://stackblitz.com/edit/evt-playground?embed=1\&file=index.ts\&hideExplorer=1))
* In React, we use the [useEvt](https://docs.evt.land/react-hooks) hook to work with DOM events.
