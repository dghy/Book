![The ZODB Ecosystem by Christopher Lozinski](ZODBTitleGraphics.svg)

## Abstract

ZODB is an object-oriented database written in Python and
optimized in C. It is widely used in the US (United States of America),
but not as well known in Poland.
This talk introduces the database, presents important concepts, and reviews
the tools available.

## About the Author

Christopher Lozinski has been using the ZODB since 1999.
He is an MIT graduate and Silicon Valley expat who has
been using dynamically bound
languages for 35 years. First it was Lisp, then Objective-C,
and now Python. On the ZODB, he built PythonLinks.info, PolandTrade.info,
PrivaCV.com, Zopache.com, and the recently decommissioned
SpecialtyJobMarkets.com. He is a dual Polish-US citizen, and polyglot.

## Introduction

You can download the talk slides [1] and watch the presentation [2].
ZODB [3] is an object-oriented database written in Python
and optimized in C.

## So Easy to Use

ZODB is really easy to use from Python.
Just subclass off of class persistent.
Persistent, and your objects become persistent.
Here is a code sample from the Tutorial [4].

```
import persistent

class Account(persistent.Persistent):

    def __init__(self):
        self.balance = 0.0

    def deposit(self, amount):
        self.balance += amount

    def cash(self, amount):
        assert amount < self.balance
        self.balance -= amount
```

Did you notice how trivial it was to create a persistent object. Magic!

You can also subclass off of class Persistent Containers (Btrees), and
class Persistent Set.

## ACID Properties

ZODB is an ACID-compliant database [5]:

- Atomicity requires that each transaction be "all or nothing";
- Consistency ensures that any transaction will bring the database from one valid state to another;
- Isolation ensures that the concurrent execution of transactions results
in a system state that would be obtained if transactions were executed
sequentially, i.e., one after the other;
- Durability ensures that once a transaction has been committed, it will
remain so, even in the event of power loss, crashes, or errors.

## ZODB Data Model

Simple ZODB applications store data as a tree of objects,
essentially a tree of dictionaries. More sophisticated
applications store Data as a graph of objects. Really sophisticated
applications can store data as any Python data structure. If the data is
reachable from the root object it is persistent. If not, it is garbage
collected.

## A Specific Data Model

PythonLinks.info/the-world is a model of the Python software industry
represented as a hierarchy. "The-world" is a category containing regions.
Regions contain countries. Countries contain cities. Cities contain
companies. Companies can contain jobs, products, and other links such as
YouTube videos, product reviews, and blogs. The tree representing the
Python industry can be displayed as a table.

## Traversal
To access a particular object, one has to start at the root, and traverse
the tree. Since the nodes are essentially dictionaries, with text keys, it
is very natural to map a URL to a particular object by traversing the tree.


## Canonical URLs
ZODB applications access objects by traversing the tree. It would be nice
if every object had its own unique name, so that even when the tree
structure changed, it would still be easy
to find the object using its canonical URL.

PythonLinks provides canonical URLs. Every node has a unique name which can
be accessed using a dictionary in the root node.
Attempts to create a new node with the same name, result in an integer
being appended to the new node name: Node, Node-1, Node-2, etc.
When objects are renamed, or deleted, the root dictionary is updated.

## ZODB History and Undo
ZODB is a versioned database. Each object preserves its historical versions
until they are garbage collected. That makes
it possible to edit an object, save the edits, and then revert to the
previous version of the object. One can even do diffs of previous objects,
in order to decide which one is best.

## ZODB Storage Options

There is a rich variety of ways to store your ZODB objects:

1. **FileStorage** is the basic way of storing ZODB (Python) objects on the
file system. There is also Blob storage for storing images, files and other
large objects on the file system.
2. **ZEO** (Zope Enterprise Objects) is the basic client-server version of the ZODB.
ZEO clients share data from the ZEO server.
Each participating Python application
server includes a ZEO client. They all talk to the ZEO server. The ZEO server
will then store the data in FileStorage and BlobStorage
3. **RelStorage** stores ZODB data in a relational database. It performs
better than ZEO. It runs on top of MySQL, Oracle and PostgreSQL.
4. **NewtDB** is a version of RelStorage optimized for PostgreSQL. It stores
data as both Python pickles, as well as JSONB PostgreSQL data structures. This
allows one to benefit from the native PostgreSQL catalogs.


## Caching
Caching is an important feature of the ZODB. Every ZEO client includes an
object cache. The object cache is usually in RAM. There is an optional
file based cache. If objects are updated on one ZEO client, they are
invalidated on the other clients, and reloaded as needed. Generally this
works fine, but in cases where the object is actively used on all ZEO
clients, and frequently updated, it does lead to more network traffic.

## Scalability
For mostly-read applications ZODB scales brilliantly.
Multiple disk heads can read simultaneously from different parts of the file
used in FileStorage. Historically its
performance under heavy writing was limited because writes all happened at
the end of a single file. Only one disk head at a time could write. That
problem has now been fixed. Writes can now occur simultaneously on
multiple files on a single computer. The database can grow even larger by
hosting the blobs on remote servers, such as Amazon S3.

At Zope Corporation, several hundred newspaper content-management systems and
web sites were hosted using a multi-database configuration with most data
in a main database and a catalog database. The databases had several hundred
gigabytes of ordinary database records plus multiple terabytes of blob data.

Of course if you need multiple computers to store your object data (not blobs),
then you may want to upgrade to NEO (Not NEO4J). NEO is a GPL'd
cousin to the ZODB.

## Multiple Views on Every Object
The modern way to use a ZODB data model on the web is to support views on
objects. There can be multiple views on any object. There can be the same
view on different objects. That same view can be one class, or else using
the Zope Component Architecture, there can be different views with the
same name on different classes of objects, depending on the interfaces
which they have defined.

## ZODB-related Platforms

So which platform should you use with ZODB?
I am not an expert in all of these platforms, but I will give you my best
advice on when to use each of them.

### Plone
If you need a Content Management System, and Plone and its plugins do what
you need. Brilliant. Go for it. Plone is mature, stable, heavily used.
They have an annual international conference, and many regional conferences.
I recommend it highly. There are lots of consultants you can hire.

But Plone is not an application development platform. If Plone does what
you need, you are in great luck. But I advise against making changes to
Plone. Small changes you may be able to make, but anything more major,
and you
are more likely than not to run into trouble. There is just so much code
in there. Zope 2 code, Zope 3 code, and Five (a merger of
Zope 2 and Zope 3). Then there is the Content Management Framework (CMF), and
all of Plone. Way way too complex to even consider doing custom app
development on it.

### Pyramid
The next obvious choice is Pyramid [6]. If you are building a corporate
application, Pyramid is a great option. You can use the ZODB, or a
relational database. You can use routing or traversal. You can configure
it three different ways. You
can choose from lots of different GUI and other libraries.


### Grok
Me, I chose to use Grok [7] with Zope 3. I am not building a corporate
application. I am building startup applications. I don't care about
computer performance, if my applications are overloaded, that is a great
problem to have. I can then optimize things. Above all else,
I care about speed and cost of development. Related to that is software flexibility.


### Zopache

Sadly Grok lacks a TTW (Through the Web) development environment, so I built one.
Zopache.com is a JavaScript IDE
which I built on top of the ZODB using Grok. ZODB
developers might call it a TTW development platform.

Basic Zopache objects include HTML, CSS, JavaScript, Python Scripts,
Image, File and Jade.
HTML objects can be edited with either the WYSIWYG HTML editor, or the
technical Ace Editor. CSS JavaScript and Python Scripts can
also be edited with the Ace Editor.

Zopache also supports containers. Trees can be built out of containers.
Folders support cut, copy, rename, paste and delete operations. HTML folders
can be edited with the Ace [8] and CKEditor [9]. JavaScript Folders allow one to
build a well organized tree of JavaScript code, but serve it as a single
minified file or as a human readable and searchable file, which clickable links
to the original JavaScript object. History and Undo are supported.

Using Zopache, in your browser, you create a tree
of folders, and populate and edit those objects.
Every object has its own URL, and can be
displayed or edited. There is no need to know Unix. But you do have the
option to grow the application with real Python code.

### Flask
Flask [10] is a very lightweight framework which can be used with the ZODB.
Flask does routing, so one has to manually map from URLs to objects.

### Other Frameworks
There are many Python frameworks, and many other less often
used ways of working with the ZODB. I won't
mention them all here. They lead to market confusion. But
it is only natural in the Python world for there to be
many different solutions. So one really needs a survey of the
ZODB ecosystem to get you started in the right direction.


## Pyramid vs Grok
So let us compare them. Of course Grok can be used in different ways,
so I will focus on how I use it.

Pyramid
is very focused on computer performance, how fast will the software
run? I am focused on developer
performance, how fast can I develop my applications. They want close to the
metal code. I want high levels of abstractions.

In particular the Pyramid folks are mosty strongly against TTW development.
I invite you to take a look at the quote on the SubstanceD
demo page [11].
Clearly they are very conflicted about minimal TTW development,
let alone the idea of building a full JavaScript IDE on top of the ZODB.

The next issue is the target market. Pyramid is firmly focused on the
consulting market. Every client is different and needs a different solution.
They want the ability to
do it this way or that way. Use a relational database or the ZODB. Use
dispatch or traversal. Configure it three different ways. There are multiple
ways to do things.

I am more focused on the product market. There is no legacy code here.
Just the web, which I have to work with.
I want to build it as fast and flexibly as is possible.
I like the Python idea that there should be one
way to do things.
I want a purist OO approach.
Like Ruby on Rails, I want an
opinionated high level framework. Grok gives me that.

Pyramid is focused on multiple deployments. So TTW makes no sense.
I am focused on building a single website, so storing
stuff in the ZODB is much more modern than storing it on a 40 year old
filesystem.

But let us do a closer technical comparison.
Actually I am not even sure what to compare. For
a fair comparison, Pyramid does not stand on its own. It needs a bunch of
libraries to match what Grok does.

The first thing I want to do is automatic CRUD. Define the schema and
get CRUD for free. zope.formlib does this for me. z3c.form does this for me for
trees. Pyramid does not include CRUD.
One has to add it manually. I am still not
sure which CRUD package to recommend. Reportedly the one used in SubstanceD
does not do CRUD. So which one does? The next one I found has just been
re-built for use with relational databases. Also not what I need.

I also use components. Pyramid only supports view components.
But the Zope Component Architecture is now available with PyAMS.
That was not true when I started.

And of course there is no way that
somethng like the high level zope.securitypolicy has been ported to Pyramid.
It is just incompatible with the performance-optimized Pyramid security model.

## Other Databases
There are many other NoSQL databases on the market. My job is
not to evaluate them all, but just to get you to ask the right
questions:

1. Are they ACID compliant?
2. Do they provide a graph database?
3. Do they support Persistent Python?
4. Can you add arbitrary instance variables at run-time?
5. Do they support history and undo?

## About PythonLinks.info
PythonLinks.info is the world's largest Python Directory. One branch
includes a geographic directory of companies using Python. For each
region, for each country for each city, Python Links lists the companies
in that city, and provides important links for that company. Their name is
hyperlinked to their web site. Their products and jos are listed. It is also
possible to add YouTube videos, blogs, product reviews, and reviews of what
it is like to work at the company.

This directory is all in JavaScript, and JSON, so it is
quite easy to add teh table of companies in your city to your website.
Let me know if you are interested in doing so.

## References

1. https://pythonlinks.info/presentations/zodbtalk.pdf
2. https://PythonLinks.info/zodb
3. ZODB. http://www.zodb.org
4. ZODB Tutorial. http://www.zodb.org/en/latest/tutorial.html
5. ACID. https://en.wikipedia.org/wiki/ACID
6. Pyramid. https://trypyramid.com
7. Grok. http://grok.zope.org
8. Ace editor. https://ace.c9.io
9. CKEditor. https://ckeditor.com
10. Flask. http://flask.pocoo.org
11. SubstanceD. http://demo.substanced.net/
