- Explain the key scenarios Azure Cache for Redis covers and its service tiers.
- Identify the key parameters for creating an Azure Cache for Redis instance and interact with the cache.
- Connect an app to Azure Cache for Redis by using .NET Core.

Caching is a common technique that aims to improve the performance and scalability of a system. It does this by temporarily copying frequently accessed data to fast storage that's located close to the application.

# Explore Azure Cache for Redis
Redis improves the performance and scalability of an application that uses backend data stores heavily. It's able to process large volumes of application requests by keeping frequently accessed data in the server memory
Azure Cache for Redis offers both the Redis open-source (OSS Redis) and a commercial product from Redis Labs (Redis Enterprise) as a managed service.

## Key scenarios
Data cache:	Databases are often too large to load directly into a cache. It's common to use the cache-aside pattern to load data into the cache only as needed. When the system makes changes to the data, the system can also update the cache, which is then distributed to other clients.

Content cache:	Many web pages are generated from templates that use static content such as headers, footers, banners. These static items shouldn't change often. Using an in-memory cache provides quick access to static content compared to backend datastores.

Session store:	This pattern is commonly used with shopping carts and other user history data that a web application might associate with user cookies. Storing too much in a cookie can have a negative effect on performance as the cookie size grows and is passed and validated with every request. A typical solution uses the cookie as a key to query the data in a database. Using an in-memory cache, like Azure Cache for Redis, to associate information with a user is much faster than interacting with a full relational database.

Job and message queuing:	Applications often add tasks to a queue when the operations associated with the request take time to execute. Longer running operations are queued to be processed in sequence, often by another server. This method of deferring work is called task queuing.

Distributed transactions:	Applications sometimes require a series of commands against a backend data-store to execute as a single atomic operation. All commands must succeed, or all must be rolled back to the initial state. Azure Cache for Redis supports executing a batch of commands as a single transaction.

## Service tiers
- Basic:	An OSS Redis cache running on a single VM. This tier has no service-level agreement (SLA) and is ideal for development/test and non-critical workloads.
- Standard:	An OSS Redis cache running on two VMs in a replicated configuration.
- Premium:	High-performance OSS Redis caches. This tier offers higher throughput, lower latency, better availability, and more features. Premium caches are deployed on more powerful VMs compared to the VMs for Basic or Standard caches.
- Enterprise:	High-performance caches powered by Redis Labs' Redis Enterprise software. This tier supports Redis modules including RediSearch, RedisBloom, and RedisTimeSeries. Also, it offers even higher availability than the Premium tier.
- Enterprise Flash: Cost-effective large caches powered by Redis Labs' Redis Enterprise software. This tier extends Redis data storage to non-volatile memory, which is cheaper than DRAM, on a VM. It reduces the overall per-GB memory cost.

# Configure Azure Cache for Redis
You can create a Redis cache using the Azure portal, the Azure CLI, or Azure PowerShell.

## Create and configure an Azure Cache for Redis instance
Name:  The Redis cache will need a globally unique name. The name has to be unique within Azure because it is used to generate a public-facing URL to connect and communicate with the service. The name must be between 1 and 63 characters, composed of numbers, letters, and the '-' character. The cache name can't start or end with the '-' character, and consecutive '-' characters aren't valid.

Location: gion. You should always place your cache instance and your application in the same region. Connecting to a cache in a different region can significantly increase latency and reduce reliability.

## Pricing tier
- `Basic`: Basic cache ideal for development/testing. Is limited to a single server, 53 GB of memory, and 20,000 connections. There is no SLA for this service tier.
- `Standard`: Production cache which supports replication and includes an SLA. It supports two servers, and has the same memory/connection limits as the Basic tier.
- `Premium`: Enterprise tier which builds on the Standard tier and includes persistence, clustering, and scale-out cache support. This is the highest performing tier with up to 530 GB of memory and 40,000 simultaneous connections.

**Microsoft recommends you always use Standard or Premium Tier for production systems. The Basic Tier is a single node system with no data replication and no SLA.**

The Premium tier allows you to persist data in two ways to provide disaster recovery:
- RDB persistence takes a periodic snapshot and can rebuild the cache using the snapshot in case of failure.
- AOF persistence saves every write operation to a log that is saved at least once per second. This creates bigger files than RDB but has less data loss.

## Virtual Network support
You can deploy it to a virtual network in the cloud. Your cache will be available to only other virtual machines and applications in the same virtual network. This provides a higher level of security when your service and cache are both hosted in Azure, or are connected through an Azure virtual network VPN.

## Clustering support
To implement clustering, you specify the number of shards to a maximum of 10. The cost incurred is the cost of the original node, multiplied by the number of shards.

## Accessing the Redis instance
Redis has a command-line tool for interacting with an Azure Cache for Redis as a client. 

Redis supports a set of known commands. A command is typically issued as COMMAND parameter1 parameter2 parameter3.

Here are some common commands you can use:

`ping`	Ping the server. Returns "PONG".
`set [key] [value]`	Sets a key/value in the cache. Returns "OK" on success.
`get [key]`	Gets a value from the cache.
`exists [key]` Returns '1' if the key exists in the cache, '0' if it doesn't.
`type [key]`	Returns the type associated to the value for the given key.
`incr [key]`	Increment the given value associated with key by '1'. The value must be an integer or double value. This returns the new value.
`incrby [key] [amount]`	Increment the given value associated with key by the specified amount. The value must be an integer or double value. This returns the new value.
`del [key]`	Deletes the value associated with the key.
`flushdb`	Delete all keys and values in the database.

## Adding an expiration time to values
In Redis this is done by applying a time to live (TTL) to a key.
When the TTL elapses, the key is automatically deleted, exactly as if the DEL command were issued. Here are some notes on TTL expirations.

- Expirations can be set using seconds or milliseconds precision.
- The expire time resolution is always 1 millisecond.
- Information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped (this means that Redis saves the date at which a key will expire).

```
> set counter 100
OK
> expire counter 5
(integer) 1
> get counter
100
... wait ...
> get counter
(nil)
```

## Accessing a Redis cache from a client
To connect to an Azure Cache for Redis instance, you'll need several pieces of information. Clients need the host name, port, and an access key for the cache. You can retrieve this information in the Azure portal through the Settings > Access Keys page.

The host name is the public Internet address of your cache

The access key acts as a password for your cache. There are two keys created: primary and secondary. You can use either key, two are provided in case you need to change the primary key. You can switch all of your clients to the secondary key, and regenerate the primary key. 

# Interact with Azure Cache for Redis by using .NET
(https://docs.microsoft.com/en-us/learn/modules/develop-for-azure-cache-for-redis/4-interact-redis-api)
A popular high-performance Redis client for the .NET language is `StackExchange.Redis`

## Connecting to your Redis cache with StackExchange.Redis
Recall that we use the host address, port number, and an access key to connect to a Redis server. 

Azure also offers a connection string for some Redis clients which bundles this data together into a single string.

`[cache-name].redis.cache.windows.net:6380,password=[password-here],ssl=True,abortConnect=False`

Notice that there are two additional parameters at the end:

- ssl - ensures that communication is encrypted.
- abortConnection - allows a connection to be created even if the server is unavailable at that moment.

Other parameters: https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md#configuration-options

## Creating a connection
The main connection object in `StackExchange.Redis` is the `StackExchange.Redis.ConnectionMultiplexer` class.
You create a `ConnectionMultiplexer` instance using the static `ConnectionMultiplexer.Connect` or `ConnectionMultiplexer.ConnectAsync` method, passing in either a connection string or a ConfigurationOptions object.

```
using StackExchange.Redis;
...
var connectionString = "[cache-name].redis.cache.windows.net:6380,password=[password-here],ssl=True,abortConnect=False";
var redisConnection = ConnectionMultiplexer.Connect(connectionString);
```

## Accessing a Redis database
The Redis database is represented by the IDatabase type. You can retrieve one using the GetDatabase() method:
`IDatabase db = redisConnection.GetDatabase();`

Here is an example of storing a key/value in the cache:
`bool wasSet = db.StringSet("favorite:flavor", "i-love-rocky-road");`

We can then retrieve the value with the StringGet method:
```
string value = db.StringGet("favorite:flavor");
Console.WriteLine(value); // displays: ""i-love-rocky-road""
```

## Getting and Setting binary values
Recall that Redis keys and values are binary safe. These same methods can be used to store binary data
```
byte[] key = ...;
byte[] value = ...;

db.StringSet(key, value);
```

## Other common operations

(https://github.com/StackExchange/StackExchange.Redis/blob/master/src/StackExchange.Redis/Interfaces/IDatabase.cs)

- `CreateBatch`	Creates a group of operations that will be sent to the server as a single unit, but not necessarily processed as a unit.
- `CreateTransaction`	Creates a group of operations that will be sent to the server as a single unit and processed on the server as a single unit.
- `KeyDelete`	Delete the key/value.
- `KeyExists`	Returns whether the given key exists in cache.
- `KeyExpire`	Sets a time-to-live (TTL) expiration on a key.
- `KeyRename`	Renames a key.
- `KeyTimeToLive`	Returns the TTL for a key.
- `KeyType`	Returns the string representation of the type.  The different types that can be returned are: string, list, set, zset and hash.

## Executing other commands

```
var result = db.Execute("ping");
Console.WriteLine(result.ToString()); // displays: "PONG"
```

You can use Execute to perform any supported commands - for example, we can get all the clients connected to the cache ("CLIENT LIST"):
```
var result = await db.ExecuteAsync("client", "list");
Console.WriteLine($"Type = {result.Type}\r\nResult = {result}");
```

## Storing more complex values
Redis is oriented around binary safe strings, but you can cache off object graphs by serializing them to a textual format - typically XML or JSON.

```
public class GameStat
{
    public string Id { get; set; }
    public string Sport { get; set; }
    public DateTimeOffset DatePlayed { get; set; }
    public string Game { get; set; }
    public IReadOnlyList<string> Teams { get; set; }
    public IReadOnlyList<(string team, int score)> Results { get; set; }

    public GameStat(string sport, DateTimeOffset datePlayed, string game, string[] teams, IEnumerable<(string team, int score)> results)
    {
        Id = Guid.NewGuid().ToString();
        Sport = sport;
        DatePlayed = datePlayed;
        Game = game;
        Teams = teams.ToList();
        Results = results.ToList();
    }

    public override string ToString()
    {
        return $"{Sport} {Game} played on {DatePlayed.Date.ToShortDateString()} - " +
               $"{String.Join(',', Teams)}\r\n\t" + 
               $"{String.Join('\t', Results.Select(r => $"{r.team } - {r.score}\r\n"))}";
    }
}
```

We could use the `Newtonsoft.Json` library to turn an instance of this object into a string:
```
var stat = new GameStat("Soccer", new DateTime(2019, 7, 16), "Local Game", 
                new[] { "Team 1", "Team 2" },
                new[] { ("Team 1", 2), ("Team 2", 1) });

string serializedValue = Newtonsoft.Json.JsonConvert.SerializeObject(stat);
bool added = db.StringSet("event:1950-world-cup", serializedValue);
```
We could retrieve it and turn it back into an object using the reverse process:
```
var result = db.StringGet("event:2019-local-game");
var stat = Newtonsoft.Json.JsonConvert.DeserializeObject<GameStat>(result.ToString());
Console.WriteLine(stat.Sport); // displays "Soccer"
```

## Cleaning up the connection
```
redisConnection.Dispose();
redisConnection = null;
```

# Exercise: Connect an app to Azure Cache for Redis by using .NET Core
(https://docs.microsoft.com/en-us/learn/modules/develop-for-azure-cache-for-redis/5-console-app-azure-cache-redis)
`az group create --name az204-redis-rg --location <myLocation>`
Create an Azure Cache for Redis instance by using the az redis create command
```
redisName=az204redis$RANDOM
az redis create --location <myLocation> \
    --resource-group az204-redis-rg \
    --name $redisName \
    --sku Basic --vm-size c0
```
In the Azure portal navigate to the new Redis Cache you created.
Select Access keys in the Settings section of the Navigation Pane and leave the portal open. We'll copy the Primary connection string (StackExchange.Redis) value to use in the app later.

# Knowledge check
Which of the Azure Cache for Redis service tiers is the lowest tier recommended for use in production scenarios? Standard
Caching is important because it allows us to store commonly used values in memory. However, we also need a way to expire values when they are stale. In Redis this is done by applying a time to live (TTL) to a key. Which value represents the expire time resolution? 1 millisecond