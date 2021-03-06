# Auth0 Documentation
This is the repository for the Auth0 documentation.

## Contributing Guidelines
* Read and follow the [STYLEGUIDE](STYLEGUIDE.md)
* Consult the [WORDS](WORDS.md) document for Auth0 specific spellings and definitions
* Always use absolute links from the root of the site. Note, that your links should NOT start with `/docs` or any other base path. If the site is hosted with a base like on `auth0.com/docs` the links will be corrected at runtime.
* Do not hard code links to auth0 sites like `docs.auth0.com`, `manage.auth0.com`, etc. Use variables instead such as `${uiUrl}`
* Do not store images in external locations like Dropbox, CloudUp, or the Auth0 CDN. Link to images in this repo using `![](/media/folder/image_name.png)`. The image will get automatically uploaded to the CDN and the link will be transformed.
* Try to keep images to no more than 750 pixels wide

## Contributing

### Reusing content
You can create document fragments that can be reused in other documents in order to avoid duplication of content. The process of including another document is shown below.

First, create your fragment document. The convention for this is to name fragments with an underscore like `_mydocument.md`.

After you create your markdown document you can reference it another document with the relative path.

```
<%= include('../_mydocument.md') %>
```

Additionally, you can send a context to the included document.

```
<%= include('../_mydocument.md', { key: 'value', something: true }) %>
```

### Screenshots
On Mac OS X screenshots need to be taken with Chrome, taking into account the following:

 1. The browser cannot show any plugins, customizations, or bookmarks
 1. The browser cannot be in incognito mode
 1. The browser needs to be resized to the standard size. Using the below script:
  ```bash
  #!/usr/bin/env osascript

  tell application "Chrome" to set the bounds of the front window to {100, 150, 1200, 900}
  ```
 1. Screenshots should use the complete browser window (Control + Shift + 4, then press Space)
 1. Highlighting should use color #0099CC

 Example:

 ![Sample CDN image](https://cdn.auth0.com/docs/img/chrome-sample-screenshot.png)


## Test Procedures
When testing a document or tutorial below are several areas to look for.

1. Ensure that the code in the tutorials is correct and functions as expected
1. Ensure that the steps (1..n) are in an order that makes sense and that there are no missing or duplicate steps.
1. Check for broken links
1. Check for old screenshots (both in auth0's dashboard/product and on referenced third-party sites)
1. Ensure that the code in the seed project that you download functions as expected
1. Check for outdated dependencies (both auth0 dependencies and third-party i.e. node modules, nuget packages, gems, etc.)

## Quickstarts
All quickstart data comes directly from the docs API at `/meta/quickstart`. This means that the quickstart on docs and manage will both consume the same datasource and will always be up to date. To add a new quickstart you simply need to add the markdown document in the appropriate folder: [server-apis](/articles/server-apids), [server-platforms](/articles/server-platforms), [native-platforms](/articles/native-platforms), or [client-platforms](/articles/client-platforms). The only requirement is that you need to specify the correct front matter.

For all quickstart docs, provide the following:

```yaml
---
title: Document Title
name: Quickstart Name
image: //cdn.auth0.com/path/to/icon.png
thirdParty: true|false  # For server apis only
hybrid: true|false # For native platforms only
snippets:
  dependencies: folder/dependencies.html
  setup: folder/app.js
  use: folder/login.js
---
```

For each quickstart you must specify the snippets and create the associated snippet file in the `/snippets` folder. You must provide all three snippets for each quickstart: `dependencies`, `setup`, and `use`. To include the snippet in the quickstart document simply reference it:

```md
${snippet(meta.snippets.use)}
```

After you publish the doc update, the new quickstart will automatically appear on both docs and manage.

Additionally, the quickstart configuration is also in this repository here: [quickstart.yml](/quickstart.yml).

## API

> WARNING: This API is for Auth0 internal use only. You should not rely on the docs API for anything. There is *NO SLA or support* for the document API.

All document content is accessible through the docs API as well as through regular HTML.

To request a document in embedded format (i.e. no template html) to embed externally simply append `?e=1` to any url.

To request a document in the framed format (i.e. no header, footer, or naviation) to use in a window popup or an iframe simply append `?framed=1` to any url.

To request content in JSON or JSONP format simply pass the header `Accept: 'application/json'` and optionally specify a ``?callback=foo` query parameter for JSONP.

In the JSON or JSONP responses you can also request the document metadata by appending `?m=1` to the query.

## Code snippets
Code snippets are available both through the API and to use in markdown docs.

Access via API:

GET: `/docs/meta/snippets/{hash}`
Response:
```json
{
  "title": "{title}",
  "content": "{<pre><code class="{class}">{code}</code></pre> }",
  "hash": "{hash}"
  "additional_metadata": ""
}
```

Use in markdown docs:

```md
${snippet('{hash}')}
```

## Connections
Connections are available both through the API and to use in markdown docs.

Access via API:

GET: `/docs/meta/connections`
GET: `/docs/meta/connections/{type}`
GET: `/docs/meta/connections/{type}/{connection}`
Response:
```json
{
  "title": "{title}",
  "hash": "{hash}",
  "url": "https://auth0.com/docs/path/to/doc",
  "image": "https://cdn.auth0.com/docs/path/to/image.png",
  "type": "database|social|enterprise",
  "alias": [
    "alias1"
  ]
}
```

## Parameter Aliases
When writing docs you can use the following variables instead of hard coding these values. Within any markdown document simply use `${variableName}` to reference the value.

### Common Variables

| Variable  | Description | Default Value |
| :---------------------------- | :----------------------------------------- | :-------------------------------------- |
| `uiURL`                       | The url to the management portal.          | `https://manage.auth0.com`              |
| `auth0js_url`                 | The url to the auth0.js CDN location.      | |
| `auth0js_url_no_scheme`       | The url to the auth0.js CDN location without the 'https:'. | |
| `auth0_angular_url`           | The url to the auth0 angular script CDN location. | |
| `auth0_angular_url_no_scheme` |  The url to the auth0 angular script in the CDN location without the 'https://'. | |
| `widget_url`                  | The url to the Lock script CDN location.   | |
| `widget_url_no_scheme`        | The url to the Lock script CDN location without the 'https:'. | |
| `lock_passwordless_url`       | The url to the Passwordless Lock script CDN location. | |


### User Specific Variables

| Variable  | Description | Default Value |
| :--------------------- | :------------------------------------------------- | :------------------------------------- |
| `account.appName`      | The name of the current Auth0 app.                 | `YOUR_APP_NAME'                        |
| `account.tenant`       | The name of the current Auth0 tenant.              | `YOUR_TENANT'                          |
| `account.namespace`    | The name of the current Auth0 namespace.           | `YOUR_NAMESPACE`                       |
| `account.clientId`     | The Client ID of the current Auth0 app.            | `YOUR_CLIENT_ID`                       |
| `account.clientSecret` | The Client Secret of the current Auth0 app.        | `YOUR_CLIENT_SECRET`                   |
| `account.callback`     | The first callback URL of the current Auth0 app.   | `http://YOUR_APP.auth0.com/callback`   |
