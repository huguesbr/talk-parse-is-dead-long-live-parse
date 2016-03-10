# Parse is dead, long live Parse!

Talk's slide available here: https://github.com/huguesbr/talk-parse-is-dead-long-live-parse/releases/download/v1.0/slides.pdf

Dropbox's paper version: https://paper.dropbox.com/doc/Parse-is-dead-long-live-Parse-aDQc136PaA0Sy9Cd5Tf1l

This document is the draft use to build the talk's slide.

# Quick Parse Intro

It’s the M of MVC. M on steroids.

## History
- Release in 2011
- Bought by Facebook in April 2013
- Open Sourcing Client SDK in August 2015
- Open Sourcing Server (with some missing features)
- Shutting down 27 January 2017
- ~600k Apps using Parse (How many in prod…)
- Founders: Kevin Lacker, James Lu
## Pros
- Really nice API
- Abstract the full server stack
- Magic
## Cons
- Magic
- Scalability
- Expensive beyond free version (> 30 requests / sec)
- Unknown miscellaneous bugs
- Safe Harbor issue (hosting France personal data in the USA)
- Limitation
  - SQL type operation (count, …)
  - Max Time for functions
  - 10k objects max (and skip max 10k)
## iOS Client
- Great even for local storage only (great ORM)
- Parse UI
  - Great for prototype, stay away for more, really hard to customize more than basics (mixing logic UI and design UI)
## How does it work
- Built on top of Bolt
- Local storage
  - dirtyness and sync
- PFQuery, PFModel, PFSubclassing
- PFRole
  - Like a role management of a CMS for REST API
- PFFacebookUtils
  - Facebook connection
- Auto login Magic
  - Pro & Cons
- PFConfig
  - Useful

# Moving on!

http://savvyapps.com/blog/parse-server-missing-features-solutions-migration

**This document cover migration from Parse to Parse Server from an iOS App point of view, not a website.**

## Why it’s good news?
- Pros
  - Bypass all Parse limit (count query, 1000 rows, 15mn background, …)
    - http://profi.co/all-the-limits-of-parse/
  - New business opportunity (Parse Hosting, Parse extensions, …)
  - Active community (check out the parse-server github)
- Cons
  - Farewest style…
  - Not as click and go


https://d2mxuefqeaa7sj.cloudfront.net/s_E85944108094AE6CE2F541F64B50AED8628CB35F29860EFE7252FB2993F0CC11_1457345789013_comparison3.png

# Solutions

Parse Hosting vs DIY vs DIHY (Do It Hard Yourself)
https://github.com/ParsePlatform/parse-server/wiki/Parse-Server-Guide
[ParsePlatform/parse-server#38](https://github.com/ParsePlatform/parse-server/issues/38)

## Parse Alternative

Not purpose of this talk.
Any non open source alternative might end up with the same issue than Parse but with less export/alternative solutions.


## Parse 3rd party hosting

### Node Chef

https://nodechef.com/
https://nodechef.com/parse-server
https://www.nodechef.com/blog/post/6/migrate-from-parse-to-nodechef’s-managed-parse-server
Pricing: starting at $9/mo
Pros

- Simple
- Parse-like experience

Cons

- Pricing
- Eventual limitation
- Eventual region restriction

## Parse Self Hosting - DIY

Using all the open source package (see below)

### Pros
- Fairly Simple
- Cheaper than Parse Hosting
- Customizable

### Cons
- More expensive than AWS

### Provider

- Heroku
  - http://blog.parse.com/announcements/hosting-your-own-parse-on-aws-and-heroku/
  - https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku
- …


## Parse Self Hosting - DIHY

Parse Server provide a docker config.

### Pros
- Cheap

### Cons
- Not straighforward

### Provider
- AWS
- Digital Ocean
- Instainer (http://beta.instainer.com/?deploy=instainer/parse-server)

https://aws.amazon.com/blogs/aws/resources-for-migrating-parse-applications-to-aws/

# Database

We highly suggest migrating your database by **4/28/2016.**
Super easy using parse migration tools:

- Create mongo instance (self hosted, heroku add:ons, mongoLab, …)
- Copy MongoDB URI (`?ssl=true`)
- Use Parse Migration Tools (be careful if live) on Parse Dashboard

http://blog.parse.com/announcements/parse-migration-guides-discounts-and-events/
http://blog.parse.com/announcements/introducing-parse-server-and-the-database-migration-tool/

## Notes
- Use US-East Hosting (Virginia) to match Parse Hosting.
- 3-4 times the data size currently used by your Parse application. Available in 'Analytics' > 'Overview'.
- If storing files, don't forget to set 'failIndexKeyTooLong'.
- The open-source Parse Server software does not support auto-indexing. You will be responsible for managing your indexes, which is crucial for maintaining optimal database performance. When you migrate your database to mLab, all existing indexes will be migrated.

## mLab Offer (MongoLab)

http://docs.mlab.com/migrating-from-parse/
$0 for prototyping/testing
Create a new mongodb instance, use Parse DB migration tools using the mongo URI.
Nice integration with Heroku.
Enjoy the “secret” table (heroku link).

## ObjectRocket

Not tested

## Cloning && Backup

```
  brew install mongodb
  mongodump -h mongodbserver.com:port -d databasename -u username -p password
  mongorestore -h mongodbserver.com:port -d databasename -u username -p password
```

# Parse Server

https://github.com/ParsePlatform/parse-server-example
https://github.com/ParsePlatform/parse-server#getting-started
https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse

- Automatic creation of classes by the client is always allowed in Parse Server.
- Class Level Permissions supported (edit directly the SCHEMA’s collection)
- In-App Purchases not supported
- Welcome Emails and Email Verification not supported but can be implemented via beforeSave.

This represent the core of Parse (REST API) only, login/signup.
Pretty easy.

- Clone the repo (via heroku deploy)
- Setup your config
- Migrate your database or point it to an existing mongoDB instance

```
# assuming node is up to date
# assuming you have already setup your mongodb
# fork [https://github.com/ParsePlatform/parse-server-example](https://github.com/ParsePlatform/parse-server-example)
git clone https://github.com/xxx/parse-server-example
cd parse-server


# local
npm install
# drink couple of coffee, or better, beers!
npm start
export DATABASE_URI="mongodb://..." APP_ID="xxx" MASTER_KEY="xxx"
curl -X POST \
   -H "X-Parse-Application-Id: xxx" \
   -H "Content-Type: application/json" \
   -d '{}' \
   https://localhost:1337/parse/functions/hello

# remote
vi app.json
# update keys, url, db uri.
heroku apps:create
heroko config:set APP_ID="xxx" MASTER_KEY="xxx" SERVER_URL="https://yourapp.herokuapp.com" DATABASE_URI="mongodb://user:password@domain.com:19498/database"
# optional CLIENT_KEY="xxx" 
git push heroku master
curl -X POST \
   -H "X-Parse-Application-Id: YOUR_APP_ID" \
   -H "Content-Type: application/json" \
   -d '{}' \
   https://yourapp.herokuapp.com/parse/functions/hello
```

## OAuth

Email signup/signin supported out of the box (no email validation?).
Easy to add any 3rd party oauth (Facebook, twitter, any oauth identification)

```
var api = new ParseServer({
  ...
  oauth: {
   facebook: {
     appIds: "xxx"
   }
  }
});
```

## Files

Parse server support storing file in MongoDB by default, but recommend to use S3 adapter larger file.
Also, you will need to migrate your file hosted on Parse S3 instance before service shutdown.
http://blog.parse.com/announcements/hosting-files-on-parse-server/

## Config (PFConfig)

Trivial to re-implement on node express

```
const fs = require('fs');
...
app.get('/parse/config', function(req, res) {
  var configFilePath = __dirname + '/config.json';
  var readable = fs.createReadStream(configFilePath);
  readable.pipe(res);
});
```

## Background Jobs

Well it’s a node express server, so…


# Dashboard
## Parse Dashboard

https://github.com/ParsePlatform/parse-dashboard
Super easy, just another instance using the Parse Server API.
Could even be hosted locally.


```
git clone git@github.com:ParsePlatform/parse-dashboard.git
cd parse-dashboard
npm install
vi Parse-Dashboard/parse-dashboard-config.json

{
  "apps": [
    {
      "serverURL": "https://api.parse.com/1",
      "appId": "xxx",
      "masterKey": "xxx",
      "appName": "name",
      "javascriptKey": "xxx",
      "restKey": "xxx"
    },
    {
      "serverURL": "http://myserver.herokuapp.com/parse",
      "appId": "xxx",
      "masterKey": "xxx",
      "appName": "name"
    }
  ],
  "users": [
    {
      "user":"xxx",
      "pass":"xxx"
    },
    {
      "user":"hugues",
      "pass":"xxx"
    }
  ]
}


# local dashboard
npm run dashboard
open "http://localhost:4040"


# remote
# don't forget users if remote dashboard
heroku apps:create
git push heroku master
open "http://yourapp.herokuapp.com"
```

### Pros
- The one we know
- Customizable


## Adminca
[Not tested]
http://adminca.com

### Pros
- More feature like a CMS (Collaborators, roles, validation, …)

### Cons
- Pricing?

# Push Notifications

https://github.com/ParsePlatform/parse-server/wiki/Push#push-adapter

## Parse Push Notifications

Build-in in Parse Server and connect directly to APNS
http://blog.parse.com/announcements/parse-server-push-notifications/

### Pros
- Similar to Parse 

### Cons
- Delivery or re-delivery handling?
- Error handling
- Scheduling

```
var api = new ParseServer({
  ...
  push: {
    ios: {
      pfx: 'aps_production.p12', // filename of private key and certificate in p12
      bundleId: 'com.wired-beauty.WiredResearch', // bundle identifier 
      production: true // production/true, development/false
    }
  }
});


curl -X POST \
    -H "X-Parse-Application-Id: xxx" \
    -H "X-Parse-Master-Key: xxx" \
    -H "Content-Type: application/json" \
    -d '{
          "where": {},
          "data": {
            "title": "Wired Research",
            "alert": "Message"
          }
        }'\ 
    http://yourserver.com/parse/push
```


## 3rd party

- AWS
  - https://mobile.awsblog.com/post/Tx3NE69QDHI7LJK/Migrating-from-Parse-Push-to-Amazon-SNS
- One Signal
  - https://onesignal.com/parse
- Batch
- Urban Airship


# Metrics
- MixPanel
- Keen.IO
- Flurry

# Emailing
- Sendgrid
- Mandrill
- Mailgun, …

# iOS Side

Really simple.


```
[Parse initializeWithConfiguration:[ParseClientConfiguration configurationWithBlock:^(id<ParseMutableClientConfiguration> configuration) {
    configuration.applicationId = applicationId;
    configuration.clientKey = clientId;
    configuration.server = url; // https://api.parse.com/1/ vs https://yourserver.com/parse/
    // important if using local store - instead of enableLocalDatastore
    configuration.localDatastoreEnabled = YES;
}]];
```

# Recommended Step
1. Migrate your DB now
2. Try Parse Server to see if it fit your need
3. Create a new Parse Server connected to your existing migrated DB
4. Create a new Parse Dashboard connected to your existing migrated DB
5. Start using S3 for files
6. Migrate to Parse Server

# Parse Alternative

http://www.shephertz.com/parse-migration-app42.php
https://pushbots.com/parse

# Resources

https://github.com/ParsePlatform/parse-server/wiki
https://parseplatform.github.io
https://parse.com/faq
http://profi.co/all-the-limits-of-parse/
http://savvyapps.com/blog/parse-server-missing-features-solutions-migration
https://github.com/ParsePlatform/parse-server-example
https://hub.docker.com/r/instainer/parse-server/
http://savvyapps.com/blog/parse-server-missing-features-solutions-migration
http://blog.parse.com/announcements/introducing-parse-server-and-the-database-migration-tool/
[ParsePlatform/parse-server#38](https://github.com/ParsePlatform/parse-server/issues/38)
https://www.quora.com/Is-it-easy-to-migrate-from-Parse-to-my-own-backend-without-losing-DB
https://www.mongodb.com/migrate-from-parse-to-mongodb-cloud-manager-and-aws
http://blog.parse.com/announcements/parse-server-push-notifications/
http://docs.mlab.com/migrating-from-parse/
http://objectrocket.com/parse
http://adminca.com/faq
https://batch.com/parse-alternative
http://venturebeat.com/2016/02/06/how-to-avoid-the-next-parse-shutdown-scenario/
http://techcrunch.com/2016/01/28/facebook-shutters-its-parse-developer-platform/
https://www.webniraj.com/2016/01/31/parse-com-setting-up-the-open-source-parse-api-server/
https://www.digitalocean.com/community/tutorials/how-to-run-parse-server-on-ubuntu-14-04
https://www.webniraj.com/2016/01/31/parse-com-setting-up-the-open-source-parse-api-server/
https://github.com/relatedcode/ParseAlternatives
http://highscalability.com/blog/2016/2/2/the-big-list-of-alternatives-to-parse.html
http://julienrenaux.fr/2016/01/29/complete-parse-server-migration-guide/
https://www.digitalocean.com/community/tutorials/how-to-migrate-a-parse-app-to-parse-server-on-ubuntu-14-04
