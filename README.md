## Summary:

The client application may not connect to Redis around 40 seconds during Redis scaling operation.

## Details:

Redis Cache scale C1 to C2 to C1.

Sample Code: Quickstart: Use Azure Cache for Redis in .NET Core | Microsoft Docs
https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/cache-dotnet-core-quickstart

## Code snippet:

``` C#
// Add try catch to handle the Timeout exception

static void Main(string[] args)
        {
            Initial:
            try
            {
                InitializeConfiguration();
                for (int i = 0; i < 10000; i++)
                {
                    RunRedisCommand();
                }
                CloseConnection(lazyConnection);           
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                goto Initial;
            }            
            
        }
        static void RunRedisCommand()
        {
            
            IDatabase cache = GetDatabase();
            // Perform cache operations using the cache object...
            Console.WriteLine("Start: " + DateTime.UtcNow.ToString("yyyyMMdd HH:mm:ss"));
            // Simple PING command
            string cacheCommand = "PING";
            Console.WriteLine("\nCache command  : " + cacheCommand);
            Console.WriteLine("Cache response : " + cache.Execute(cacheCommand).ToString());
            // Simple get and put of integral data types into the cache
            cacheCommand = "GET Message";
            Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
            Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());
            cacheCommand = "SET Message \"Hello! The cache is working from a .NET Core console app!\"";
            Console.WriteLine("\nCache command  : " + cacheCommand + " or StringSet()");
            Console.WriteLine("Cache response : " + cache.StringSet("Message", "Hello! The cache is working from a .NET Core console app!").ToString());
            // Demonstrate "SET Message" executed as expected...
            cacheCommand = "GET Message";
            Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
            Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());
            // Get the client list, useful to see if connection list is growing...
            // Note that this requires allowAdmin=true in the connection string
            cacheCommand = "CLIENT LIST";
            Console.WriteLine("\nCache command  : " + cacheCommand);
            var endpoint = (System.Net.DnsEndPoint)GetEndPoints()[0];
            IServer server = GetServer(endpoint.Host, endpoint.Port);
            ClientInfo[] clients = server.ClientList();
            Console.WriteLine("Cache response :");
            foreach (ClientInfo client in clients)
            {
                Console.WriteLine(client.Raw);
            }
            Console.WriteLine("End: " + DateTime.UtcNow.ToString("yyyyMMdd HH:mm:ss"));
        }
```


**dotnet run > log.txt**

**Start: 20211027 12:33:59**

Cache command  : PING
Cache response : PONG

Cache command  : GET Message or StringGet()
Cache response : Hello! The cache is working from a .NET Core console app!

Cache command  : SET Message "Hello! The cache is working from a .NET Core console app!" or StringSet()
Cache response : True

Cache command  : GET Message or StringGet()
Cache response : Hello! The cache is working from a .NET Core console app!

Cache command  : CLIENT LIST
Cache response :
id=1032 addr=168.63.141.141:12101 fd=6 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1033 addr=168.63.141.141:12102 fd=50 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1089 addr=36.226.71.185:51563 fd=4 name=JACKYNUC age=0 idle=0 flags=N db=0 sub=2 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=subscribe numops=6
id=1090 addr=36.226.71.185:51562 fd=38 name=JACKYNUC age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=client numops=14
id=1034 addr=168.63.141.141:12103 fd=12 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1035 addr=168.63.141.141:12105 fd=42 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
**End: 20211027 12:33:59**

Start: 20211027 12:33:59

Cache command  : PING
Cache response : PONG

Cache command  : GET Message or StringGet()
Cache response : Hello! The cache is working from a .NET Core console app!

Cache command  : SET Message "Hello! The cache is working from a .NET Core console app!" or StringSet()
Cache response : True

Cache command  : GET Message or StringGet()
Cache response : Hello! The cache is working from a .NET Core console app!

Cache command  : CLIENT LIST
Cache response :
id=1034 addr=168.63.141.141:12103 fd=12 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1035 addr=168.63.141.141:12105 fd=42 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1032 addr=168.63.141.141:12101 fd=6 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1033 addr=168.63.141.141:12102 fd=50 name= age=17 idle=17 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=setclientaddr numops=1
id=1089 addr=36.226.71.185:51563 fd=4 name=JACKYNUC age=0 idle=0 flags=N db=0 sub=2 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=subscribe numops=6
id=1090 addr=36.226.71.185:51562 fd=38 name=JACKYNUC age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 ow=0 owmem=0 events=r cmd=client numops=20
End: 20211027 12:33:59

…

**Start: 20211027 12:46:19**

Cache command  : PING
Cache response : PONG

Cache command  : GET Message or StringGet()
Timeout performing GET (5000ms), next: SELECT, inst: 0, qu: 0, qs: 2, aw: False, rs: ReadAsync, ws: Idle, in: 0, serverEndpoint: jackyredisc1.redis.cache.windows.net:6380, mc: 1/1/0, mgr: 10 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=0,Free=32767,Min=12,Max=32767), v: 2.2.79.4591 (Please take a look at this article for some common client-side issues that can cause timeouts: https://stackexchange.github.io/StackExchange.Redis/Timeouts)
Start: 20211027 12:46:24

Cache command  : PING
Timeout performing UNKNOWN (5000ms), next: SELECT, inst: 0, qu: 0, qs: 3, aw: False, rs: ReadAsync, ws: Idle, in: 0, serverEndpoint: jackyredisc1.redis.cache.windows.net:6380, mc: 1/1/0, mgr: 10 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=1,Free=32766,Min=12,Max=32767), v: 2.2.79.4591 (Please take a look at this article for some common client-side issues that can cause timeouts: https://stackexchange.github.io/StackExchange.Redis/Timeouts)
Start: 20211027 12:46:29

Cache command  : PING
Timeout performing UNKNOWN (5000ms), next: SELECT, inst: 0, qu: 0, qs: 5, aw: False, rs: ReadAsync, ws: Idle, in: 0, serverEndpoint: jackyredisc1.redis.cache.windows.net:6380, mc: 1/1/0, mgr: 10 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=0,Free=32767,Min=12,Max=32767), v: 2.2.79.4591 (Please take a look at this article for some common client-side issues that can cause timeouts: https://stackexchange.github.io/StackExchange.Redis/Timeouts)
Start: 20211027 12:46:34

Cache command  : PING
SocketFailure on jackyredisc1.redis.cache.windows.net:6380/Interactive, Idle/Faulted, last: UNKNOWN, origin: ReadFromPipe, outstanding: 7, last-read: 19s ago, last-write: 4s ago, keep-alive: 60s, state: ConnectedEstablished, mgr: 9 of 10 available, in: 0, last-heartbeat: 0s ago, last-mbeat: 0s ago, global: 0s ago, v: 2.2.79.4591
Start: 20211027 12:46:38

Cache command  : PING
No connection is active/available to service this operation: PING; A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond., mc: 1/1/0, mgr: 10 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=0,Free=32767,Min=12,Max=32767), v: 2.2.79.4591
Start: 20211027 12:46:38

Cache command  : PING
No connection is active/available to service this operation: PING; A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond., mc: 1/1/0, mgr: 10 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=0,Free=32767,Min=12,Max=32767), v: 2.2.79.4591
Start: 20211027 12:46:38


…

Start: 20211027 12:47:00

Cache command  : PING
No connection is active/available to service this operation: PING; A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond., mc: 1/1/0, mgr: 9 of 10 available, clientName: JACKYNUC, IOCP: (Busy=0,Free=1000,Min=12,Max=1000), WORKER: (Busy=1,Free=32766,Min=12,Max=32767), v: 2.2.79.4591


**Start: 20211027 12:47:00**

Cache command  : PING
Cache response : PONG

Client can connect to Redis now.

