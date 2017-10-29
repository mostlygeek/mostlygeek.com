---
date: "2017-10-28"
title: "Automating mostlygeek.com"
description: "Let robots work for you. How to automate a `git push` into published website. "
---

Automating things is really complex.  Automating an entire publising pipeline triggered by a github push is really, really complex. However, once set up it's reliable, nearly zero maintenance and very cheap to run. As I was writing this post it started growing out of control in the AWS and CircleCI sections. I've focused on writing about things that took the longest to figure out.

I had several goals for mostlygeek.com. Most of them let me focus on writing instead of publishing and maintenance.

1. A completely static website. No security updates or database to maintain.
1. A simple way to write and manage posts.
1. Automatic uploading of files to the web host.
1. Hosting I don't have to manage.
1. __CHEAP!__

These are the tools I chose:

- Hugo+Github for managing the content
- CircleCI to build and publish the website
- AWS for hosting the website


## Hugo + Github

__Why Hugo?__ I chose Hugo since it's written in Go. Which I'm a big fan of.  It's open source, mature and fast. The best part is that it's fairly opinionated in how content is organized.  It does takes some effort to learn how use and configure it.  Fortunately there is lots of  documentation to get you started.

__Why Github?__ Personal preference. Also the integration with CircleCI, which I use to build and publish the website.

## CircleCI

I came to prefer CircleCI's approach to automation and Github integration while working on the [Dockerflow](https://github.com/mozilla-services/dockerflow) project at work.  It's also free for open source (non private repo) projects.  When I push a new content to Github it automatically triggers CircleCI to build and publish the website.

When a CircleCI job starts it looks for a [circle.yml](https://github.com/mostlygeek/mostlygeek.com/blob/master/circle.yml) in your repo to tell it what to do. Getting this configured can be quite a chore.  You can use mostlygeek.com's as a starting point:

{{< highlight yaml>}}
{{<includeFile "circle.yml">}}
{{< /highlight >}}

There's some hidden magic you don't see in the source above.  There are some secrets like AWS credentials that are set up through CircleCI's web interface. These are:

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `CLOUDFRONT_DISTRIBUTION_ID`
* `PUSHOVER_APP_TOKEN`
* `PUSHOVER_USER_KEY`

I skipped the pushover setup. I'm lazy and using Pushover to notify me when the site's deployed so I don't have to check it manually. 

Check out my [CircleCI deploy logs](https://circleci.com/gh/mostlygeek/mostlygeek.com/tree/master) to see the above configuration in action. You may notice some (many) trial and error attempts at fixing weird bugs.

## Hosting on AWS

I'm going to gloss over a lot of details and instead focus on tips to save you some time.  You can fill in the gaps with AWS documentation (even though they can sometimes be impenetrable). Sorry. :)

My goals for hosting:

1. Host files in S3
1. Put Cloudfront in front of S3 to make it fast and secure with a TLS certificate
1. A new AWS IAM user with access limited to only the S3 bucket and Cloudfront
1. R53 to host the DNS

### Create an S3 bucket for serving the site

1. Create a new bucket, I named mine `mostlygeek.com`. Using a website's DNS will let you point a DNS record directly at the bucket. I'm not doing that, opting instead to use Cloudfront to serve the site super fast.
2. Enable static website hosting.  Note, you're going to need the S3 hosting endpoint later for setting up Cloudfront.
3. Add a bucket policy.  Use this to save you much painful documentation reading. Be sure to replace `<BUCKET NAME>` with the name of the bucket you just created.

{{<highlight json>}}
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<BUCKET NAME>/*"
        }
    ]
}
{{</highlight>}}

### Create an IAM user

In AWS IAM:

1. Create a new user, with a username like `blog_uploader`.
2. Create a set of security credentials and copy/paste them into the CircleCI environment as `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. The `s3deploy` tool in `circle.yml` will automatically use these.
3. Add a permission policy for `blog_uploader` so it can upload to S3 and update cloudfront. Just copy/paste mine, replacing `<BUCKET NAME>` with your S3 bucket name.

__Copy/Tweak/Paste me:__

{{<highlight json>}}
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": "arn:aws:s3:::<BUCKET NAME>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:DeleteObjectVersion",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:GetObjectVersion",
                "s3:GetObjectVersionAcl",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:PutObjectAclVersion"
            ],
            "Resource": "arn:aws:s3:::<BUCKET NAME>/*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "cloudfront:CreateInvalidation",
            "Resource": "*"
        }
    ]
}
{{< /highlight>}}


### Create a CloudFront Distribution

1. Create a new Cloudfront distribution in the AWS console.
1. Set an Origin to your S3 bucket. __There's a trick here.__ Since we enabled Static Website Hosting on the bucket you can just use the hosting DNS name. You can get it from the bucket's Properties > Static website hosting configuration. For example, this blog's is: `mostlygeek.com.s3-website-us-east-1.amazonaws.com`.  I find this easier than using the pure S3 route.
1. Copy/paste the distribution id into a new CircleCI environment variable as `CLOUDFRONT_DISTRIBUTION_ID`.  This will allow CircleCI to flush Cloudfront's cache when deploying the site.
1. If you want an HTTPS certificate you can request a free one from AWS ACM. There's a big button and a bunch of hoops to jump through but it should be pretty straightforward.

### Pointing your DNS at your Cloudfront Distribution

I use R53 for DNS hosting. A really nice feature is being able to create an R53 alias to Cloudfront. This way when your computer resolves mostlygeek.com, it will come back as IP addresses for the closest Cloudfront servers instead of a CNAME.
