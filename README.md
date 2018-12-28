# instafeed.es6

Instafeed is a dead-simple way to add Instagram photos to your website. No jQuery required, just good 'ol plain javascript.

This is a continuation and a remake of [Steven Schobert's original instafeed.js](https://github.com/stevenschobert/instafeed.js) in order to provide ES6 support.

## :warning: IMPORTANT! Instagram is changing the API that Instafeed depends on.

Before you decide to use instafeed, be aware that [Instagram is shutting down the API platform](https://developers.facebook.com/blog/post/2018/01/30/instagram-graph-api-updates/) that enables instafeed to work. As of now, instafeed works for some common uses (eg. embedding a single user's feed on a web page), but can't work for more complex uses (eg. retrieving all public images with a particular hashtag, finding posts based on a location, etc).

The platform API will be turned off completely in 2020, which means that instafeed in its current form will stop working then.

For more information on the current limitations of the API, please see the following:

- [Official API status](https://developers.facebook.com/blog/post/2018/01/30/instagram-graph-api-updates/)
- [Issue #345](https://github.com/stevenschobert/instafeed.js/issues/345)
- [Issue #571](https://github.com/stevenschobert/instafeed.js/issues/571)

## Using instafeed

**Examples:**

- [Hemeon.com](http://hemeon.com/) by [Marc Hemeon](https://twitter.com/hemeon)
- [VinThomas.com](http://vinthomas.com/) by [Vin Thomas](https://twitter.com/vinthomas)

## Installation

Setting up Instafeed is pretty straight-forward.

### NPM/Bower

Instafeed is available on NPM:

```sh
npm install instafeed.es6      # npm
```

## Basic Usage

Here's how easy it is to get all images tagged with **#awesome**:

Instafeed with automatically look for a `<div id="instafeed"></div>` and fill it with linked thumbnails. Of course, you can easily change this behavior using [standard options](#standard-options). Also check out the [advanced options](#advanced-options) for some advanced ways of customizing **Instafeed**.

## Requirements

The only thing you'll need to get going is a valid **client id** from Instagram's API. You can easily register for one on [Instagram's website](http://instagram.com/developer/register/).

If you need help with that step, just try Googling ["How to get an Instagram client ID"](https://google.com/search?q=How%20to%20get%20an%20instagram%20client%20id).

## Standard Options

- `clientId` - **Required**. Your API client id from Instagram.
- `accessToken` - A valid oAuth token. Can be used in place of `clientId`.
- `target` - Either the ID name or the DOM element itself where you want to add the images to.
- `template` - Custom HTML template to use for images. See [templating](#templating).
- `get` - Customize what Instafeed fetches. Available options are:
  - `popular` (default) - Images from the popular page
  - `tagged` - Images with a specific tag. Use `tagName` to specify the tag.
  - `location` - Images from a location. Use `locationId` to specify the location
  - `user` - Images from a user. Use `userId` to specify the user.
- `tagName` (string) - Name of the tag to get. Use with `get: 'tagged'`.
- `locationId` (number) - Unique id of a location to get. Use with `get: 'location'`.
- `userId` (number) - Unique id of a user to get. Use with `get: 'user'`.
- `sortBy` (string) - Sort the images in a set order. Available options are:
  - `none` (default) - As they come from Instagram.
  - `most-recent` - Newest to oldest.
  - `least-recent` - Oldest to newest.
  - `most-liked` - Highest # of likes to lowest.
  - `least-liked` - Lowest # likes to highest.
  - `most-commented` - Highest # of comments to lowest.
  - `least-commented` - Lowest # of comments to highest.
  - `random` - Random order.
- `links` - Wrap the images with a link to the photo on Instagram.
- `limit` - Maximum number of Images to add. **Max of 60**.
- `useHttp` - By default, image urls are protocol-relative. Set to `true`
  to use the standard `http://`.
- `resolution` - Size of the images to get. Available options are:
  - `thumbnail` (default) - 150x150
  - `low_resolution` - 306x306
  - `standard_resolution` - 612x612

## Advanced Options

- `before` (function) - A callback function called before fetching images from Instagram.
- `after` (function) - A callback function called when images have been added to the page.
- `success` (function) - A callback function called when Instagram returns valid data. (argument -> json object)
- `error` (function) - A callback function called when there is an error fetching images. (argument -> string message)
- `mock` (bool) - Set to true fetch data without inserting images into DOM. Use with **success** callback.
- `filter` (function) - A function used to exclude images from your results. The function will be
  given the image data as an argument, and expects the function to return a boolean. See the example
  below for more information.

**Example Filter (get username + tagged):**

```js
const feed = new Instafeed({
  get: "user",
  userId: "USER_ID",
  filter: function(image) {
    return image.tags.indexOf("TAG_NAME") >= 0;
  },
});
feed.run();
```

To see a full list of properties that `image` has, see [issue #21](https://github.com/stevenschobert/instafeed.js/issues/21).

## Templating

The easiest way to control the way Instafeed looks on your website is to use the **template** option. You can write your own HTML markup and it will be used for every image that Instafeed fetches.

Here's a quick example:

```js
const feed = new Instafeed({
  get: "popular",
  tagName: "awesome",
  clientId: "YOUR_CLIENT_ID",
  template: '<a class="animation" href="{{link}}"><img src="{{image}}" /></a>',
});
feed.run();
```

Notice the `{{link}}` and `{{image}}`? The templating option provides several tags for you to use to control where variables are inserted into your HTML markup. Available keywords are:

- `{{type}}` - the image's type. Can be `image` or `video`.
- `{{width}}` - contains the image's width, in pixels.
- `{{height}}` - contains the image's height, in pixels.
- `{{orientation}}` - contains the image's orientation. Can be `square`, `portrait`, or `landscape`.
- `{{link}}` - URL to view the image on Instagram's website.
- `{{image}}` - URL of the image source. The size is inherited from the `resolution` option.
- `{{id}}` - Unique ID of the image. Useful if you want to use [iPhone hooks](http://instagram.com/developer/iphone-hooks/) to open the images directly in the Instagram app.
- `{{caption}}` - Image's caption text. Defaults to empty string if there isn't one.
- `{{likes}}` - Number of likes the image has.
- `{{comments}}` - Number of comments the image has.
- `{{location}}` - Name of the location associated with the image. Defaults to empty string if there isn't one.
- `{{model}}` - Full JSON object of the image. If you want to get a property of the image that isn't listed above you access it using dot-notation. (ex: `{{model.filter}}` would get the filter used.)

## Portrait and Landscape Photos

Until **June 1, 2016**, Instagram's API will return square images (with white borders),
regardless of how they were originally uploaded.

If you'd like to get images in their original landscape and portrait forms, you can opt-in
to the API change by editing your Instagram API client, and clicking on the "Migrations" tab:

<img width="757" alt="screen shot 2015-10-31 at 2 02 56 pm" src="https://cloud.githubusercontent.com/assets/896486/10865600/560ad6a6-7fde-11e5-8e14-2013e51eda7c.png">

> Note: If you have the `resolution` option set to `thumbnail` (default), all images will be square regardless of your API settings.

#### Image Size Template Helpers

As of **v1.4.0**, Instafeed includes several helpers you can use in your `template` option to work with the new image sizes. These helpers are meant primarily to help control styling
of the images through CSS.

- `{{width}}` - contains the image's width, in pixels
- `{{height}}` - contains the image's height, in pixels
- `{{orientation}}` - contains the image's orientation. Can be `square`, `portrait`, or `landscape`.

## Getting images from your user account

To fetch images specifically from your account, set the `get` and `userId` options:

```js
const userFeed = new Instafeed({
  get: "user",
  userId: YOUR_USER_ID,
  accessToken: "YOUR_ACCESS_TOKEN",
});
userFeed.run();
```

> Note: `YOUR_USER_ID` option corresponds to your Instagram **account ID (eg: 4385108)**, not your username. If you do not know your account ID, do a quick google search for ["What is my Instagram account ID?"](https://google.com/search?q=What%20is%20my%20Instagram%20account%20ID%3F). There a several free tools available online that will look it up for you.

> Troubleshooting: If you are seeing the error `No user specified. Use the 'userId' option` in your browser console, make sure there are no quotation marks around the value for `userId`. Instafeed is expecting the `userId` as a number, not as a string.

## Pagination

Instafeed has a `.next()` method you can call to load more images from Instagram.
Under the hood, this uses the _pagination_ data from the API. Additionally, there is a helper
`.hasNext()` method that you can use to check if pagination data is available.

Together these options can be used to create a "Load More" button:

```js
// grab out load more button
const loadButton = document.getElementById("load-more");
const feed = new Instafeed({
  // every time we load more, run this function
  after: function() {
    // disable button if no more results to load
    if (!this.hasNext()) {
      loadButton.setAttribute("disabled", "disabled");
    }
  },
});

// bind the load more button
loadButton.addEventListener("click", function() {
  feed.next();
});

// run our feed!
feed.run();
```

## Contributing

If you'd like to contribute a feature or bugfix: Thanks! To make sure your
fix/feature has a high chance of being included, please read [`CONTRIBUTING.md`](./CONTRIBUTING.md) for more details on contributing.

Thank you to all [the contributors](https://github.com/davidcunha/instafeed.es6/graphs/contributors)!

## LICENSE

[MIT License](https://opensource.org/licenses/MIT)
