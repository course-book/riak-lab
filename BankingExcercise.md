# Banking Excercise
We will consider how to model a simplistic bank and its interactions. We will model a `user`
and interactions a user must perform with the bank. 

A `user` must have some reference to their balance, their credit cards, and their profile information. 
Your task is to implement a bank that can handle CRUD operations with these types of users in mind.

You must be able to:
1. Create a `user`.
2. Create a credit card for a `user`.
3. Create transactions for a given credit card.
4. Read the balance of a user given a unique `user` ID (UUID).
5. Read the `user`'s profile information (`name` and `address`).
6. Update a `user`'s profile information (`name` and `address`).
7. Update the `user`'s balance.
8. Delete a credit card for a `user`.

### Solution
One way you might create a user object represented by the JSON string
```
user = {
  uuid : uuid
  money : value,
  profile : {
    name : name,
    address : address
  }
  credit : [
    {
      cardId : number,
      transactionIds : [ ... ]
    }
  ]
}
```

We will be using the python drivers. Here is how you might implement the solution:
```python
import riak
client = riak.RiakClient()

# Create a user
riak_obj = client.bucket('users').new('user', data=user)
riak_obj.store()
```

Now that our bank has users, we can work on updating properties of the user.
Since riak is, at its essence, a key-value store and our data is a JSON, we can do the following:
```python
riak_obj = client.bucket('users').get('user') // Remember that 'user' is whatever you put
in the first argument of the new query!
riak_obj.data['profile.name'] = 'johnson' 
riak_obj.store()
```
You can imagine how this might extend to other needed updates. 
Just remember that you must first get the user object.

To delete a user we do the following
```python
riak_obj = client.bucket('users').get(uuid)
riak_obj.delete()
```

## Added Constraint
Now consider that all the modeling is pretty easy as long as you do everything with JSON.

What if we had an additional constraint that transaction IDs are only unique to a given card number.
Design a system that supports this constraint.

### Constrained Solution
Consider simply having an object ID be the concatenation of user.credit.cardId
and the specific tid and return the json object that looks something like this

```
TransactionInfo: {
  ObjectsPurchased : [ ],
  Price : value,
  Date : date
}
```

```python
riak_obj = client.bucket('transactions').new('transactioninfo', data = TransactionInfo)
riak_obj.store()
```

Operations are then similar to the above for user.
