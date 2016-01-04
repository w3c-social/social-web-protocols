---
layout: default
---

# Social Web Protocols

## Editor's Draft 01 Janary 2016

**This version:** http://w3c-social.github.io/social-web-protocols/ed

**Latest published version:** http://www.w3.org/TR/social-web-protocols/

**Editor:** [Amy Guy](http://rhiaro.co.uk), University of Edinburgh

**Repository:**

* [Github](https://github.com/w3c-social/social-web-protocols)
* [Issues](https://github.com/w3c-social/social-web-protcols/issues)
* [Commits](https://github.com/w3c-social/social-web-protocols/commits/gh-pages)

## Status of this document

This document was published by the [Social Web Working Group](http://www.w3.org/Social/WG) as an Editor's Draft. If you wish to make specific comments regarding this document please create [github issues](https://github.com/w3c-social/social-web-protocols/issues). More general comments, please send to public-socialweb@w3.org ([subscribe](mailto:public-socialweb-request@w3.org?subject=subscribe), [archives](http://lists.w3.org/Archives/Public/public-socialweb/)). All comments are welcome.

Publication as an Editor's Draft does not imply endorsement by the W3C Membership. This is a draft document and may be updated, replaced or obsoleted by other documents at any time. It is inappropriate to cite this document as other than work in progress.

### Contents

This is an overview of the current state of specs that are inputs to the WG: [ActivityPump](http://w3c-social.github.io/activitypump/), [Micropub](http://micropub.net),  [Webmention](http://webmention.net)) and [SoLiD](https://github.com/solid/solid-spec); all of which are subject to ongoing development. Arranged based on [Social API Requirements](https://www.w3.org/wiki/Socialwg/Social_API/Requirements).

The WG may produce several small 'building block' specifications, or one unified document. In the former case, this document serves as a guide for implementors to demonstrate how the pieces fit together. In the latter case, optimistically, this has the potential to become the location for convergence of the alternatives. Ultimately an implementation of any subsection of this spec should be compatible with the equivalent subsection of one of the aforementioned specs. Subsections which have multiple implementation routes listed are work-in-progress.

### Relationship to other WG drafts

| Spec      | Status | Relationship |
| --------- | ------ | ------------ |
| ActivityStreams2 | WD | The expected representation of JSON data mentioned here |
| ActivityPump | ED | Most parts correspond with a subsection here |
| Webmention | ED | Corresponds with [Mentioning](#mentioning) |
| Micropub  | ED | Corresponds with [Creating content](#creating-content) |
| jf2 | ED | as AS2 where applicable |
| Post Type Discovery | ED | No overlap |

For comparison of the subsections, the aforementioned specs are restructured according to the structure of this document (work-in-progress, pending feedback from spec authors/editors!):

* [ActivityPump restructure](https://github.com/rhiaro/activitypump/blob/restructure/index.md)
* [Indieweb specs](https://github.com/rhiaro/Social-APIs-Brainstorming/blob/gh-pages/indiewebspecs.md)
* [Solid restructure](https://github.com/rhiaro/solid-spec/tree/restructure/README.md)



## Overview

People and the content they create are the core componants of the social web; they make up the social graph. This document describes a standard way in which people can:

* connect with other people and subscribe to their content;
* create, update and delete social content;
* interact with other people's content;
* be notified when other people interact with their content;

regardless of *what that content* is or *where it is stored*.

This provides the core building blocks for interoperable social systems.

This specification is divided into parts that can be implemented independently as needed, or all together in one system, as well as extended to meet domain-specific requirements. Users can store their social data across any number of compliant servers, and use compliant clients hosted elsewhere to interact with their own content and the content of others. Put simply, this specification tells you:

* how to [expose/consume](#reading) social content (reading).
* [what to post](#content-representation), and where to, to [create](#creating), [update](#updating) or [delete](#deleting) content.
* how to [ask for notifications](#subscribing) about content (subscribing).
* how to [send notifications](#mentioning) about content or users (mentioning).
* how to expose [profiles](#profiles) and [relationships](#relationships).

## Profiles

The subject of a profile document can be a person, persona, organisation, bot, location, ... the type of the subject of the profile is not required. Each profile document MUST have a URL which SHOULD return attributes of the subject of the profile; SHOULD return at least one link to [a stream of content](#reading) and MAY return content the subject has created. A JSON format MUST be available; other content types MAY be returned as well.

### Relationships

Semantics and representation of personal relationships are implementation specific. This specification deals with relationships only when distribution of content is affected, for example if one user 'friending' another triggers a subscription request from the first user's server to the second. Lists of other relationships MAY be discoverable from a user profile, SHOULD be represented according to the ActivityStremas 2 syntax and MAY (and are likely to) use extension vocabularies as needed.

* **ActivityPump:** When a server receives a `Follow` Activity in its `inbox`, the subject is added to a `Followers` `Collection`, which is discoverable from the subject's profile.

### Authorization and access control

Servers may restrict/authorize access to content however they want?

* **ActivityPump:** see [auth](http://w3c-social.github.io/activitypump/#authorization)
* **Indieweb:** see [private posts](https://indiewebcamp.com/private_posts), [private webmention](https://indiewebcamp.com/private-webmention)
* **SoLiD:** see [acl](https://github.com/solid/solid-spec#web-access-control)

## Reading

### Content representation

Content MUST be available as [ActivityStreams](#) JSON and MAY additionally be served as an alternative syntax. The JSON format may be accessible via a typed rel=alternate link.

Content SHOULD be described using the [ActivityStreams](#) vocabulary, but MAY use other vocabularies in addition or instead.

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

### Objects

All objects must have URLs which return the properties of an object in an [appropriate format](#content-representation).

### Streams

Each stream MUST have a URL which MUST result in the contents of the stream (according to the requesters right to access, and could be paged), and MAY include additional metadata about the stream (such as title, description).

Each [object](#objects) in a stream MAY contain *only* its URL, which can be dereferenced to retrieve all properties of an object.

One user may publish one or more streams of content. Streams may be generated automatically or manually, and might be segregated by post type, topic, audience, or any arbitrary criteria decided by the curator of the stream. A user [profile](#profiles) MAY include links to multiple streams, which a consumer could follow to read or subscribe to. Eg.

`<link rel="feed" href="http://rhiaro.co.uk/tag/socialwg">`

```HTTP/1.1 200 OK .... Link: <http://rhiaro.co.uk/tag/socialwg>; rel="feed"```

<div class="issue">
  <div>Need examples</div>
</div>

## Subscribing

An agent (client or server) may *ask* to be notified of changes to a content object (eg. edits, new replies) or stream of content (eg. objects added or removed from the stream).

<div class="issue">
  <div class="issue-title"><span>Issue</span></div>
  <div>Lots of ways of doing this, what to use?</div>
</div>

Here are some options...

* **Web Push Protocol**: The subscriber follows the `urn:ietf:params:push` link relation to the target's Push Service, and then [Subscribes for Push Messages](https://tools.ietf.org/html/draft-ietf-webpush-protocol-02#section-4)
* **ActivityPump**: The subscriber posts a `Follow` Activity (JSON object) to the target's `inbox` endpoint, and adds the target to the subscriber's `Following` Collection. The target's server adds the subscriber to the target's `Followers` Collection, and subsequently `POST`s all new activities of the target to the subscriber's `inbox` endpoint. (*See [ActivityPump](http://w3c-social.github.io/activitypump/) 7.4.2, 8 and 9.2.4*)
* **SoLiD**: The subscriber sends the keyword `sub` followed by an empty space and then the URI of the resource, to the target's websockets URI. The target's server sends a websockets message containing the keyword `pub`, followed by an empty space and the URI of the resource that has changed, whenever there is a change. (*See [SoLiD - Live Updates](https://github.com/solid/solid-spec#live-updates)*)
* **PubSubHubbub**: The subscriber discovers the target's hub, and sends a form-encoded `POST` request containing values for `hub.mode` ("subscribe"), `hub.topic` and `hub.callback`. When the target posts new content, the target's server sends a form-encoded `POST` to the hub with values for `hub.mode` ("publish") and `hub.url` and the hub checks the URL for new content and `POST`s updates to the subscriber's callback URL. (*See [PuSH 0.4](http://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html) and [How To Publish And Consume PuSH](http://indiewebcamp.com/How_to_publish_and_consume_PubSubHubbub)*)
* **Salmentions**: The subscriber creates content that links to the target (eg. a reply) and sends a form-encoded `POST` containing values for `source` and `target` to the target's webmention [webmention](https://indiewebcamp.com/webmention) endpoint. The target verifies the link and includes a link back to the subscriber's source on the target content. The target sends form-encoded `POST` requests containing values for `source` and `target` to the webmention endpoint of every link in the content, including that of the subscriber, to indicate that there has been a change. (*See [webmention](https://indiewebcamp.com/webmention) and [salmentions](https://indiewebcamp.com/salmentions)*)

A server may also receive notifications of changes to content it has *not subscribed* to: see [mentioning](#mentioning).

## Mentioning

A user may wish to push a notification to another user, for example because they have linked to (replied, liked, bookmarked, reposted, ...) their content or linked to (tagged, addressed) the user directly.

* **ActivityPump:** When an Activity is posted to a user's `outbox` endpoint, the server checks for values of `object`, `target`, `inReplyTo`, `to`, `cc`, and `bcc`; discovers the `inbox` endpoint of any objects found, and `POST`s the Activity to the discovered `inbox` endpoints. Servers receiving such an Activity proceed to do the same for the target object to propagate the update further. *(See [ActivityPump 8.2](http://w3c-social.github.io/activitypump/#notification))*
* **Webmention:** The target publishes a link to their 'webmention endpoint' via `rel="webmention"`. The source sends a form-encoded `POST` request containing values for `source` (the URL of a webpage with a link to the target) and `target` (the URL of the webpage being linked to). The target MUST validate that the source really does link to target, and proceeds to do with this information as desired. *(See [webmention](https://indiewebcamp.com/webmention))*

*Note: we need to leave it open for users to refuse content they have not explicitly subscribed to, ie. nothing else should rely on implementation of Mentioning.*

## Creating content

`POST` a JSON object (see [content representation](#content-representation)) to the appropriate endpoint.

* **ActivityPump**: `POST` to `"outbox": "..."` (*See [ActivityPump 7.4.1](http://w3c-social.github.io/activitypump/#outbox)*)
* **Micropub**: `POST` to `rel="micropub"` (*See [Micropub](https://indiewebcamp.com/micropub)*)
* **SoLiD**: `POST` to an LDP container (*See [SoLiD - Creating new resources](https://github.com/solid/solid-spec#creating-new-resources)*)

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
