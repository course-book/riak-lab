# riak-lab
This activity assumes the driver being used is python, but there are drivers for many languages and instructions found [here](https://docs.basho.com/riak/kv/2.2.3/developing/getting-started)

# Connecting to Riak
Almost direct [reference](https://docs.basho.com/riak/kv/2.2.3/developing/getting-started/python/)

```python
import riak
my_client = riak.RiakClient(pb_port=8087)
```

# Short Example
Let's imagine we're reusing one of the scenarios from our previous labs so we have a list of ads.
Each ad has a description, date, author. Each author has a username, and name.

Let's consider how we would model that in Riak.
We could think about this similarly to redis and make several maps.
```
author: {
  "username": username,
  "name": name
}
```

Then we simply need a reference to the author map in our ad map which gives us this sort of structure
```
ad: {
  "description": desc,
  "date": date,
  "author": author
}
```

Let's see what this would look like:

First create a bucket, we'll use this to store all of our ad objects
```python
ads_bucket = my_client.bucket('ads')
```

We can then represent a single ad the following way
```python
ad = ads_bucket.net('ad1')
ad.registers['description'].assign('some description')
ad.registers['data'].assign('some data')
ad.registers['author'].assign('some author')
ad.store()
```

Then you can loop and do all the ads in a given list.
Now that the database has a list of all the ads, let's consider setting a flag if the ad
was answered.

For instance let's say `a1` has been answered we could just do
```python
ad.flags['answered'].enable()
ad.store()
```

Then, if we wanted to count how many times someone has requested an ad we could store a counter
```python
ad.counters['viewed'].increment()
ad.store()
```

We can mirror all of these activities for sets. We would just have to change the bucket type.

To store the user's info, we can do
```python
user_bucket = my_client.bucket('users')

users = users_bucket.new('user')
users.add('user1') // Now we have a set containing a user1

userinfo_bucket = my_client.bucket('user_info')
user1 = userinfo_bucket.net('user1')
user1.register['name'].assign('some name')
user1.register['username'].assign('some username')
```

Now we have a map containing a user's info that is stored in the users set.
