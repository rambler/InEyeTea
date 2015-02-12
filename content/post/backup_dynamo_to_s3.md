+++
date = "2015-02-06T17:36:03-06:00"
title = "Backing up DynamoDB to S3"

+++

Sounds easy! <!--more-->
## Things I tried

1. The obvious choice is to use the AWS tools to do this. However, I found that the tool for this, [AWS Data Pipeline](http://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-importexport-ddb-part2.html), is a bit too heavy for our needs. Basically the approach there is to partition the data and launch a dynamic Hadoop cluster to manage the transfer. This isn't consistent with the scale of our application (small), so this method didn't work. Specifically, it takes money and time to launch instances and we're perfectly capable of doing this from an instance we already have launched.

2. I got lucky and the second thing I tried worked. Thanks, internets!

3. These tools might be worth checking out:

* [dynamo-archive](https://github.com/yegor256/dynamo-archive)
* [dynamo-dump](https://github.com/bchew/dynamodump)

## What worked

1. I was able to get [dynamo-backup-to-s3](https://github.com/markitx/dynamo-backup-to-s3) to work. The result of this tool is a bunch of json files in S3. How do I know it worked? I looked at the files in S3.

## What's wrong with this

While this tool appears to do the job when configured correctly, it doesn't report when there are problems. I ran the tool and it happily reported success when I:

* Specified a "read percentage" that was greater than 100% (I was specifying 10% as integer 10 rather than 0.1).
* Specified a bucket to write to that wasn't writiable with the supplied credentials.

In both of these cases, nothing was written to the output - apparent success and actual fail on a tool for backups is a user interface problem.

Actually, after a bit more digging, I found out that there are some more substantial bugs in the code. The good news is, there's not much code, so it's pretty easy to track them down. This is a good opportunity to get some better error handling into this tool as well.

## Working with the developer

I identified a specific bug and created [a bug](https://github.com/markitx/dynamo-backup-to-s3/issues/3) for it, then fixed it and created [a pull request](https://github.com/markitx/dynamo-backup-to-s3/pull/4). The developer accepted the pull request and pushed to NPM right away. Cool!

It turned out that there was another bug, so I repeated the process. Now it seems to be working as expected.
