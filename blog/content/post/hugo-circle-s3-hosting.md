+++
date = "2017-05-17T15:02:42-07:00"
draft = true
title = "hugo circle s3 hosting"

+++

Publishing from Hugo to S3+Cloudfront

* Hugo
	* (lots of great resources for this)
* AWS Setup
	* IAM Setup
		* creating a user for circleci to upload content
	* S3 Setup
		* creating an S3 bucket
		* setting up hosting
	* setting up CloudFront
	  * setting up the distribution
	  * setting up a CNAME / dns name
	  * setting up TLS
	  * setting up caching
* CircleCI setup
	* add shell script to pull down tools 
		* [https://github.com/bep/bego.io/blob/master/circle.yml](https://github.com/bep/bego.io/blob/master/circle.yml)
	* adding a circle.yml ([example](https://github.com/bep/bego.io/blob/master/circle.yml))
	* install [s3deploy](https://github.com/bep/s3deploy)
	* Publishing to S3
