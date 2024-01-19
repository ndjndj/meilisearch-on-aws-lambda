
# Meilisearch on Amanzon Lambda

[English](./README.md) | [日本語](./README_ja.md)

**I wrote this text while using a translation tool and apologize if there are any mistakes or if the wording is impolite.**

Building a serverless full-text search environment for Meilisearch on AWS.

## Introduce 

- Meilisearch environment was able to build and run on AWS Lambda using the Lambda Web Adapter.
- Amazon EFS was used to store the indexes.
- Amazon EFS DataReadIOBytes was high average.。

→ seems impractical.

## Introduction 

Meilisearch is an open source full-text search engine developed by Meili and a lot of contributors.  
It has a very fast millisecond search performance, a simple API, and a rich feature set that has gained a lot of support.  
I personally like the fact that it has an active community and is continuously updated (and the icon is kawaii).  

Although I succeeded in building Meilisearch environment what using AWS Lambda as one of FaaS and Lambda Web Adapter, It may be impractical in a using of production environment.

However, I have learned quite a bit myself, so I am publishing this article in the faint hope that maybe there will be an opportunity.   
Any ideas, opinions, or advice would be greatly appreciated.

## Notice  

Note that the use of the following AWS services incurs usage fees.  
The fee to AWS varies depending on the size of the Meilisearch index to be created, the frequency of index creation, and the number of times the search is performed.

For detailed pricing information, please see the appropriate page or the [AWS Pricing Calculator](https://calculator.aws/#/).

- [Amazon EFS](https://aws.amazon.com/jp/efs/pricing/)
    - As a storage for Meilisearch indexes.
- [Amazon EC2](https://aws.amazon.com/jp/ec2/pricing/) (option)
    - Will be used to transfer Meilisearch indexes to Amazon EFS.
- [AWS DataSync](https://aws.amazon.com/jp/datasync/pricing/) (option)
    - Will be used to transfer Meilisearch indexes to Amazon EFS.

## Why use serverless? 

The options for implementing a full-text search engine are to use a cloud service such as Meilisearch Cloud, Algolia, or to self-host a full-text search engine.
Either way, you will need to continue paying some running costs.  

Not every product has a generous budget allocated to it, and developers are looking for low-cost ways to ensure ongoing development and operation.  
Especially, individuals products or small products.  

An idea was that we might be able to fulfill the need for a full-text search engine at the lowest possible cost,  
I tried to build Meilisearch in a serverless environment.  

FaaS such as AWS Lambda provide to many free tier. 
I also hypothesized that Meilisearch latency would be enough for practical use even after taking into account the delay caused by cold start[^1],  
and I thought that be able to meet the need for a full-text search engine at the lowest possible cost (and also because it sounds interesting this challenge).

reference: [Serverless search with Meilisearch and Google Cloud Run](https://blog.simonireilly.com/posts/serverless-search)

## Usage 

The procedure for creating a serverless environment can be broken down into several steps as follows.
The data to be searched must be created locally (in a non-Lambda environment).

1. Create a [Dockerfile](. /prod/Dockerfile) of the Meilisearch environment.  
2. A created [Dockerfile](. /prod/Dockerfile) in step 1, apply Lambda Web Adapter and build image.
    - Set up the environment settings.
    - PORT must be set to 8080
3. Push the Dockerimage created in step 2 to Amazon ECR. 
4. Deploy the Dockerimage of step 3 to AWS Lambda.
    - Lambda memory is 128 MB OK (may depend on index size) 
5. Mount Amazon EFS on AWS Lambda. 
6. Import data.ms to the EFS access point mounted in step 4.
    - Any import method is acceptable (DataSync, EC2, etc.) 
7. Set the function URL or attach to API Gateway.

## Utility 

The point for discussion of practicality is the handling of data.ms used for persistent storage.  

As an assumption, Meilisearch seems to update files directly under data.ms each time[^2],   
in Lambda, a 'read-only file system error' occurs if you try to operate on a file placed outside of tmp[^3].  
Therefore, data.ms/, where updates occur, cannot be placed anywhere other than directly under tmp.  

On the other hand, if data.ms is placed directly under tmp, AWS Lambda will not be able to perform a good search because the file directly under tmp will disappear after some time[^4].

With the above background, decided to use Amazon EFS.  
In other words, I wanted to import data.ms into EFS and mount it on Lambda in order to keep data.ms persistent.  
However, using this method would have meant loading the data.ms each time, which would have resulted in a very large amount of EFS loading.  

The data we used in this case is about 120,000 documents with an index size of about 140 MiB. In this case, nearly 100 MB of reads were occurring each time.  
Checking the EFS logs, it seems that reads are being performed at each cold start.  

Also, it is not practical to use the write API via Meilisearch-Lambda, so it is necessary to update data.ms/ in EFS by a method other than Lambda[^5].  

So, although succeeded in creating a Meilisearch environment with AWS Lambda, However, the cost increase was unavoidable due to the large amount of EFS reads.

Initial goal was to reduce the cost of running Meilisearch, but it would likely be cheaper to host it on EC2 if it were to be used in its current state.  

It may be possible if the site is individuals operated and accessed at a consistent time, or if the index size is not so large anymore.  

## Acknowledgment  

Without Meilisearch's high search performance, I would never have considered building serverless. 
Thanks to all the contributors to Meilisearch that give a great product and the opportunity to take on this challenge!

## PS 
This time, we tried to increase the options for implementing a low-cost full-text search engine, but the results were indescribable.  

However, there is a difference between trying and finding out that it is no good and being told it is no good from the start.  
I was able to experience the performance of Meilisearch and learned the usefulness of the Lambda Web Adapter, which will eventually become part of my blood.  
There is always room for improvement😊

Comments and improvements are welcome!  
[Issue](https://github.com/ndjndj/meilisearch-on-aws-lambda) or comment.

[^1]: Not concerned about the fact that can't write documents. Meilisearch specification is to stack write tasks in a queue and then process them in order, so I think it would be difficult to do it serverless in the first place.
[^2]: It's probably a lock system file, but I haven't been able to catch up on the timing of the update. It is based on the date the file was updated.  
[^3]: This is AWS Lambda's specification, directories other than /tmp are not authorized to write.  
[^4]: Occured Index not found error. There may be some room for verification. Is the data still in the image in the first place?  
[^5]: It does not seem to be non-functional, but it is quite unstable, and the reflection is slow, or it is not reflected in the first place, or the API response is quite slow. As mentioned above, a mechanism that stacks write tasks in a queue and processes them in order may not be compatible with a server-less environment that terminates in a certain amount of time.  