# Common Use Case Example


## Considering the following problem
Let's consider modeling a bank. We've got to model a user and we need to model aspects of that user.
Users have money values and a profile that stores certain information they have. Additionally you will
probably want to model credit cards and how they relate to users. Implement some regular operations:
create, update, delete.




## Solution
```
Consider a user object being represented by the json:
user {
  uuid : uuid
  money : value,
  profile : {
    name : name,
    address : address
  }
  credit : {
    cardId : number,
    transactionIds : [ ]
  }
}


This has been written in Python, you might have some things different depending on your driver.

client = riak.RiakClient()

Creating a user:
riak_obj = client.bucket('users').new('user', data = user)
riak_obj.store()
  
  
Now that our bank has some customers we can work on updating them
Since riak is a simple key store and our data is json this is pretty easy! We can do the following

riak_obj = client.bucket('users').get(uuid)
riak_obj.data['profile.name'] = "johnson"
riak_obj.store()

Since it's json you can see how this will extend to almost any type of update query. Just remember to get it!

Furthermore we can consider deletion:
riak_obj = client.bucket('users').get(uuid)
riak_obj.delete()
```

Now consider that all the modeling is pretty easy as long as you do everything with json! 

Consider that transaction IDs are only unique to a card number! A bank obviously has multiple
users who may have shared tids. Desgin a new system that will allow this to occur.

## Solution 2 riak
```
Consider simply having an object ID be the concatenation of user.credit.cardId
and the specific tid and return the json object that looks something like this

TransactionInfo: {
  ObjectsPurchased : [ ],
  Price : value,
  Date : date
}
riak_obj = client.bucket('transactions').new('transactioninfo', data = TransactionInfo)
riak_obj.store()

Operations are then similar to the above for user.
```
