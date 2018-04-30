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
First create a bucket, we'll use this to store all of our ad objects
adsBucket = myClient.bucket_type('map_bucket').bucket('ads')

We can then represent a single ad the following way
ad = adsBucket.net('ad1')
ad.registers['description'].assign(DESCRIPTION)
ad.registers['data'].assign(DATE)
ad.registers['author'].assign(AUTHOR)
ad.store()

Then you can loop and do all the ads in a given list.
Now that the database has a list of all the ads, let's consider setting a flag if the ad
was answered.

For instance let's say a1 has been answered we could just do
ad.flags['answered'].enable()
ad.store()

Then if we wanted to count maybe how many times someone has requested an ad we could store a counter
ad.counters['viewed'].increment()
ad.store()

We can mirror all of these activities for sets we'd just have to change the bucket type.

To store the users therefore we can do
userBucket = myClient.bucket_type('sets').bucket('users')

users = usersBucket.new('user')
users.add('user1') // Now we have a set containing a user1

userinfoBucket = myClient.bucket_type('map_bucket').bucket('userInfo')
user1 = userinfoBucket.net('user1')
user1.register['name'].assign(NAME)
user1.register['username'].assign(USERNAME)

Now we have a map containing a user's info that is stored in the users set.




```

