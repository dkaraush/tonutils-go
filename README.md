# tonutils-go
Golang library for interacting with TON blockchain.

This library is native golang implementation of ADNL and lite protocol. It works like connection pool and can be connected to multiple lite servers in the same time, balancing is done on lib side.

Its concurrent safe and can be used from multiple goroutines.

**This library is under active development**, so more cool features will come soon! Also, if you have some idea of useful functionality, just open issue with description or even pull request! 

## How to use
You can find full usage examples in **example** directory

### Connection
You can get list of public lite servers from official TON configs:
* Mainnet - https://newton-blockchain.github.io/global.config.json
* Testnet - https://newton-blockchain.github.io/testnet-global.config.json

from liteservers section, you need to convert int to ip and take port and key.

Or you can run your own full node, see TON docs.

You can connect like that:
```golang
// initialize new client
client := liteclient.NewClient()
// connect to lite server, can be connected to multiple servers in the same time
err := client.Connect(context.Background(), "65.21.74.140:46427", "JhXt7H1dZTgxQTIyGiYV4f9VUARuDxFl/1kVBjLSMB8=")
if err != nil {
    panic(err)
}

// initialize ton api lite connection wrapper
api := ton.NewAPIClient(client)
```
### Interacting with contracts 
Here are the description of features which allow us to trigger contract's methods

#### Using GET methods
Let's imagine that we have contract with method
```func
cell mult(int a, int b) method_id {
  return begin_cell().store_uint(a * b, 64).end_cell();
}
```

We can trigger it and get result this way:
```golang
// api = initialized ton.APIClient, see Connection in readme

// we need fresh block info to run get methods
block, err := api.GetBlockInfo(context.Background())
if err != nil {
    panic(err)
}

// contract address
addr := address.MustParseAddr("kQB3P0cDOtkFDdxB77YX-F2DGkrIszmZkmyauMnsP1gg0inM")

// run get method `mult` of contract with int arguments 7 and 8
res, err := api.RunGetMethod(context.Background(), block, addr, "mult", 7, 8)
if err != nil {
    panic(err)
}

// we are sure that return value is 1 cell, we can directly cast it and parse
val, err := res[0].(*cell.Cell).BeginParse().LoadUInt(64)
if err != nil {
    panic(err)
}

// prints 56
println(val)
```

### Features to implement
* Support cell and slice as arguments for run get method
* Get account state method
* Send external query method
* Cell dictionaries support
* MustLoad methods
* Event subscriptions
* Parse global config json