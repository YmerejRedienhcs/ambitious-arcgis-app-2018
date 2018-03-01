# Addons

Previously we added the Bootstrap CSS, which works for the page layout and styles, but the components are not interactive. For example, open chrome developer tools (cmd+opt+i) and toggle the device toolbar (cmd+shift+M) to view what the app looks like on a mobile device.

Notice that:
- the nav menu items are hidden
- we can't get back to the home page

We'll need to add bootstrap's button to hide/show the collapsible nav bar.

The easiest way is to use http://www.ember-bootstrap.com/, which is an ember implementation of most Bootstrap components.

## Add ember-bootstrap
- stop app (`cmd+C`)
- `ember install ember-bootstrap`
- configure to use the latest Bootstrap (v4) by running:
`ember generate ember-bootstrap --bootstrap-version=4`
- in app/styles/app.css, delete this line:
`@import "https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css";`
- start app (`ember serve`)

Notice that:
- app looks the same
- no longer making a network request for bootstrap.min.css from CDN
- bootstrap styles have been imported into vendor.css
- the navbar toggle on mobile still doesn't work

## Use ember-bootstrap components
- in app/templates/application.hbs replace the `<nav>` with:

```hbs
{{#bs-navbar class="navbar-expand-md navbar-dark fixed-top bg-dark" as |navbar|}}
  <div class="navbar-header">
    {{navbar.toggle}}
    <a class="navbar-brand" href="#">Ambitious ArcGIS App</a>
  </div>
  {{#navbar.content}}
    {{#navbar.nav as |nav|}}
      {{#nav.item}}
        {{#nav.link-to "index"}}Home{{/nav.link-to}}
      {{/nav.item}}
      {{#nav.item}}
        {{#nav.link-to "items"}}Items{{/nav.link-to}}
      {{/nav.item}}
    {{/navbar.nav}}
  {{/navbar.content}}
{{/bs-navbar}}
```

Notice that:
- you can now use the navbar toggle on mobile

## Add ember-arcgis-portal-services
- stop app (`cmd+C`)
- depends on torii-provider-arcgis, so first [install and configure that](https://github.com/dbouwman/torii-provider-arcgis#usage):
 - `ember install torii`
 - `ember install torii-provider-arcgis`
 - in config/environment.js add the following above `APP`:
```js
torii: {
  sessionServiceName: 'session',
  providers: {
   'arcgis-oauth-bearer': {
      portalUrl: 'https://www.arcgis.com'
    }
  }
},
```
- `ember install ember-arcgis-portal-services`
- remove fake implementation of itemsService:
`ember destroy service items-service`
- `ember serve` and visit `localhost:4200`

Notice that:
- entering search terms returns real results!
- Only getting 10 though, so let's add paging

## Add paging parameters to routes
in app/routes/items.js:
- add these query paramters above the `q` param:

```js
// paging query params
start: { refreshModel: true },
num: { refreshModel: true },
```

- then update the `model()` hook as follows:

```js
return itemsService.search({ q, num: params.num, start: params.start });
```

in app/controllers/items.js:
- add this to the top of the controller:

```js
// query parameters used by components
queryParams: ['start', 'num'],
start: 1,
num: 10,
```

- in `transitionToRoute()` update the `queryParams` as follows:

```js
// for a new query string, sart on first page
queryParams: { q , start: 1 }
```

in app/controllers/index.js:
- in `transitionToRoute()` update the `queryParams` as follows:

```js
// for a new query string, sart on first page
queryParams: { q , start: 1 }
```

- run `ember serve`
- search for `water`
- append the following to query string: `&start=11&num=5` and hit enter
- play around w/ those parameters and see what happens

Notice that:
- you can control the number and starting point for results via query string params
- searching from a different term (either from home or items route) will reset the starting point, but not the number of records shown
