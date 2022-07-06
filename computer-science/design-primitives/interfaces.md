# Interfaces
> An interface is a particular set of method signatures and properties that objects can embody. Objects can embody multiple interfaces (shapes) simultaneously - all of which combined constitute it's own interface (shape).
> - My attempt at defining it

> The interface of an object is the set of requests to which it can respond.
> - Gang of Four

Consider the following function:
```ts
function calculateShipping(item: any) {
  return item.weight * item.volume + minimumCost
}
```
Since this function will take `any` item, it's just trusting that the caller will provide an argument whose shape conforms to the contract (has .weight, .volume). This is dangerous. 

The first thought might be to define a type

```ts
class AmazonItem {
	azid: string
	weight: number
	volume: number
}
```

and then update our function to take this type

```ts
function calculateShipping(item: AmazonItem) {
	return item.weight * item.volume + minimumCost
}
```

Our code is safer now, but less flexible. What if we later wanted to add support for Ebay?

We could declare another class
```ts
class EbayItem {
	ebid: number
	weight: number
	volume: number
}
```

But how do we update the function to accept `EbayItem` as well? We could use a union.
```ts
function calculateShipping(item: AmazonItem | EbayItem)
```

But this isn't scalable. Everytime we add support for a new type of item, we have to edit the function by adding a new union to it's parameter type.
```ts
function calculateShipping(item: AmazonItem | EbayItem | EtsyItem | ShopifyItem | StockXItem | CraigslistItem | FBMarketItem | WishItem)
```

In addition to this being visually cumbersome, it violates the **Open Closed Principle**, as we are having to modify the function everytime we want to open it for extension.

What we can notice is that all of these different item classes share a common set of properties and methods, or a common *interface*. We know that they must, given that the function which takes them contractually expects them to by trying to access certain properties and call certain methods. There is an implied interface that all of these classes share. (why not just do class hierarchy)

So let's explicitly define that interface, or *shape*, that classes must be matched against.
```ts
interface ShippableItem {
	weight: number
	volume: number
}
```

and update our function to take that interface as the type
```ts
function calculateShipping(item: ShippableItem) {
	return item.weight * item.volume + minimumCost
}
```

We can then pass instances of any classes which conform to that interface, or *implement* that interface.
```ts
const amazonItem = new AmazonItem(15,17)
const ebayItem = new EbayItem(10,13)
const customItem = {
	weight: 9,
	volume: 11
	dimensions: `3"x7"x5"` // superfluous as far as the interface is concerned
}

calculateShipping(amazonItem) // works
calculateShipping(ebayItem) // works
calculateShipping(custom) // works
```

Now the compiler will ensure that callers pass arguments which implement the interface; it will also prevent the function's implementation from trying to access members that are not specified in the interface (aka the "superfluous" aspects), even if the particular object passed has that member., the function will not be able to legally access it given that `id` is not part of the `ShippableItem` interface demanded.
```ts
function calculateShipping(item: ShippableItem) {
	console.log('Calculating shipping for item: ', item.id) // error TS2339:
	// Property 'id' does not exist on type 'ShippableItem'.`
	return item.weight * item.volume + minimumCost
}
```

We can see that the first line attempting to reference `item.id` causes a compile error because `id` is outside of the `ShippableItem` interface demanded, so attempting to access it is illegal.

This restriction makes sense, as if we allowed functions to attempt to access extra data that an object of an interface may or may not have, we end up right back to where we started with potentially breakable function calls that makes us have to trust our callers to provide the correct arguments.
