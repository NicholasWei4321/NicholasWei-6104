# Problem Set 2: Composing Concepts

## Concept Questions

1. The contexts are scopes for each separate set of unique nonce strings. In the case of URLShortening, one instance of URLShortening would have one context, so that in the set of Shortenings, no two shortUrls are the same.
2. The set of used strings must be stored to ensure uniqueness, so that the **generate** action never returns same string more than once. The counter number could be 1:1 mapped/encoded to a unique string, so by monotically increasing the counter, we ensure the same stirng is never generated more than once.
3. One advantage of using real words for the nonce string is that it is easier to remember what the short URL is and potentially easier to type as well. One disadvantage is that the space of unique strings is reduced, which might make nonce strings more guessable. This could be undesirable for users. If I were to modify the NonceGeneration concept, I would add a dictionary of usable word sto the state of the NonceGeneration. Then, the **generate** action would require there to be a word that exists in the dictionary that does not exist in the set of used Strings. It would then return that word and add it to the set of used Strings.

## Synchronization Questions
1. In the generate sync, the shortenURl action does not include the targetUrl argument. This is because the generate sync does not need or modify the targetUrl. Because it only needs the shortUrl base, that is the only argument it needs to take. Parameters that do not affect/are affected by the generate sync are not included. On the other hand, the register sync needs to take both the targetUrl and the shortUrlBase. This is because the register sync needs to associate the targetUrl with the shortUrl constructed from the url base and nonce string. In this case, the target and url base are the minimal arguments needed for the sync to function.
2. If the variable name is different from the parameter name, the mapping has to be explicitly written out so that the binding is clear. Also, in the case that names are repeated, or the same parameter is bound to different variables in different places, explicit binding can help reduce confusion.
3. Requests are used when a request originates from a user. It requires arguments that are passed in externally. On the other hand, the third sync, setExpiry, is an internal process that does not originate from the user, and it requires no arguments that are passed in from the user. Therefore, it does not need a Request action.
4. I would change the sync as follows:
```
  sync generate
  when Request.shortenUrl ()
  then NonceGeneration.generate (context: hardCodedUrlBase)

  sync register
  when
    Request.shortenUrl (targetUrl)
    NonceGeneration.generate (): (nonce)
  then UrlShortening.register (shortUrlSuffix: nonce, hardCodedUrlBase, targetUrl)
```
I removed the paramter shortUrlBase from the parameters of the when action in generate and register. In the then actions, I replaced it with a hardCodedUrlBase (eg. bit.ly).

5. This is my design for a sync for expiry:
```
sync expire_short_url
  when ExpiringResource.expireResource (resource: shortUrl)
  then UrlShortening.delete (shortUrl)
```

## Extending the Design

### 1. New Concepts
I decided to create two new concepts, Analytics and AccessibilityManagement. The Analytics concept keeps track of the actual numbers, ie., how many clicks are associated with each shortened URL. Since the specs say that these Analytics should only be viewable to a user who registered the shortening, I decided that a separate concept could take care of the task of associating register-ers with the analytics they need to see.

```
concept Analytics
purpose  associates resources with the number of times it has been accessed
principle each resource in a set of resources is associated with the number of times it has been accessed. each time it is accessed, the usage count increases.

state
    a set of Resources with
        a useCount Number

actions
    addResource(resource: Resource)
        requires: resource does not exist in set of Resources
        effect: adds resource to set of resources and sets its associated useCount to 0.
    
    incrementUse(resource: Resource)
        requires: resource exists in set of Resources
        effect: increments useCount associated with the specified resource by 1.

    getUse(resource: Resource): (useCount: Number)
        effect: if the resource exists in set of Resources, return the useCount associated with the specified resourcee. otherwise, return nothing.
```

```
concept AccessibilityManagement
purpose  manages what resources each user to access
principle users are associated with resources. users can access only the resources they are associated with.

state
    a set of Users with
        a set of Resources

actions
    addResource(user: User, resource: Resource)
        effect: if the user is not in the set of Users, add it to the set of users. then, add the resource to the set of resources associated with that user.
    
    accessResource(user: User, resource: Resource): (resource: Resource)
        requires: user exists in set of Users
        effect: if the resource exists in the set of Resources associated with the specified user, return that resource. otherwise, return nothing. 
```

### 2. New Syncs
Now, there need to be some additional syncs to incorporate these concepts. There needs to be one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics. (Hint: for the last one, you can simplify by assuming a request for analytics that just asks for the number of lookups for a particular short URL, but that ensures that the result is not visible to everyone.)

I wasn't sure if multiple actions could be performed in one "then" of a sync, so if that happened, I split into two syncs.

1. The first sync occurs when a shortening is registered. The AccessibilityManagement needs to associate the creator of this shortening with an Analytics resource. Also, the Analytics resource needs to add the shortURL as a resource. I needed to split this into 2 syncs.
```
sync createAnalytic
  when Request.shortenUrl (User)
       UrlShortening.register () : (shortUrl)
  then Analytics.addResource (resource: shortUrl)
```
```
sync registerAnalytic
  when Request.shortenUrl (User)
       UrlShortening.register () : (shortUrl)
       Analytics.addResource ()
  then AccessibilityManagement.addResource (user: User, resource: shortUrl)
```

2. The second sync occurs when shortenings are translated to targets. The Analytics needs to increment the usage accordingly.
```
sync updateUsage
  when Request.lookup (shortUrl)
  then Analytics.incrementUse(resource: shortUrl)
```

3. The third sync occurs when a user examines analytics. I needed to split this into two syncs.
```
sync requestAccess
  when Request.accessResource(User, shortUrl)
  then AccessiblityManagement.accessResource(user: User, resource: shortUrl)
```
```
sync accessAnalytics
  when Request.accessResource(User, shortUrl)
       AccessiblityManagement.accessResource(user: User, resource: shortUrl): (resource: shortUrl)
  then Analytics.getUse(resource: shortUrl)
```

The reason I wrote it is this way is because this way, when a user registers a shortening, it is associated with that User. The usage count is updated every time a lookup with that shortening happens. Then, crucially, when a user attempts to access the analytics with a shortening, the AccessibilityManagement.accessResource essentially acts as a Flag. If it returns the shortUrl, then that url is passed to Analytics.getUse, and the data is returned. Otherwise, nothing is passed to analytics.getUse, and the way I specified getUse, if the input to getUse (in this case, nothing) is not in the set of resources, then nothing is returned.

This is a bit convoluted but I wasn't sure if syncs could use conditionals. If they can use conditionals, then I would just have AccessibilityManagement.accessResource return a Flag (True/False). If the flag is True, then Analytics.getUse(resource: shortUrl) would be performed.

### 3. New feature evaluation
Lastly, I need to evaluate some new possible new features.

**Allowing users to choose their own short URLs**: This would require me to change several syncs. However, I don't think any new concepts need to be created. I'm not sure exactly how the sync syntax would allow it, but the **generate** sync would check if a custom suffix is passed in. If it is, the nonce generation does not occur. Instead, in the **register** sync, the shortURLSuffix is set to the customSuffix instead of the nonce generated by the original **generate** sync. Nothing else changes due to the modular design. Since my all the other syncs (including my proposed ones) just take in the shortUrl returned by the **register** sync, the way that the shortUrl is constructed does not affect other components.

**Using the “word as nonce” strategy to generate more memorable short URLs**: The NonceGeneration concept would need to be modified. If I were to modify the NonceGeneration concept, I would add a dictionary of usable words to the state of the NonceGeneration. Then, the **generate** action would require there to be a word that exists in the dictionary that does not exist in the set of used Strings. It would then return that word and add it to the set of used Strings. Like with the previous feature, none of this affects the other concepts or syncs.

**Including the target URL in analytics so that lookups of different short URLs can be grouped together when they refer to the same target URL**: This is undesirable because users shouldn't be able to look at short URLs they didn't generate even if they refer to the same target URL. The point of the Analytics is to assess the usage of URLs that the users themselves created.

**Generate short URLs that are not easily guessed**: This seems slightly undesirable because the definition of guessable is unclear. The nonce generation seems to already do this. Unless the unguessable part is the base URL, and that is extremely undesirable. Overall, this feature does not seem to be very clearly specified, so I don't see how it could be implemented.

**Supporting reporting of analytics to creators of short URLs who have not registered as user**: This could be doable. The generation of a short URL could be accompanied by another generation of a access key/token of some kind. I would reuse the same NonceGeneration concept for that so I don't need any new concepts. Instead, I would add a sync that occurs after the register sync. It would call NonceGeneration.generate. Then, the syncs for accessing analytics need to take in either a User or an accesstoken (from NonceGeneration). If it's not possible for syncs to take in multiple different types, then this would be undesirable.