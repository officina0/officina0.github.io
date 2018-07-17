---
layout: pure/pure-post
title:  "Access external data with solidity"
date:   2018-07-17 10:49:41 +0200
categories: solidity
---
Simple article demonstrating how access external data in a Oraclize style with a homemade implementation.


**Attention:** the contracys in example doesn't implement any kind of security, so before using it in production environment it should be reviewed with security constraints.

In this article I want to demonstrate how to handling external data with solidity.

Ethereum contracts cannot communicate directly with the outside world, so they rely on an external entity that push something in it (Oracle pattern).

One of the most used service that do it is using OraclizeAPI, a solid, well architetcured implementation of Oracle pattern.
But if we have some reason not to use it ( eg: we don't want send our data to external entities ) we can rely on ethereum ability to emit event.
Event in Ethereum are just a sort of publish pattern and we have to write the subscribe part for that.
So we can easily deploy a sample contract that will be implemented in all our contracts that will be listenable by some entity:

{% highlight ruby %}
contract CallableBack {


    ListenableContract  listenable;
    function _callBack(string message) public {}

    constructor(ListenableContract listenableContract) {
        listenable = listenableContract;
    }

    

}
{% endhighlight %} 

We define the contract `CallableBack` with just a `_callback` function that will be called by `ListenableContract` defined in costructor.
Before proceding is useful to see also the implementation of `ListenableContract` :

{% highlight ruby %}
contract ListenableContract {
     event ListenEvent(string message,CallableBack emitter);

     function engage(string message) public  {
         emit ListenEvent(message,CallableBack(msg.sender));
     }

     function callBack(CallableBack callableBack,string message) public {
         callableBack._callBack(message);
     }
{% endhighlight %} 
As we can see when `CallableBack` is deployed, the constructor has the reference on an alredy deployed `ListenableContract` instance.
The same contract can be both `CallableBack` both `ListenableContract` and the result is the same, but seaparating it in two differnt contracts we can use single listener for multiple contracts.

So for example we can have a demo contract like the following:
{% highlight ruby %}
contract ListenableDemo is CallableBack{

        uint callBackCalls = 0;
        string[] messages;

       constructor
        (
            ListenableContract listenable
        )
        public
        CallableBack(listenable)
        {

        }


        function listened(string message) public returns(string){
            listenable.engage(message);
            return message;
        }

        function _callBack(string message) public {
            callBackCalls += 1;
            messages.push(message);
        }

        function getCallBackCalls() public view returns (uint callbacks) {
            return callBackCalls;
        }

        function getLastMessage() public view returns(string message) {
            return messages[messages.length-1];
        }

}
{% endhighlight %} 

As we can see there are two important methods: 
- `listened`
- `_callBack`
- 
The first one is delgate to call `ListenableContract.engage` the method that trigger the 
event `ListenEvent` that will be listened as we will see in some row.

The second is `_callBack` and will be called by  `ListenableContract`, through his `callback` method, hooked by listener entity when external data will be ready.

By now we saw the Solidity part, but we miss the last, important thing:
**the listener**


I used web3js to write code example, but it can be done with every web3 client.
{% highlight javascript %} 
      //I supposed doing some work and then call the callback on listenableContract
                    listenable.callBack(demo.address,originalMessage+" after some works").then(() => {
                        //work complete
                        });
                    });

{% endhighlight %}

And is all, by this way we can easily handle event and external data on blockchain by ourside, without depending on external actor.

You can find complete example and working test on [github][github-repository].

Thank you for time spent on article,

Christian

[github-repository]: https://github.com/officina0/external-data-ethereum
