# PostOffice

This library is a convinient API for the `MailboxProcessor`.
It helps to build a simple actor model.

## Creating a pure `MailboxProcessor` actor
```fsharp
open PostOffice

type Message =
    | Increment of int
    | Decrement of int

let applyMessage state msg =
    match msg with
    | Increment i -> state + i
    | Decrement i -> state - i

let actor = 0 |> Mailbox.buildAgent applyMessage
```
Then you can use `Mailbox` module to interact with your actor:
```fsharp
open PostOffice

let increment myActor =
    Increment 1 |> Mailbox.post myActor

let getCount myActor =
    Mailbox.get myActor
```
## Building an actor network
Build your mailbox processor as shown above, add unique address to it and create a `MailAgent`
That way you don't need to call `Mailbox` module directly
```fsharp
let [<Literal>] myAgentAddress = "address"
let myAgent = (myAgentAddress, []|> Mailbox.buildAgent applyMessage) |> MailAgent
do Increment 1 |> myAgent.Post
let count = myAgent.GetState()
```
When you have your agent(s), you can create a network
```fsharp
let network = new MailboxNetwork()
do network.RespawnBox myAgent
let myAgentInNetwork = network.Box<Message, int32>(myAgentAddress)
```
## Lost messages
Sometimes you mess up your addresses or generic parameters and messages get lost.
You can find the map with all of them in the dead box in your network like this:
```fsharp
let deadLetters = network.DeadLetters.GetState()
```
