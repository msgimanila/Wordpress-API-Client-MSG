# Defaults & Options

With a bare instance of WpApiClient you will get methods to retrieve, add and
update any post, page, media item, post category or post tag.
To instantiate a WP-API Client you only need to pass the base URL of your WordPress
website to the WpApiClient's constructor.

Refer to the WP [REST API Handbook](https://developer.wordpress.org/rest-api/) for
a quick-start on the WP-REST-API basics.

---

## Constructor Options

With the second constructor argument, the constructor options, authentication can
be configured, auth-protected routes and default headers can be defined, an async
`onError` function can be set and also the API url-base and the list of post types
with trash bins can be adjusted.

?> If `options.onError` is undefined, any error will be printed to the console.

<details>
<summary>Type Definition</summary>

```typescript
interface WpApiOptions {
	auth?: {
		type: 'basic' | 'jwt' | 'nonce' | 'none'
		token?: string // basic
		password?: string // jwt
		user?: string // jwt
		nonce?: string //nonce
	}
	headers?: Record<string, string>
	onError?: (message: string) => Promise<void>
	protected?: {
		GET: string[] // ['wp/v2/users', ...]
		POST: string[] // ['wp/v2/posts', ...]
		DELETE: string[] // ['wp/v2/posts', ...]
	}
	public?: {
		GET: string[] // ['wp/v2/posts', ...]
		POST: string[] // []
		DELETE: string[] // []
	}
	restBase?: string // wp-json
	trashable?: string[] // ['wp/v2/posts', 'wp/v2/pages', 'wp/v2/blocks']
}
```

</details>

---

## Methods List

<details>
<summary><h3 style="display:inline-block" id="methods-list">Methods List</h3></summary>

```typescript
import { WpApiClient } from 'wordpress-api-client'
const client = new WpApiClient('https://my-wordpress-website.com')


client.post().create()
client.post().find()
client.post().update()
client.post().delete()
client.post().total()
client.post().dangerouslyFindAll()
client.post().revision().create()
client.post().revision().find()
client.post().revision().update()
client.post().revision().delete()

client.page().create()
client.page().find()
client.page().update()
client.page().delete()
client.page().total()
client.page().dangerouslyFindAll()
client.page().revision().create()
client.page().revision().find()
client.page().revision().update()
client.page().revision().delete()

client.comment().create()
client.comment().find()
client.comment().update()
client.comment().delete()

client.postCategory().create()
client.postCategory().find()
client.postCategory().update()
client.postCategory().delete()
client.postCategory().total()
client.postCategory().dangerouslyFindAll()

client.postTag().create()
client.postTag().find()
client.postTag().update()
client.postTag().delete()
client.postTag().total()
client.postTag().dangerouslyFindAll()

client.media().create()
client.media().find()
client.media().update()
client.media().delete()
client.media().total()
client.media().dangerouslyFindAll()

client.user().create()
client.user().find()
client.user().findMe()
client.user().delete()
client.user().update()
client.user().deleteMe()
client.user().total()
client.user().dangerouslyFindAll()

client.plugin().create()
client.plugin().find()
client.plugin().update()
client.plugin().delete()

client.applicationPassword().create()
client.applicationPassword().find()
client.applicationPassword().update()
client.applicationPassword().delete()

client.reusableBlock().create()
client.reusableBlock().find()
client.reusableBlock().update()
client.reusableBlock().delete()
client.reusableBlock().total()
client.reusableBlock().dangerouslyFindAll()
client.reusableBlock().autosave().create()
client.reusableBlock().autosave().find()

client.blockDirectory()
client.blockType()
client.postType()
client.renderedBlock()
client.search()
client.siteSettings()
client.status()

```

</details>

Take a look at the [Function Reference](function-reference/README.md) for more details
on all of the class methods.

---

## Default Methods

<details>
<summary>
  <h3 style="display:inline-block" id="find">.find(params?: URLSearchParams, ...ids: number[])</h3>
</summary>

All `.find` methods can take a URLSearchParams object as optional parameter (except
of  `.plugin().find()` and `.siteSettings().find()`). The available query filters
vary depending on the installed WP plugins and themes, although there is, of course,
a set of [defaults](https://developer.wordpress.org/rest-api/using-the-rest-api/),
such as `page`, `per_page`, `order`, `orderby` and many more.

#### find all

To retrieve a list of all of your site's posts, call `await client.post().find()`.
The response will be an empty array if no posts were found, otherwise it will
return a maximum of the first 100 posts. You can find out how many pages of posts
you need to fetch by calling [`.total()`](#total).

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
				...this.defaultEndpoints(
					`${END_POINT.POSTS}/revisions`,
					queryParams,
				)
			},
		}
    }
}
```

#### dangerously find all

The WP REST API is capped at a maximum of 100 entries per page, so any `.dangerouslyFindAll()`
method will perform as many parallel requests as necessary in order to return
**all** results in one response.

If your WordPress hosts tens of thousands of posts, executing this method will most
definitely put a heavy strain both on your WordPress and on your Browser or NodeJS
thread – you should only **use it very carefully**.

#### find one or many

Specific posts can be retrieved via post id, e.g.:

```typescript
const [welcome, contact, products] = await client.page().find(12, 34, 123)
```

?> **Note:** If there is an error with one of the posts, the client will throw
an informative error – requests should always be wrapped in a try-catch block.

?> You do not need to be authenticated to retrieve password-protected posts/pages
– the password must be appended as URLSearchParams:

#### find revision

You cannot retrieve a list of revisions of all posts, but you can retrieve all
revisions for a specific post:

```typescript
const revisions = await client.post(1).revision().find()
```

</details>

<details>
<summary><h3 style="display:inline-block" id="create">.create(body: WPCreate<WPPost>)</h3></summary>

When creating new content you should be aware of a couple of things, most of which
an internal function of this package automatically takes care of:

- You need to be [authenticated](usage/authentication.md)
- The `id` field must be omitted (needs to be designated by WP)
- Unlike the API response objects, the fields `title`, `content` and `excerpt`
  of the request body only accept plain HTML strings
- Taxonomies can be assigned by referencing the respective term IDs, e.g.
  `categories: [2, 34], tags: [5, 67]`

?> See [Helper Methods](usage/helper-methods.md) for more info on the
`WPCreate` type

</details>

<details>
<summary><h3 style="display:inline-block" id="update">
  .update(body: WPCreate<WPPost>, id: number)
</h3></summary>

The pointers above, for the `.create` method, are also valid for `.update`.

</details>

<details>
<summary><h3 style="display:inline-block" id="delete">.delete(id: number)</h3></summary>

You need to be [authenticated](usage/authentication.md) to use this method.
DELETE for objects that have no trash can status (e.g. categories or media) will
be called with the query parameter `force=true`. This behavior can be controlled
with the `trashable` constructor option, which receives a list of end points (e.g.
`trashable: ['wp/v2/media', 'wp/v2/categories']`).

</details>

<details>
<summary><h3 style="display:inline-block" id="total">
  .total()
</h3></summary>

The WordPress REST API is capped at a maximum of 100 posts per page. This means
that if you call `client.post().find(new URLSearchParams({ per_page: '25' }))`
you will get the first set of 25 of all of your posts. If you call
`client.post().find(new URLSearchParams({ per_page: '2500' }))`, on the other
hand, you will only get the first 100 of your posts.

If [`.dangerouslyFindAll()`](#dangerously-find-all) is not an option for you, you can paginate
through all of your posts via URLSearchParams
(`client.post().find(new URLSearchParams({ per_page: '25', page: '2' }))`).

`.total()` tells you how many pages you have to iterate over (with the default 
`per_page=100`) in order to have retrieved a complete list of your posts.

</details>

---