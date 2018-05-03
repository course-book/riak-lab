## Scalability in Riak 

# Replication

As you might remember from our introduction Riak already does
a lot of replication for you! It is dependent on the n_val setting
in the configuration file to determine how many copies of the data
it makes across its adjacent ring segments. 

Additionally you can configure this down the bucket type level.
For instance you could specific that all buckets of type "map"
must have an n_val of 6 (replicated across 6 adjacent nodes).
Can be done the following way
```
riak-admin bucket-type create n_val_of_2 '{"props":{"n_val":2}}'
riak-admin bucket-type activate n_val_of_2
```
Special Note: You may increase an n_val but you should never decrease an n_val
for already written data because this will cause dead data to be persisted
across some nodes due to the way Riak handles indexing.
Source: http://docs.basho.com/riak/kv/2.2.3/learn/concepts/replication/#setting-the-n-value-n-val

The default value for this is 3 adjacent nodes. 

# Sharding
This is a premium addon on the defunt webpages of basho.
Documentation to be determined.


# Clustering
We've covered this setup in our configuration introduction but
Riak recommends you add additional nodes whenever you will either
run out of space at current usage within 10 days or when your total
capacity reaches 80% of usage.

Clustering also always add additional space and speed.


Sources:
http://docs.basho.com/riak/kv/2.2.3/learn/concepts/replication/
