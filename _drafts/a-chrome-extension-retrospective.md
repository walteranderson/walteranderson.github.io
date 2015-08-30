---
layout: post
title: A Chrome Extension Retrospective
---

Many months ago, I experienced what many would call "scratching an itch". As an avid
[Trello](www.trello.com) user, I realized how many steps were involved in creating a card
based on a website. So I made a chrome extension for it!

This is something I would do quite frequently:

- decide I want to save whatever article or github project I was looking at
- open a new tab
- go to Trello
- go to the board I keep any interesting resources
- add a new card
- click back to the article and grab the title and url
- go back to Trello
- type out the title of the card
- click on it
- click on the description
- paste the url.

That's just too many steps for me. After looking into the Trello API, I discovered this was
some pretty low-hanging fruit. So the next weekend was spent hacking away at a chrome extension that
gave you a form with some fields auto populated with the page title and url, along with your Trello
boards and lists. It turned out to be a fairly simple, yet incredibly satisfying project. That said,
I wanted to go back through and talk through some of the decision I made and how they went.


## Module Javascript

I was already pretty sold on the benefits of creating modules for related functionality, but
this pattern really helped me consolidate some critical application logic. I created two modules:
"storage" and "api". As you might expect, storage is a wrapper around localStorage to cache the data
I am fetching from Trello as well as the user's configuration options and default settings. The api
module contained any API interactions with Trello like authorization and the few API calls I need.

The storage module was really useful because localStorage can be a little cumbersome to deal with,
especially since I wanted to store the data as JSON strings. So not only was this module responsible
for getting and setting from localStorage, but also encoding and decoding the data. I did run into one
issue with this module that I found out through a bug request. You can view my
[commit that fixes this](https://github.com/walteranderson/add-to-trello/commit/0e77dfd479247e545a491c0bac4f6ec16044b81c).

To give some context, there is a method called resetDefaults. I wanted to be able to assign a default
Board and List if the user had not selected something yet. So I simply assigned the board and list
to the first element in the array because I just want something selected the first time a user opens
extension.

{% highlight javascript %}
var resetDefaults = function() {
  var orgs  = getOrgs();
  var board = orgs.me.boards[0];
  var list  = board.lists[0];
  ...
};
{% endhighlight %}

Bad idea. If the user didn't have any boards, a syntax error would occur later when trying to reference
board.id. Defensive programming is your friend in these cases. Never trust data that you do not explicitly
provide.

But this bug highlights a good question. Why is the storage module responsible for setting default
settings? Ideally it should be responsible for getting and setting the data, not logic around
what kind of data to store.

Another thought about modules, returning an explicit public API separate from the defined methods
is overly complex for an application this size. As an example, here's the object I return for the api
module.

{% highlight javascript %}
return {
    ready: ready,
    isAuthorized: isAuthorized,
    authorize: authorize,
    deauthorize: deauthorize,
    getOrgsAndBoards: getOrgsAndBoards,
    getBoards: getBoards,
    getOrgs: getOrgs,
    submitCard: submitCard
}
{% endhighlight %}

All of those methods have the same name! I ran into the issue many many times where I would change
the name of the method to better reflect the functionality and forget to update the return object. Then
spend 15 minutes debugging trying to figure out why I was getting undefined errors. For something this
size, a more reasonable approach would be to create a named object at the top, assign behaviors to that
object, and then return it.

{% highlight javascript %}
var module = (function() {
  var obj = {};

  obj.someMethod = function() {
    return 'some fancy logic';
  };

  ...

  return obj;
}());
{% endhighlight %}

I feel like this is a much more readable pattern for those that are unfamiliar with the code, and
you're not concerned with changing the module's API.


## Application Structure

The file structure I chose is pretty ad-hoc, which is fine for developing. I did most of the coding in
one weekend, after all. But that can make things a little confusing for people new to the code.
Since I'm just using jQuery, I think a reasonable separation would be 1) DOM events and 2) business logic.
Business logic, in this case, is just interacting with the Trello API, localStorage, and some data transformation.

Right now there is a file for each page (the popup and the settings page). Inside each file, there's a
mish-mash of bootstrapping for each page, some helper functions specific to that page, and the DOM events.
Those should really only contain the events that react to DOM changes and call any bootstrapping functions
that are implemented elsewhere.

Also, each module that I've defined would really be better in a separate file. Right now they're just
appended to the end of the functions.js file. This was only for convenience, since that file was already
created and I was too lazy to add the import statement to each html page. It would go a long way in
clearing up the code to put those in their own files though.

## Functional Programming
http://spf13.com/presentation/7-common-mistakes-in-go-2015/
- I unknowningly used some functional programming concepts
  - abstracting out createOptionGroup
  - changeList does not follow the paradaigm though
