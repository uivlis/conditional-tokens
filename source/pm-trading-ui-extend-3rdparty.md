# Extending 3rd-party integrations (Analytics, Support, Bugtracking, etc)

If you want to integrate a 3rd party software into the interface, that is applied globally, such as an analytics tool, a chat-support tool or a bugtracker, you'll most likely want to integrate it according to our existing 3rd party integrations. Let's take a look at `src/utils/analytics`, this folder is used for current 3rd party integrations, namely `googleAnalytics` and `intercom`, our chat support tool.

We have implemented our integrations according to GPDA, so they won't drop cookies and track users before they give consent. This is done via our `CookieBanner`, to be found in `src/containers/CookieBannerContainer`, you might need to change this to your needs to include a new integration.

The integration must follow these criteria in order to work with the interface.
- Must implement the same exports as the existing integrations:
  - `THIRD_PARTY_ID`, the identifier used in storage to determine if an integration is allowed or not.
  - `default`, the default export needs to be a function that will initialize the integration. This is usually the code meant to be embedded into the website, in our case `intercom` will inject a `window.Intercom` function, load a script via `document.createElement('script')` and attach itself to the documents body.
  - **All other functionality should be exported from the integrations file.** For example, the google analytics `ga` function, which sends event to google analytics is done via an export called `gaSend`, it will no-op as long as the user has not given consent. You can chose how this functionality behaves without consent for your own needs, for example you might chose to store events until the user has given consent instead of discarding them.

