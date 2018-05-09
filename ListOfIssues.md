# Things to watch out for

In this file we will discuss some things about Riak that the new user should watch out for.


## Documentation

We have found out that the documentation might become unavailable at any moment because of the nature of basho.
Bash was recently bought out by Bet365 and Riak is transitioning into an open source project. As of writing this
basho.com is down but docs.basho.com is up. For safety we have forked their documentation into this repo.


## Basho no longer exists

As mentioned previously Basho was been acquired by Bet365 which means that typical support avenues are not available
to the typical Riak user. Currently they are transitioning which means more esoteric or specific support requests
might not be available to you. We did not run into these issues but have noted that they could come up.


## Ring Structure

The ring structure of Riak is what really gives Riak a lot of its functionality but it comes at a cost. 
As Riak was based off of a paper written about Amazon's dynamo it shares a lot of similar functionality
especially this ring structure. Things to watch out for on the ring structure are the default replication
of data and the riskt that come with that. The configurable replication is great but it means if you change
certain settings you could end up with unreachable data on your various nodes or your writes could be extremely
slow depending on how much you want any data to be replicated. Additionally as every write is reflected (by default 3)
times to nearby nodes any write will in general be slower than a system that does not do this.
