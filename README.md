Jiron
=====

A self-documenting, progressive discovery API protocol ideal for AI agents.

Jiron is a faster, more token-efficient alternative to the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/getting-started/intro).

Make your API self documenting and your clients adaptable to API changes. It's all in the hypermedia type. Jiron is a hypermedia type based on Siren and the pug template language. Instead of delivering your data in JSON messages, pack that data with related links, actions that clients can request on the reperesented resource, even code on demand to upgrade the client's capabilities to deal with the given resource.

> Jiron: Part or small portion of a whole: patch of fog.


## Justification

APIs are becoming crucially important to AI agents and application development in general, but there are a few problems with the current state of types used for APIs:


### JSON troubles

JSON is a defacto standard format for types, used in the MCP specification, and the syntax is relatively terse, but:

1. It meets very few requirements for proper hypermedia support.
2. What few hypermedia types for JSON that do exist tend to have far fewer affordances than HTML.
3. It consumes far more tokens than Jiron to represent the same information.


### HTML troubles

HTML is the most popular hypermedia type in the world, but:

1. We take it for granted as a markup language for UI. It's hard to imagine how to apply it to good API design.
2. There are no popular standards or implementations of HTML as an API type.
3. HTML is verbose, and expensive over the network compared to JSON.


### Siren troubles

Siren is an elegant way to add hypermedia semantics to JSON, including:

* Links and relations
* Entities with classes
* Actions with templates
* Metadata

The thing that makes Siren great is that it defines a set of semantics that revolve around the concept of entities -- which happen to map really well to resource representations. Put Siren semantics on top of a resourceful API, and you've got a recipe for a highly scalable, highly decoupled architecture.

Even with the extra affordances that Siren provides, it is still not as rich as HTML in what you can express with it. In particular, it lacks:

* Code on demand
* Style support


### pug

pug is a very terse, elegant markup syntax that was originally designed as a template language that compiles to HTML. One day as I was contemplating the problems we have with our current breed of hypermedia types, it occurred to me that pug might be a great format to base a new type on.

Some things it has going for it:

* Supports all the features of HTML
* Light weight syntax
* Easy to learn
* Since it's a template language, it's trivial to create serializers for it
* Since it outputs HTML, it's easy to work with in browsers

Downsides:

* No clear standard for representing server resources and entities
* It has the same shortcomings as HTML


## Solution

What we need is a best of all worlds. Something terse like JSON, but expressive like HTML. A clear way of representing resources and entities. Bonus if we can take advantage of lots of existing tooling in the process.


## Jiron

Jiron is a mapping of Siren semantics combined with the expressive power and affordances of HTML, with the slick syntax of pug.

Here's a sample:

```pug
head(profile='http://ericelliott.me/profiles/resource')
  title Albums
  
body.albums
  h1.title Albums
  ul.properties
    li.property
      p.description A list of albums you should listen to.
    li.property
      // A count of the total number of entities
      // available. Useful for paging info.
      label(for="entityCount") Total count: 
      span.entityCount(id="entityCount") 3

  ul.entities
    li.entity.album
      a(rel='item', href='/albums/a65x0qxr')
        ul.properties
          li.property.name Dark Side of the Moon
          li.property.artist Pink Floyd
    li.entity.album
      a(rel='item', href='/albums/a7ff1qxw')
        ul.properties
          li.property.name Random Access Memories
          li.property.artist Daft Punk

  ul.links
    li.link
      a(rel='next', href='/albums?offset=2&limit=1') Next
    li.link
      link(rel='self, canonical', href='http://albums.com/albums')
```

And the equivalent HTML:

```html
<head profile="http://ericelliott.me/profiles/resource">
    <title>Albums</title>
</head>

<body class="albums">
    <h1 class="title">Albums</h1>

    <ul class="properties">
        <li class="property">
            <p class="description">A list of albums you should listen to.</p>
        </li>

        <li class="property"><!-- A count of the total number of entities-->
        <!-- available. Useful for paging info.-->
        <label for="entityCount">Total count:</label> <span class="entityCount"
        id="entityCount">3</span></li>
    </ul>

    <ul class="entities">
        <li class="entity album">
            <a href="/albums/a65x0qxr" rel="item">
            <ul class="properties">
                <li class="property name">Dark Side of the Moon</li>

                <li class="property artist">Pink Floyd</li>
            </ul></a>
        </li>

        <li class="entity album">
            <a href="/albums/a7ff1qxw" rel="item">
            <ul class="properties">
                <li class="property name">Random Access Memories</li>

                <li class="property artist">Daft Punk</li>
            </ul></a>
        </li>
    </ul>

    <ul class="links">
        <li class="link">
            <a href="/albums?offset=2&amp;limit=1" rel="next">Next</a>
        </li>

        <li class="link">
            <link href="http://albums.com/albums" rel="self, canonical">
        </li>
    </ul>
</body>
```

Here's a translation of the example from the Siren README:

```pug
head
  title Order
body.order
  h1 Order
  ul.properties
    li.property
      label Order Number
        span.orderNumber 42
    li.property
      label Item Count
        span.itemCount 3
    li.property
      label Status
        span.status pending

  ul.entities
    li.entity.items.collection
      a(rel='http://x.io/rels/order-items',
        href='http://api.x.io/orders/42/items')
        | Items

    li.entity.info.customer
      a(rel='http://x.io/rels/customer'
        href='http://api.x.io/customers/pj123'),
        ul.properties
          li.property
            label Customer ID
              span.customerId pj123
          li.property
            label Name
              span.name Peter Joseph

  ul.actions
    li.action
      // Action is one of:
      // index, create, show, put, delete, patch
      // The method in html is determined by the
      // mapping between actions and HTML methods.
      form(action='create',
        href='http://api.x.io/orders/42/items',
        type='application/x-www-form-urlencoded')
        fieldset
          legend Add Item
          label Order Number
            input(name='orderNumber', hidden='hidden', value='42')
          label Product Code
            input(name='productCode', type='text')
          label Quantity
            input(name='quantity', type='number')

  ul.links
    a(rel='self', href='http://api.x.io/orders/42',
      style='display: none;')
    a(rel='previous', href='http://api.x.io/orders/41') Previous
    a(rel='next', href='http://api.x.io/orders/43') Next
```

Which translates to the following HTML:

```html
<head>
    <title>Order</title>
</head>

<body class="order">
    <h1>Order</h1>

    <ul class="properties">
        <li class="property"><label>Order Number<span class=
        "orderNumber">42</span></label></li>

        <li class="property"><label>Item Count<span class=
        "itemCount">3</span></label></li>

        <li class="property"><label>Status<span class=
        "status">pending</span></label></li>
    </ul>

    <ul class="entities">
        <li class="entity items collection">
            <a href="http://api.x.io/orders/42/items" rel=
            "http://x.io/rels/order-items">Items</a>
        </li>

        <li class="entity info customer">
            <a href="http://api.x.io/customers/pj123" rel=
            "http://x.io/rels/customer">,

            <ul class="properties">
                <li class="property"><label>Customer ID<span class=
                "customerId">pj123</span></label></li>

                <li class="property"><label>Name<span class="name">Peter
                Joseph</span></label></li>
            </ul></a>
        </li>
    </ul>

    <ul class="actions">
        <li class="action">
            <!-- Action is one of:-->
            <!-- index, create, show, put, delete, patch-->
            <!-- The method in html is determined by the-->
            <!-- mapping between actions and HTML methods.-->

            <form method="POST" action="http://api.x.io/orders/42/items"
              enctype="application/x-www-form-urlencoded">
                <fieldset>
                    <legend>Add Item</legend> <label>Order Number
                    <input hidden="hidden" name="orderNumber" value=
                    "42"></label> <label>Product Code <input name="productCode"
                    type="text"></label> <label>Quantity <input name="quantity"
                    type="number"></label>
                </fieldset>
            </form>
        </li>
    </ul>

    <div class="links" style="margin-left: 2em">
        <a href="http://api.x.io/orders/42" rel="self" style=
        "display: none;"></a><a href="http://api.x.io/orders/41" rel=
        "previous">Previous</a><a href="http://api.x.io/orders/43" rel=
        "next">Next</a>
    </div>
</body>
```

Since `jiron+pug` is based on an existing HTML template syntax. It can be interpreted programatically by pug processors, and by any sufficiently advanced AI language model.

Using it in the browser is simple:

```js
pugRenderer('a.album(href="/albums/123") Pretty Hate Machine');
```

Which creates the string:

```html
<a href="/albums/123" class="album">Pretty Hate Machine</a>
```

Now you can add that to a `documentFragment` and use CSS selectors to query it. Better yet -- just slap some CSS on it and render it as-is. Try that with JSON.


## Cool party tricks

Here are some cool things you can do with Jiron that you can't do with JSON:

* Deliver code on demand with JavaScript links (including any client SDK you need to customize any Jiron browser for your particular API)
* Deliver default templates that make browsing your API directly in a browser pleasant
* Deliver clickable links and usable forms while your users are browsing the API directly
* Use CSS for styling default views
* Intersperse human-readable text and media with hypertext controls and affordances -- like HTML, only structured specifically for APIs

Tricky with HTML:
* Support any HTTP method type. PUT, PATCH, DELETE? No problem.
* Support HTTP header changes in links:

### Method types

Jiron is purposely very similar to HTML, except where HTML falls short on important affordances. For example, HTML lacks support for important methods, and doesn't prescribe any mapping between intended actions and HTTP method. HTML is also a little inconsistent with the URL for form targets, calling it `action` instead of `href` (which is used in almost all other cases involving links). Jiron can improve on HTML on both counts. Imagine this syntax:

```pug
form(action='create',
  href='http://api.x.io/orders/42/items',
  type='application/x-www-form-urlencoded')
```

Which would map to the following raw (broken) HTML:

```html
<form action="create" href="http://api.x.io/orders/42/items" type="application/x-www-form-urlencoded"></form>
```

And finally be translated to this by the Jiron client:

```html
<form method="POST" action="http://api.x.io/orders/42/items" type="application/x-www-form-urlencoded"></form>
```

### Support for headers in links:

```pug
a(headers='Accept:application/vnd.jiron+pug') Some Jiron resource
```

Which would translate to this, if HTML knew how to deal with it:

```html
<a headers="Accept:application/vnd.jiron+pug">Some Jiron resource</a>
```

The Jiron client  will attach a link activation handler that will set the appropriate accept header and fetch the resource using AJAX.
