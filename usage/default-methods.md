# Default Methods

With a bare instance of WpApiClient you will get methods to retreive, add and
update any post, page, media item, post category or post tag.
To instantiate a WP-API Client you need to pass the base URL of your WordPress
website to the constructor.

With the second argument, the constructor options,
you can define authentication, default headers, and an onError function.

?> if `options.onError` is undefined, any error will be printed to the console

## Methods List

```typescript
import { WpApiClient } from 'wordpress-api-client'
const client = new WpApiClient('https://my-wordpress-website.com')


client.post().create()
client.post().find()
client.post().delete()
client.post().update()
client.post().revision().create()
client.post().revision().find()
client.post().revision().delete()
client.post().revision().update()

client.page().create()
client.page().find()
client.page().delete()
client.page().update()
client.page().revision().create()
client.page().revision().find()
client.page().revision().delete()
client.page().revision().update()

client.comment().create()
client.comment().find()
client.comment().delete()
client.comment().update()

client.postCategory().create()
client.postCategory().find()
client.postCategory().delete()
client.postCategory().update()

client.postTag().create()
client.postTag().find()
client.postTag().delete()
client.postTag().update()

client.media().create()
client.media().find()
client.media().delete()
client.media().update()

client.user().create()
client.user().find()
client.user().findMe()
client.user().delete()
client.user().deleteMe()
client.user().update()

client.plugin().create()
client.plugin().find()
client.plugin().delete()
client.plugin().update()

client.applicationPassword().create()
client.applicationPassword().find()
client.applicationPassword().delete()
client.applicationPassword().update()

client.reusableBlock().create()
client.reusableBlock().find()
client.reusableBlock().delete()
client.reusableBlock().update()
client.reusableBlock().autosave().create()
client.reusableBlock().autosave().find()

client.blockDirectory()
client.blockType()
client.postType()
client.search()
client.siteSettings()
client.status()

WpApiClient.addCollection()
WpApiClient.collect()
WpApiClient.clearCollection()

```

## .find(...ids: number[])

### find all

To retrieve a list of all of your site's posts, call `await client.post().find()`.
The response will be empty if no posts were found, otherwise it is paginated at
100 objects per page.

Below is an example how to change up the default query parameters, e.g. if you would
like to change the defaults for the `.post` methods. But you can also
[modify the query parameters](#find-with-params) directly on any `.find` method.

```typescript
import WpApiClient, {
	END_POINT,
    DefaultEndpointWithRevision,
	WPPost,
} from 'wordpress-api-client'
import { baseURL } from './constants'
import { CustomPage } from './types'

export class CmsClient extends WpApiClient {
    constructor() {
        super(baseURL)
    }

    public post<P = WPPost>(): DefaultEndpointWithRevision<P> {
		const queryParams = new URLSearchParams({
			_embed: 'true',
			order: 'asc',
			per_page: '8',
		})
        return {
			...this.defaultEndpoints(END_POINT.POSTS, queryParams),
			revision: {
				...this.defaultEndpoints(END_POINT.POSTS + '/revisions', queryParams)
			},
		}
    }
}
```

### find one or many

Specific posts can be retrieved via post id, e.g.:

```typescript
const [frontPage, contactPage, productPage] = await client.page().find(12, 34, 123)
```

?> **Note:** If there is an error (e.g. [authentication](usage/authentication.md)),
the respective promise will resolve to `null`.

?> You do not need to be authenticated to retrieve password-protected posts/pages
– the password musst be appended as URLSearchParams:

### find with params

Query paramenters can be added/modified for any `.find` method by simply providing
an instance of URLSearchParams to it, as the first parameter:

```typescript
const posts = await client.post().find(new URLSearchParams({ per_page: '12' }))
```

### find revisios

You cannot retrieve a list of revisions of all posts, but you can retreive all
revisions for a specific post:

```typescript
const revisions = await client.post(1).revision().find()
```

## .create(body: WPCreate<WPPost>)

When creating new content you should be aware of a couple of things, most of which
an internal function of this package automatically takes care of:

- You need to be [authenticated](usage/authentication.md)
- The `id` field must be omitted (needs to be designated by WP)
- Unlike the API response objects, the fields `title`, `content` and `excerpt`
  of the request body only accept plain HTML strings
- Taxonomies can be assigned by referencing the respective term IDs, e.g.
  `categories: [ 2, 34 ], tags: [5, 67]`

?> See [Helper Methods](usage/helper-methods.md) for more info on the
`WPCreate` type

## .update(body: WPCreate<WPPost>, id: number)

The pointers above, for the `.create` method, are also valid for `.update`.

## .delete(id: number)

You need to be [authenticated](usage/authentication.md) to use this method.

---

## .media()

This library only supports one way of uploading media to your WP Media Library:

```typescript
client.media().create(
	fileName: string,
	data: Buffer,
	mimeType?: string,
)
```

The `data` parameter only accepts a Buffer which will be base64-encoded for transmission.
This makes import-jobs uncomplicated, where you can buffer a file from disk and
do not have to care about encoding. But there is always

```typescript
Buffer.from('my-encoded-data')
```

if your to-be-uploaded media is a string (e.g. a file retrieved via HTTP request).

---

## .user()

Besides the usual `.create`, `.delete`, `.find`, and `.update` there are
two additional `.user` methods, for the currently **logged-in user**: `.findMe`
and `.deleteMe`. In order to delete or to retrieve their information,
you do not have to provide a parameter – but you need to be [authenticated](usage/authentication.md).

---

## .search()

You can simply search by text and/or modify the search query:

```typescript
const searchResults = await client.search(
	'Search by string.',
	new URLSearchParams({ per_page: '25' }),
)
```

The response is an array of `WP_REST_API_Search_Result`s.

---

## .plugin()

You can list, install, activate and deactivate plugins with the client, although
you need to be [authenticated](usage/authentication.md) to use this method.

---

## .theme()

The `.theme` method only returns a list of the installed themes.