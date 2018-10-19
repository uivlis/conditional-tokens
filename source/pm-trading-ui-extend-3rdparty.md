# Extending 3rd-party integrations (Analytics, Support, Bugtracking, etc)

If you want to integrate 3rd party software into the interface and apply it globally you'll most likely want to integrate it according to our existing 3rd party integrations. This would apply for things like an analytics tool, a chat-support tool or a bugtracker.

Let's take a look at `src/utils/analytics`. This folder is used for current 3rd party integrations, namely `googleAnalytics` and `intercom`, our chat support tool.

We have implemented our integrations according to GPDA, so they won't drop cookies and track users before they give consent. This is done via our `CookieBanner`, to be found in `src/containers/CookieBannerContainer`, you might need to change this to suit your needs if you are attempting a new integration.

The integration must follow these criteria in order to work with the interface.

- And you must implement the same exports as the existing integrations:
  - `THIRD_PARTY_ID`, the identifier used in storage to determine if an integration is allowed or not.
  - `default`, the default export needs to be a function that will initialize the integration. This is usually the code meant to be embedded into the website, in our case `intercom` will inject a `window.Intercom` function, load a script via `document.createElement('script')` and attach itself to the documents body.
  - **All other functionality should be exported from the integrations file.** For example, the google analytics `ga` function, which sends events to google analytics is done via an export called `gaSend`, it will not function if the user has not given consent. You can choose how this functionality behaves without consent for your own needs, for example you might chose to store events until the user has given consent instead of discarding them.

