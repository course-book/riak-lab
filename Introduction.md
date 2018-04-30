# Introduction to Riak
Riak is a distributed key-store that ensures maximal data availability by
storing the data across multiple servers. It is an open source NoSQL
database with high availability, fault tolerance, and is quite scalable.
Riak is written in Erlang and supports automatic data replication and
data distribution via consistent hashing. This gives it a purported
near-linear performance increase as you add to its capacity. Riak supports
variable consistency depending on the configuration. The unique thing
about Riak is not the typical master slave model. Riak stores key-values
into a bucket then the bucket and key are hashed together which maps
the result to a 160 bit integer space. Riak then divides this space into
partitions which are managed by virtual nodes. Physical nodes then divide
up the virtual nodes among themselves. The way Riak does implicit data
replication is that when you write data to one of these virtual partitions
it is automatically copied to n more partitions that are adjacent to the
written one. Riak is based off a paper written about Amazon’s Dynamo therefore
 it shares a lot things in common with Dynamo.

# Underlying Data Model:
## Enabling Data Types
By default, Riak is schemaless and does not enable any data types for you.
If you want to use these data types listed below, you will have to enable
it using the riak-admin CLI tool. To enable a data type, run:
```shell
  riak-admin bucket-type create <data-type> ‘{“props”: {“datatype”: “datatype”}}’
```
For example, to enable the map data type you would run:
```shell
  riak-admin bucket-type create maps ‘{“props”: {“datatype”: “map”}}’
```

You can read more about [here](http://docs.basho.com/riak/kv/2.2.3/developing/data-types/).

## Data Types Overview
In this section we will give a short summary of what each data type supported
by Riak out-of-the-box. We will assume all examples are executed using the
 `python`
[driver](https://docs.basho.com/riak/kv/2.2.3/developing/getting-started/python/).
You can read more about each data type
[here](http://docs.basho.com/riak/kv/2.2.3/developing/data-types/)

### Bucket
A Bucket is essentially a flat namespace in Riak. It allows you to duplicate
key names as long as they exist in separate buckets.

#### Example
```python
# Creates a default bucket with name key
customers  = client.bucket(key)
```

### Flags
Flags are essentially boolean values and can only be stored within a map.

#### Example
```python
map.flags[‘enterprise_customer’].disable() # Set the value
map.store() # Writes the value
map.reload().flags[‘enterprise_customer’].value # Get the value
```

### Registers
Registers are named binary values like strings. They must also be used in
a map exclusively.

#### Example
```python
map.registers[‘first_name’].assign(‘User’)
map.registers[‘phone_number’].assign(‘5555555555’)
map.store()
```

### Counters
Counters: are bucket level and can be used by themselves. Their values can be
positive, zero, or negative integers.

#### Example
```python
bucket = client.bucket_type(‘counters’).bucket(key) # Create the bucket for it
counter = bucket.new(key) # Create the counter
counter.increment(value) # Can be empty for 1 or an integer value
counter.decrement(value)
counter.store() # Write to server
counter.value
```

### Sets
Sets are bucket level and can be used by itself or be put into another
collection like a bucket. This is a collection of binary values that are
unique. Duplicate additions will be ignored by Riak.

#### Example
```python
set = bucket.new(key)
# or create the bucket
key = client.bucket_type(‘sets’).bucket(‘travel’)
cities_set = key.new(‘cities’)
```

### Maps
All data types can be put in a map. This enables a map to be the basic data
type of Riak.

#### Example
```python
customers = client.bucket_type(‘map_bucket’).bucket(‘customers’)
map = customer.net(key)
```

### HyperLogLogs
This is a data structure that is used to estimate (within 2%) of the number of
distinct entries of an input. This data structure is used in queries as a
representation of the `HyperLogLog`
[algorithm](https://en.wikipedia.org/wiki/HyperLogLog).

#### Example
```python
bucket_type = client.bucket_type('hlls')
bucket = bucket_type.bucket('my_hlls')
hll = bucket.new(key)
```

# Installation of Riak
Installation process is paraphrased from general installation page
For direct information about Ubuntu look
[here](https://docs.basho.com/riak/kv/2.2.3/setup/installing/).

Assuming that you are installing on the Ubuntu 16.04 operating system, execute
the command
```shell
  curl -s https://packagecloud.io/install/repositories/basho/riak/script.deb.sh | sudo bash
  sudo apt-get install riak
```

# Configuring Riak
## Basic configuration
- Riak was written almost exclusively in Erlang and runs on an Erlang
  virtual machine. Before building and starting a cluster, there are some
  Erlang-VM-related changes that you should make to your configuration files.
  In your `riak.conf` file, add the next two lines:
  ```
    erlang.schedulers.force_wakeup_interval = 500
    erlang.schedulers.compaction_of_load = false
  ```
- Before using the cluster, you will need to set the ring size,
  the number of data partitions that comprise the cluster, which
  will impacts the scalability and performance of a cluster. This
  needs to be done before the cluster receiving any data. Change
  the ring creation size parameter by uncommenting it in
  `riak.conf` and setting it to the desired value, for example:
  ```
    ring_size = 64
  ```
- Other available configuration options are
   [here](http://docs.basho.com/riak/kv/2.2.3/configuring/reference/).
## Cluster configuration
- Configuring a Riak cluster involves instructing each node to
  listen on a non-local interface (i.e. not
  `127.0.0.1/localhost`) and then joining all of the nodes
  together to participate in the cluster. Most configuration
  changes will be applied to the `riak.conf` file located in your
  `/riak/etc` directory.
- Configuring the first node. First, you need to stop the Riak
  node if it is currently running:
  ```shell
    riak stop
  ```
- Second, you need to select an IP address and port:
  If you are using the protocol buffers interface:
  ```
    listener.protobuf.internal = <IP Address>:<port number>
  ```
  If you are using HTTP interface:
  ```
    listener.http.internal = <IP Address>:<port number>
  ```
- Next, you need to name your node. Every node in Riak has a name
  associated with it, the default name is riak@127.0.0.1. You can
  change it from the `riak.conf` file:
  ```
    nodename = riak@<IP Address>
  ```
- Now, your node is properly configured, you can start it by:
  ```shell
    riak start
  ```
- If the Riak node has been previously started, you must use the
  following command to change the node name and update the node’s
  ring file:
  ```shell
    riak-admin cluster replace <old nodename> <new nodename>
  ```
- As with all cluster changes, you need to view the planned
  changes by running the following command to finalize those
  changes:
  ```shell
    riak-admin cluster plan
    riak-admin cluster commit
  ```

## Adding a second node to your cluster
- Repeat the above steps for a second host on the same network,
  giving the second node a different host/port and node name.
  Then start the second node.
- Use the following command to join the second node to the first
  node, thereby creating an initial Riak cluster:
  ```shell
    riak-admin cluster join <first node’s nodename>
  ```
- Next, plan and commit the changes using:
  ```shell
    riak-admin cluster plan
    riak-admin cluster commit
  ```
- After the last command, you should see Cluster changes
  committed. You can use the same approach to add more nodes to
  the cluster.

## Checking your Riak cluster status
- You can use `riak-admin` command to check the status from shell
  command line:
  ```shell
    bin/riak-admin status | grep ring_members
  ```
- If you are running your riak node, just use:
  ```shell
    riak-admin status
  ```

## Removing a node from a cluster
You can use the riak-admin command from any other node in the cluster to do so:
```shell
  riak-admin cluster leave <nodename of the leaving node>
```

## Riak Search Setting
Riak support a function called Riak Search with Solr integration.
Riak Search is off by default. To enable it, you need to add the following
line to `riak.conf`:
```
  search = on
```

In the default `riak.conf` file, you can find all the Riak Search
configuration settings in `riak.conf`. Setting `search = on` is required,
but other search settings are optional. You can explore them more at link.

## Managing Configuration:
A simple command can help you retrieving a list of all of the configs currently applied in the node:
```shell
  riak config effective
```

For detailed information about a particular configuration variable, use
```shell
  riak config describe <variable> command.
```
For example:
```shell
  riak config describe ring_size
```
Riak has a `chkconfig` command that enables you to determine whether the
syntax in your configuration files is correct. It will output `config is OK`
if your configuration files are syntactically sound:
```shell
  riak chkconfig
```

Riak supports a command that you can use to debug your configuration:
```shell
  riak config generate -l debug
```

# Warm Up Activity:
Now that you have read through a quick introduction and how to setup
your own Riak cluster, head over to our
[Warm Up Activity](https://github.com/course-book/riak-lab/blob/master/WarmUpActivity.md).

# Resources
## Introduction
- https://github.com/basho/riak
- https://en.wikipedia.org/wiki/Riak
- https://www.techopedia.com/definition/26740/riak
- https://github.com/course-book/basho_docs

## Underlying data model examples
- http://docs.basho.com/riak/kv/2.2.3/developing/data-types/maps/
- http://docs.basho.com/riak/kv/2.2.3/developing/data-types/counters/
- http://docs.basho.com/riak/kv/2.2.3/developing/data-types/sets/

## Configuration reference:
- http://docs.basho.com/riak/kv/2.2.3/configuring/reference/

## Cluster configuration:
- http://docs.basho.com/riak/kv/2.2.3/using/
