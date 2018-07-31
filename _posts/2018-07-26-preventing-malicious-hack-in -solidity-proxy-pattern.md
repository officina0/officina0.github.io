---
layout: pure/pure-post
title:  "Preventing malicious hack in solidity proxy pattern"
date:   2018-07-26 10:49:41 +0200
categories: solidity
---
Simple article demonstrating how prevent proxy hacks as described by [Patricio Palladino][patricio-palladino]

In the interesting posts above we can learn how to hack a proxed contract and I would like to suggest a different approach to upgradable contract in solidity.

We start anlyzing what proxy pattern do: reverting calling from itself to the proxed object.

But if we know that in our solidity code code we can refer to a contract in a simple way by getting the address, like this `contract(0xAdddress)`.

So my idea is to have a referenced address instead of a proxyfied contract.

We start with a basic contract and we call it `ReferencedContract`
{% highlight ruby %}
pragma solidity 0.4.24;
contract ReferencedContract {

    int private amount;

    function increase(int inc) public {
        amount += inc;
    }

    function getAmmo() public constant returns(int) {
        return amount;
    }

   
  

    

}
{% endhighlight %} 

And we need two other actor, a `ReferencingContract` that will be the contract that refer to our updatable contract and a `UsingReferencig` contract that will work on our `ReferencedContract` by calling `ReferencingContract`
{% highlight ruby %}
contract ReferencingContract{


    address  referenced;


    function getRef() public view  returns(address){
        return referenced;
    }

    function changeRef(address newRef)  public{
        referenced = newRef;
    }
{% endhighlight %} 

We see how we can refer to a contract, by method `getRef` we get the address of referenced contract and by `changeRef` we change the address in case of update;

In [working example][github-repository] we saw a little bit ensecured `changeRef`, allowing only the owner of the contract calling th method.

{% highlight ruby %}
contract UsingReferenceContract {

    ReferencingContract referencing;

    constructor (address ref) {
        referencing = ReferencingContract(ref);
    }

 


    function increaseOnReferenced(int amount) public {
        address ret = referencing.getRef();
        ReferencedContract referenced = ReferencedContract(ret);
        referenced.increase(amount);
        
       
    }
{% endhighlight %} 

And in the end we use our contract by `UsingReferenceContract`.
We define a constructor where we pass our `ReferencingContract` and every time we want to use our `ReferencedContract`
we refer to it by the address given in the reference.

Not so difficult, isnt't so?

In my [github][github-repository] you can find simple example with two test testing using referenced contract, updating his reference and check value in old and in new contract

Thank you for time spent on article,

Christian

[github-repository]: https://github.com/officina0/avoid-proxy-hacks
[patricio-palladino]:https://medium.com/nomic-labs-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357
