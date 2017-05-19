---
date: "2017-05-17T15:02:42-07:00"
draft: true
title: "hugo circle s3 hosting"
---

# Publishing from Hugo to S3+Cloudfront

My first static website (90's). All you had to do was write HTML and FTP it somewhere. This is still another static website but now we have a lot of abstraction to make things easier. What we really want to get to is this idea workflow: 

1. Write a blog or an article 
2. git commit / push it to Github 
3. site is automatically published to the web 
  - fast! CDN backed
  - scalable, no hug of deaths
  - cheap. paying by traffic / requests from the CDN is expensive


How i have my blog publishing environment. This is meant to be a fairly technical set up blog post / documentation on how to get all the pieces wired up.

Perhaps I should make a shell script that automates a lot of these pieces? 
  - the main 3rd party service providers: AWS, CircleCI, GitHub

Complex and lots of pieces. This is really for highly technical people... though some of the parts can be made *faster* with a shell script. 

## Setting up Hugo
* A new Markdown editor (macdown)
	* detect front matter
* Hugo Installation
	* config.toml
		* `MetaDataFormat = "yaml"` - works better with macdown for rendering frontmatter

## GitHub
## Setting up AWS
* S3 Setup


### IAM User/Role
* about IAM, ... creating a user to write to the S3 bucket
* creating a user for circleci to upload content
* setting IAM inline policies to write to S3 bucket

### S3 Bucket
* creating an S3 bucket
* setting up hosting
* bucket name has to be the same as your DNS name (mostlygeek.com) Ref: AWS Blog [root hosting](https://aws.amazon.com/blogs/aws/root-domain-website-hosting-for-amazon-s3/), [S3 website](https://aws.amazon.com/blogs/aws/host-your-static-website-on-amazon-s3/)


### DNS Route53 

* hosting your DNS in R53 makes everything easier
* Set up an R53 alias to your S3 bucket. This will just return an IP address and does not require using CNAMEs

## Automatic Publishing
* add shell script to pull down tools 
	* [https://github.com/bep/bego.io/blob/master/circle.yml](https://github.com/bep/bego.io/blob/master/circle.yml)
* adding a circle.yml ([example](https://github.com/bep/bego.io/blob/master/circle.yml))
	* ... should i merge in the ci-install-hugo.sh into circle.yml? makes it easier to copy/paste
* install [s3deploy](https://github.com/bep/s3deploy)
* Publishing to S3
* Using that IAM user (env variables)
* set AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY env variables in circle's configuration


## Making it fast and secure

* setting up the distribution
* setting up a CNAME / dns name
* setting up TLS
* setting up caching settings
	- cache things for a long time 
	- invalidate everything (?) on deployment ([example](https://github.com/rdegges/ipify-www/blob/master/.travis.yml#L11))
	- sparse invalidation would be better... 
	- clouddfront IAM permission must be done on `*` ([SO Source](http://stackoverflow.com/questions/29558655/restrict-access-to-a-particular-cloudfront-distribution-using-iam))
* Setting up TLS like it's 2017
  - not entirely necessary. HTTP is fine for serving a static website
      - however, HTTPS gives more guarantees that content wasn't changed by somebody in the middle

I