# riak-lab
This activity assumes the driver being used is python, but there are drivers for many languages and instructions found here <https://docs.basho.com/riak/kv/2.0.6/developing/getting-started/> 

# Connecting to Riak
Almost direct reference <https://docs.basho.com/riak/kv/2.0.6/developing/getting-started/python/>

```
import riak
myClient = riak.RiakClient(pb_port=8087)
```

# Short Example
Let's imagine we're reusing one of the scenarios from our previous labs so we have a list of ads.
Each ad has a description, date, author. Each author has a username, and name.

Let's consider how we would model that in Riak.
We could think about this similarly to redis and make several maps.
author: {
  "username":username,
  "name":name
}

Then we simply need a reference to the author map in our ad map which gives us this sort of structure
ad: {
  "description":desc
  "date":date
  "author":author
}

Let's see what this would look like:
```
First create a bucket
adsBucket = myClient.bucket('ads')

Riak inherently supports storing objects as JSON (among other data) 
ad = adsBucket.new(ad

```
