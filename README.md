# ServiceFabric-StateFullService

Azure Service Fabric is a distributed systems platform provided by Microsoft Azure that simplifies the development, deployment, and management of scalable and reliable applications. It provides a framework for building microservices-based applications that can run on a cluster of machines and can handle high-scale workloads.

Service Fabric organizes applications into small, independently scalable and upgradeable services called microservices. These microservices can communicate with each other and maintain their own state. Stateful services in Service Fabric have their state persisted, making them suitable for scenarios where data needs to be stored and maintained across multiple requests or sessions.

To create a stateful service in Service Fabric, you typically define a reliable stateful service class that inherits from the Microsoft.ServiceFabric.Services.Runtime.StatefulService base class. Here's an example code snippet of a stateful service in C#:

```typescript
using Microsoft.ServiceFabric.Data;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;
using System.Collections.Generic;
using System.Fabric;
using System.Threading;
using System.Threading.Tasks;

namespace StatefulServiceExample
{
    internal sealed class MyStatefulService : StatefulService
    {
        public MyStatefulService(StatefulServiceContext context)
            : base(context)
        {
        }

        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                new ServiceReplicaListener(context =>
                    this.CreateServiceRemotingListener(context))
            };
        }

        protected override async Task RunAsync(CancellationToken cancellationToken)
        {
            IReliableDictionary<string, string> myDictionary =
                await this.StateManager.GetOrAddAsync<IReliableDictionary<string, string>>("myDictionary");

            while (!cancellationToken.IsCancellationRequested)
            {
                using (ITransaction tx = this.StateManager.CreateTransaction())
                {
                    ConditionalValue<string> result = await myDictionary.TryGetValueAsync(tx, "myKey");

                    if (result.HasValue)
                    {
                        string value = result.Value;
                        // Do something with the value...
                    }

                    await tx.CommitAsync();
                }

                await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
            }
        }
    }
}
```

In this example, MyStatefulService inherits from StatefulService and overrides methods like CreateServiceReplicaListeners and RunAsync to define the service's behavior. The CreateServiceReplicaListeners method sets up a service remoting listener to handle communication with other services.

The RunAsync method demonstrates a loop that retrieves and updates the state stored in a reliable dictionary called myDictionary. The state is accessed within a transaction to ensure consistency and durability. In this case, the service retrieves the value associated with the key "myKey" and performs some operation with it.

This is a basic example, and you can extend it to include more complex logic and additional reliable collections like queues or reliable actors to suit your application's requirements.

Remember that to run this code, you need to create a Service Fabric project in Visual Studio, add the necessary NuGet packages, and configure the application manifest and service manifest appropriately.

For a detail explanation see the youtube video: https://www.youtube.com/watch?v=MUiYV3SVJws&list=PLalrWAGybpB_dBdtvLXUjOFp78X97lren&index=7

