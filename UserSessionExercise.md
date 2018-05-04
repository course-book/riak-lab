# User Session Exercise
In this activity we will describe something a little more advanced for
interested students to apply themselves to. We will discuss an application
known as user session storage/management. This is something that riak
is commonly used for so we figured it would be perhaps the best activity
for this.

## Prompt
Keeping track of user data is a serious task for most modern applications.
You can reduce the problem quite a bit by taking a sort of simplified
approach to what user data is. For example you could only count how often
a user creates, deletes, updates, fetches, logins, etc. These are all simple
counters that Riak is well suited to handle. 

1. Keep track of a user's login number
2. Keep track of a user's number of submitted updates
3. Keep track of a user's number of submitted creates
4. Keep track of a user's number of submitted fetches
5. Keep track of a user's number of submitted deletes

Try the exercise yourself and check our solution below!

## Solution
We've done it in the javascript driver to show the versatility of the platform.
Content is the incoming data from the server.
```javascript
const client = new Riak.Client(IP, (error, c) => // Remember to use the IP of your server!
{
    if (error) {
          this.logger.error(`[ RIAK ] failed to connect: ${error}`);
          reject(false);
          return;
        }
      switch (content.action) {
          case "LOGIN":
            return this.handleLogin(content, c)
              .then((result) => resolve(result));
          case "REGISTRATION":
            return this.handleRegistration(content, c)
              .then((result) => resolve(result));
          case "COURSE_CREATE":
          case "COURSE_DELETE":
            return this.handleCreateOrDelete("COURSE", content, c)
              .then((result) => resolve(result));
          case "COURSE_FETCH":
          case "COURSE_UPDATE":
            return this.handleFetchOrUpdate("COURSE", content, c)
              .then((result) => resolve(result));
          case "WISH_CREATE":
          case "WISH_DELETE":
            return this.handleCreateOrDelete("WISH", content, c)
              .then((result) => resolve(result));
          case "WISH_UPDATE":
          case "WISH_FETCH":
            return this.handleFetchOrUpdate("WISH", content, c);
          default:
            this.logger.warn(`[ RIAK ] Unexpected type ${content.action}.`);
            return resolve(true);
        }
});
 
  handleLogin(content, client) {
    return new Promise((resolve, reject) => {
      const logTag = "LOGIN";
      this.logger.info(`[ ${logTag} ] handling login`);
      const mapOp = new Riak.Commands.CRDT.UpdateMap.MapOperation();
      mapOp.incrementCounter(content.username, 1);
      const datum = {
        bucketType: "maps",
        bucket: content.action,
        key: content.ip,
        op: mapOp
      };
      client.updateMap(datum, (error, result) => {
        let status;
        if (error) {
          this.logger.error(`[ ${logTag} ] failed to store value ${error}`);
          status = false;
        } else {
          this.logger.info(`[ ${logTag} ] successfully stored value ${JSON.stringify(result)}`);
          status = true;
        }
        this.stop(logTag, client);
        resolve(status);
      });
    });
  }
  
    handleRegistration(content, client) {
    return new Promise((resolve, reject) => {
      const logTag = "REGISTRATION";
      const datum = {
        bucketType: "counters",
        bucket: content.action,
        key: content.ip,
        increment: 1
      };
      this.updateCounter(logTag, client, datum, resolve, reject);
    });
  }

  handleCreateOrDelete(logTag, content, client) {
    return new Promise((resolve, reject) => {
      const datum = {
        bucketType: "counters",
        bucket: content.action,
        key: content.username,
        increment: 1
      };
      this.updateCounter(logTag, client, datum, resolve, reject);
    });
  }

  handleFetchOrUpdate(logTag, content, client) {
    return new Promise((resolve, reject) => {
      const datum = {
        bucketType: "counters",
        bucket: content.action,
        key: content.id,
        increment: 1
      };
      this.updateCounter(logTag, client, datum, resolve, reject);
    });
  }

  updateCounter(logTag, client, datum, resolve, reject) {
    client.updateCounter(datum, (error, result) => {
      let status;
      if (error) {
        this.logger.error(`[ ${logTag} ] failed to store value ${error}`);
        status = false;
      } else {
        this.logger.info(`[ ${logTag} ] successfully stored value ${JSON.stringify(result)}`);
        status = true;
      }
      this.stop(logTag, client);
      resolve(status);
    });
  }

```


