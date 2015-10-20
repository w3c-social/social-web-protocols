---
layout: default
---

# The Social API

## Status of this document

An overview of the current state of specs that are inputs to the WG: [ActivityPump](http://w3c-social.github.io/activitypump/), the Indieweb ecosystem (including [Micropub](http://indiewebcamp.com/Micropub) and [Webmention](http://indiewebcamp.com/Webmention)) and [SoLiD](https://github.com/solid/solid-spec); all of which are subject to ongoing development. Arranged based on [Social API Requirements](https://www.w3.org/wiki/Socialwg/Social_API/Requirements).

Optimistically, this has the potential to become a unified working draft. Ultimately an implementation of any subsection of this spec should be compatible with the equivalent subsection of one of the aforementioned specs. Sometimes the overlap between spec subsections is unclear (to me), which is where I've written all options out for now.

## Overview

People and the content they create are the core componants of the social web; they make up the social graph. This document describes a standard way in which people can:

* connect with other people and subscribe to their content;
* create, update and delete social content;
* interact with other people's content;
* be notified when other people interact with their content;

regardless of *what that content* is or *where it is stored*.

This provides the core building blocks for interoperable social systems.

This specification is divided into parts that can be implemented independantly as needed, or all together in one system, as well as extended to meet domain-specific requirements. Users can store their social data across any number of compliant servers, and use compliant clients hosted elsewhere to interact with their own content and the content of others. Put simply, this specification tells you:

* how to [expose/consume](#reading) social content (reading), and how to [discover](#discovery) content someone has published.
* [what to post](#content-representation), and where to, to [create](#creating), [update](#updating) or [delete](#deleting) content.
* how to [ask for notifications](#subscribing) about content (subscribing).
* how to [send notifications](#mentioning) about content or users (mentioning).
* how to expose [profiles](#profiles) and [relationships](#relationships).

## Reading

Each stream MUST have a globally unique identifier (HTTP URI). Each object in a stream MUST have a globally unique identifier (HTTP URI) in the `@id` property, and MAY contain *only* this identifier, which can be dereferenced to retrieve all properties of an object.

A `GET` on the identifier retrieves JSON[-LD] representation of the object or stream of objects, *or* an HTML representation from which the equivalent JSON representation can be parsed.

### Content representation

Content SHOULD be represented according to [ActivityStreams](#) (JSON or JSON-LD) but MAY be structured according to an alternative syntax (eg. HTML with Microformats2 or HTML with RDFa).

Content SHOULD be described using the [ActivityStreams](#) vocabulary, but MAY use other vocabularies instead.

<!--
* **ActivityPump**
  * **syntax**: JSON-LD
  * **vocab**: AS2
* **Micropub**
  * **syntax**: form-encoded or JSON
  * **vocabulary**: Microformats
* **SoLiD**
  * **syntax**: RDF (any)
  * **vocabulary**: Any RDF ontology
-->

**TODO:** limit/paging

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>Microformats uses <code>url</code> not <code>@id</code></div>
</div>

**TODO:** Example single object.

**TODO:** Example stream of objects.


## Creating content

`POST` a JSON object (see [content representation](#content-representation)) to the appropriate endpoint.

* **ActivityPump**: `POST` to `"outbox": "..."` (*See [ActivityPump 7.4.1](http://w3c-social.github.io/activitypump/#outbox)*)
* **Micropub**: `POST` to `rel="micropub"` (*See [Micropub](https://indiewebcamp.com/micropub)*)
* **SoLiD**: `POST` to an LDP container (*See [SoLiD - Creating new resources](https://github.com/solid/solid-spec#creating-new-resources)*)

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>Some people want things to be RESTish..</div>
</div>

<!--

|              | ActivityPump | Micropub |
| ------------ | --------------------------------------------- | -------- |
| **Endpoint** | discoverable outbox                           | `rel="micropub"` |
| **Create**   | `{`                                           | Form-encoding: |
|              | ` "@type": "Create",`                         | `h=entry&` |
|              | ` "published": "2015-05-15T13:06:00+02:00",`  | `content=hello+moon&` |
|              | ` "actor": "http://rhiaro.co.uk/about#me",`   | `category[]=indieweb&` |
|              | ` "object": {`                                | `category[]=micropub&` |
|              | `    "content": "hello world",`               | `author=http://rhiaro.co.uk/about#me&` |
|              | `    "category": ["indieweb","micropub"]`     | `published=2015-05-15T13:06:00+02:00` |
|              | `  }`                                         | JSON: |
|              | `}`                                           | `{` |
|              |                                               | `  "type": [h-entry],` |
|              |                                               | `  "properties": {` |
|              |                                               | `    "content": ["hello world"],` |
|              |                                               | `    "category": ["indieweb","micropub"]` |
|              |                                               | `  }` |
|              |                                               | `}` |
| **Update**   |                                               | |
| **Delete**   |                                               | |

-->

### Updating

Updating an object SHOULD have the side effect of notifying those [subscribed](#subscribing) and [mentioned](#mentioning).

* **ActivityPump:** `POST` an AS2 `Update` Activity to the `outbox` endpoint.
* **Micropub:** `POST` an `mp-edit` action to `rel="micropub"` endpoint.
* **SoLiD:** `PUT` or `PATCH` the resource being updated.

### Deleting

Deleting an object SHOULD have the side effect of notifying those [subscribed](#subscribing) and [mentioned](#mentioning).

When an object is deleted, it SHOULD be replaced with a 'tombstone' containing its unique identifier, deleted (or last updated) date, and optionally replacement content (eg. "this post was removed"). Its URI SHOULD return a 410.

*Note: using SHOULD not MUST to allow for silently deleting objects*

* **ActivityPump:** `POST` an AS2 `Delete` Activity to the `outbox` endpoint.
* **Micropub:** `POST` an `mp-delete` action to `rel="micropub"` endpoint.
* **SoLiD:** `DELETE` on the resource being deleted.

<!--
Notifications from updates and deletes are SHOULD not MUST to allow for modular implementation. ie. if I delete a post right now, I don't notify anyone, even those mentioned, or who have replied to it. It's not ideal, but the world/web doesn't break. It's linkrot, but we've managed to cope so far...
-->

## Discovery

One user may [publish](#publishing) one or more streams of content. Streams may be generated automatically or manually, and might be segregated by post type, topic, audience, or any arbitrary criteria decided by the curator of the stream. The result of a `GET` on the HTTP URI of a [profile](#profiles) MAY include links to other streams, which a consumer could follow to read or subscribe to. Eg.

`<link rel="feed" href="http://rhiaro.co.uk/tag/socialwg">`

```HTTP/1.1 200 OK .... Link: <http://rhiaro.co.uk/tag/socialwg>; rel="feed"```

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>How to include additional info, eg. title/description of feed?</div>
</div>

* **h-feed**: `rel="feed"` (*See [h-feed](http://indiewebcamp.com/h-feed#rel_feed)*)
* **ActivityPump**: contains some pre-defined ActivityStreams `Collections`, whose URIs are discoverable from the JSON returned by `GET`ing a user's profile, eg. via the `inbox`, `outbox`, `favorites` properties; but not sure what scope is for linking to arbitrary collections. (*See [ActivityPump - Discovery](http://w3c-social.github.io/activitypump/#endpoint-discovery)*)

## Subscribing

An agent (client or server) may *ask* to be notified of changes to a content object (eg. edits, new replies) or stream of content (eg. objects added or removed from the stream).

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>Lots of ways of doing this, what to use?</div>
</div>

Here are some options...

* **ActivityPump**: The subscriber posts a `Follow` Activity (JSON object) to the target's `inbox` endpoint, and adds the target to the subscriber's `Following` Collection. The target's server adds the subscriber to the target's `Followers` Collection, and subsequently `POST`s all new activities of the target to the subscriber's `inbox` endpoint. (*See [ActivityPump](http://w3c-social.github.io/activitypump/) 7.4.2, 8 and 9.2.4*)
* **SoLiD**: The subscriber sends the keyword `sub` followed by an empty space and then the URI of the resource, to the target's websockets URI. The target's server sends a websockets message containing the keyword `pub`, followed by an empty space and the URI of the resource that has changed, whenever there is a change. (*See [SoLiD - Live Updates](https://github.com/solid/solid-spec#live-updates)*)
* **PubSubHubbub**: The subscriber discovers the target's hub, and sends a form-encoded `POST` request containing values for `hub.mode` ("subscribe"), `hub.topic` and `hub.callback`. When the target posts new content, the target's server sends a form-encoded `POST` to the hub with values for `hub.mode` ("publish") and `hub.url` and the hub checks the URL for new content and `POST`s updates to the subscriber's callback URL. (*See [PuSH 0.4](http://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html) and [How To Publish And Consume PuSH](http://indiewebcamp.com/How_to_publish_and_consume_PubSubHubbub)*)
* **Salmentions**: The subscriber creates content that links to the target (eg. a reply) and sends a form-encoded `POST` containing values for `source` and `target` to the target's webmention [webmention](https://indiewebcamp.com/webmention) endpoint. The target verifies the link and includes a link back to the subscriber's source on the target content. The target sends form-encoded `POST` requests containing values for `source` and `target` to the webmention endpoint of every link in the content, including that of the subscriber, to indicate that there has been a change. (*See [webmention](https://indiewebcamp.com/webmention) and [salmentions](https://indiewebcamp.com/salmentions)*)

A server may also receive notifications of changes to content it has *not subscribed* to: see [mentioning](#mentioning).

## Mentioning

*(was 'Notifications')*

A user may wish to push a notification to another user, for example because they have linked to (replied, liked, bookmarked, reposted, ...) their content or linked to (tagged, addressed) the user directly.

* **ActivityPump:** When an Activity is posted to a user's `outbox` endpoint, the server checks for values of `object`, `target`, `inReplyTo`, `to`, `cc`, and `bcc`; discovers the `inbox` endpoint of any objects found, and `POST`s the Activity to the discovered `inbox` endpoints. Servers receiving such an Activity proceed to do the same for the target object to propagate the update further. *(See [ActivityPump 8.2](http://w3c-social.github.io/activitypump/#notification))*
* **Webmention:** The target publishes a link to their 'webmention endpoint' via `rel="webmention"`. The source sends a form-encoded `POST` request containing values for `source` (the URL of a webpage with a link to the target) and `target` (the URL of the webpage being linked to). The target MUST validate that the source really does link to target, and proceeds to do with this information as desired. *(See [webmention](https://indiewebcamp.com/webmention))*

*Note: we need to leave it open for users to refuse content they have not explicitly subscribed to, ie. nothing else should rely on implementation of Mentioning.*

## Profiles

The subject of a profile document can be a person, persona, organisation, bot, location, ...whatever. Each profile document MUST have a globally unique identifier (HTTP URI). Performing a `GET` on a profile document SHOULD return a JSON object containing attributes of the subject of the profile; MAY return objects the subject has created, such as an ActivityStreams `Collection`; and SHOULD return at least one link to a stream of content (see [discovery](#discovery)). The JSON representation of a profile document MAY be parsed from an HTML representation (eg. via Microformats (`h-card`) or RDFa).

### Relationships

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>Unsolved...</div>
</div>

*Note: a user should not be required to publish their friends/followers, or may selectively publish them. However, if they're going to (which is useful for eg. switching readers without having to resubscribe to everyone) we should make sure there's a standard way of doing it.*

*Note: I think defining a vocabulary for types of relationships is out of scope and generally not very useful.*

* **ActivityPump:** When a server receives a `Follow` Activity in its `inbox`, the subject is added to a `Followers` `Collection`, which is discoverable from the subject's profile.

### Authorization and access control

* Bearer tokens for authentication
* Leave obtaining the bearer token out of the spec, since there are already several RFCs for ways to obtain bearer tokens.

**TODO:** Access control

* **ActivityPump:** see [auth](http://w3c-social.github.io/activitypump/#authorization)
* **Indieweb:** see [private posts](https://indiewebcamp.com/private_posts), [private webmention](https://indiewebcamp.com/private-webmention)
* **SoLiD:** see [acl](https://github.com/solid/solid-spec#web-access-control)