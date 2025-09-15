# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a Concept (GiftRegistration)

### 1. Invariants

Two invariants of the state are:

- **Aggregate counts >= 0:** For every Request, the Item count must be >= 0 (or > 0).
- **Request/purchase consistency**: For every Purchase, the Item should exist in the set of Requests, and the total count of each Item across all Purchases should not exceed the count of each Item in Requests.

The **request/purchase consistency invariant** is more important because it is essential to one of the basic needs of a GiftRegistry, which is that guests can only purchase items that are requested by a host, and not more than the host needs.

The **purchase** action is most affected by the second invariant. It preserves the invariant through both its requirements and effects.

1. Requirement: By requiring that the registry exists, is active, and has a request for this item with at least count, the **purchase** action prevents guests from buying extraneous gifts.
2. Effect: By decrementing the count in the matching request after creating the purchase, the **purchase** action ensures that only up to the initial count amount of an item is ever purchased.

### 2. Fixing an action

The **removeItem** action can potentially break the purchase consistency invariant. Its effect is to remove an existing item from the registry. If purchases for that item already exist, removing the request would violate the consistency invariant that states that all Purchases must be for an Item that exists in the set of Requests. 

One simple fix would be to add an additional requirement to the **removeItem** action that there must be no existing Purchases for that Item in order for it to be removed, or that the count for that Item in the set of Purchases equals the count in the initial Request.

### 3. Inferring behavior

Yes, a registry can be opened and closed repeatedly. The spec for **open** only requires that the registry exists and is inactive, and its effect is to make the registry active. The spec for **close** only requires that the registry exists and is active, and the effect is to make the registry inactive. Therefore, as long as the registry exists, it can be opened and closed multiple times.

One reason to allow this is a host might want to temporarily close the registry to edit it, or they might be done with an event, but they want to use the registry for a different event in the future.

### 4. Registry deletion

Deletion could matter because users may want to keep track of previous purchases even for inactive events, and conversely, a host might want to free up storage space.

### 5. Queries

Two common queries are:

- **Query purchases (by owner):** The owner wants to see which of their requests has been purchased and by whom.
- **Query requests (by gift-giver)** Gift-givers want to see which items still need to be purchased.

### 6. Hiding purchases

To augment the concept specification, I would add an additional hide Flag for each Registry. I would also add an additional action, toggleHide. The spec would be:

```
toggleHide (registry: Registry)
    requires: registry exists
    effects: flip the hide Flag
```

Then, for the purchase query from part 5., Purchases should only be shown to the owner if hide is set to False.

### 7. Generic types

Generic identifiers are preferable because they abstract away implementation details about Items and Users, which makes this concept more generalizable to different use cases of registries. They also address problems such as similar/identical names, etc.

## Exercise 2: Extending a familiar concept

### 1, 2. Concept Specification

```
concept PasswordAuthentication
purpose  limit access to known users
principle after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user

state
    a set of Users with
        username String
        password String

actions
    register(username: String, password: String): (user: User)
        requires username is not equal to the username of any existing Users in the set of Users
        effects create a new user with specified username and password and add to set of Users
    
    authenticate(username: String, password: String): (user: User)
        requires a User to exist in the set of Users with the specified username
        effects: if the password matches the password of the existing User, return the matching, otherwise, return nothing.
```

### 3. Essential invariant

The essential invariant is that all usernames are unique, so that no two Users in the state have the same username String. It is held because initially the set of Users is empty. Therefore, no two users could possibly have the same username String. Then, the **register** action maintains this condition because each time the action is performed, a new User is only added if the new username String does not belong to any existing user.

### 4. Extension

I would modify the **register** action to return an additional token String that is emailed to the user via a sync. I would also add an additional **confirm** action that takes in a username String and a token String that completes the registration. Further, I would add a differentiate Users into two sets in the concept state. The first is confirmed users, and the second is pending users. Together, the new concept would look something like this:

```
state
    a set of PendingUsers with
        a User
        a token String
    a set of Users (same as before)
actions
    register(username: String, password: String) : (user: User, token: String)
        requires username not in set of existing Users and not in set of PendingUsers
        effects return a User with specified username and password along with a token that will be emailed to the user via a sync. Add the User and associated token to the set of PendingUsers. 

    confirm(username: String, token: String): (user: User)
        requires username in set of PendingUsers
        effects if the token matches the associated token of a PendingUser, return the User. Remove the User from the set of PendingUsers and add to the set of confirmed Users.
    
    authenticate (same as before)
```

## Exercise 3: Comparing Concepts

### 1. Concept specification for PersonalAccessToken 

```
concept PersonalAccessToken
purpose  revocable credential granting access to specific resources
principle a user is granted tokens that confers certain permissions on what the user can access. Tokens are expirable or revocable, and a user can have multiple tokens.

state
    a set of Users with
        a username String
        a set of Tokens
    a set of Tokens with
        a token_string String
        a token_name String
        an expires_at Number
        a set of Permissions

actions
    createToken(username: String, token_name: String, expires_at: Number, permissions: Set(Permissions)): Token
        requires username to exist in set of Users and token_name to not be in the set of Tokens associated with the specified User
        effects create a new Token with a token_string (generated by some random/algorithmic method) with the specified token_name, an expiration time expires_at, and the specified set of permissions. Adds this new Token to the set of Tokens associated with the specified User

    authenticateWithToken(username: String, token_name: String, requested_perm: Permission): Permission
        requires username to exist in set of Users and token_name to exist in set of Tokens associated with the specified User
        effects if the requested_perm exists in the set of Permissions in the Token associated with the specified token_name, and the current time has not exceeded the expires_at time associated with that Token, return the Permission. Otherwise, return nothing.

    revokeToken(username: String, token_name: String): Token
        requires username to exist in set of Users and token_name to exist in set of Tokens associated with the specified User
        effects: changes the expires_at time for the Token associated with token_name to be set to a time such that the Token instantly expires.
```

Note: alternatively, I might add an extra active Flag to the Token state. Then, revokeToken simply sets this Flag to False, and authenticate only checks the active Flag instead of the expires_at time. We might need some external sync that can check whether the expires_at time as passed, and if so, set the active Flag to false.

I also abstracted Permissions as a generic parameter (like with User and Item) because I think the concept of a permission could be generalized to many different scenarios.

### 2. How PersonalAccessToken differs from PasswordAuthentication

PersonalAccessToken differs from PasswordAuthentication in that a username can be associated with more than one token, and authentication with tokens grants access to specific permissions, rather than access to a User account. Moreover, tokens have the ability to expire, whereas passwords generally don't. 

I would change the GitHub documentation to be more explicit that a user can have multiple access tokens, and that tokens are meant to be used to grant access to specific permissions/resources rather than a user account. 

## Exercise 4: Defining familiar Concepts

I will create concept specifications for URL Shortener, Conference Room Bookings, and Electronic Boarding Pass.

### 1. URL Shortener
```
concept URLShortener
purpose  shortens web URL strings
principle a user submits a URL and optionally a user-defined suffix. A short URL that redirects to the original user-submitted URL is created, ending with the optional user-defined suffix if it exists or an autogenerated one otherwise. 

state
    a homeDomain String
    a set of URLPairs with
        an originalURL String
        a shortenedURL String
        an active Flag

actions
    shortenURL_default(url: String) : URLPair
        requires: None (see note below)
        effects: Create an active URLPair consisting of the original URL and a shortened URL and return it. The shortened URL should consist of the homeDomain and a short string of characters generated from a sync. The shortened URL must not be associated with an active URLPair. The shortenedURL should direct to the orginal URL.

    shortenURL_custom(url: String, suffix: String) : URLPair
        requires: the shortened URL consisting of the homeDomain and the suffix does not exist in the set of URLPairs
        effects: Create an active URLPair consisting of the original URL and a shortened URL and return it. The shortened URL should consist of the homeDomain and the specified suffix. The shortenedURL should direct to the orginal URL.
```
Notes: I put the requirements to be none for the default shortener action. This means that 1. the original URL doesn't have to valid and 2. multiple shortened URLs can map to the same original URL. I think this is justifiable because 1. the core purpose of a URL shortener is to create redirect links, which doesn't depend on whether the redirect target site actually exists, and 2. since multiple custom URLs can map to the same original URL, it seems simpler to also allow multiple randomly generated URLs to map to the same original URL as well. 

I'm not sure exactly how to expire old shortened URLs. It's not really a user interaction so I don't think it is an **action**. I think maybe a sync can take care of expired URLs. Maybe the URLPair could have a time_created Number and something else deletes URLs once a certain time has passed since the time_created.

### 2. Conference Room Bookings
```
concept ConferenceRoomBooking
purpose reserves conference rooms in a shared institution space
principle a user selects a conference room to reserve, along with a time block during which the user wants to occupy the room. The room is then booked/associated with the user for that time period

state
    a set of Rooms with
        an id String
        a set of Slots
        a set of Bookings
    a set of Bookings with
        a User
        a Slot
    a set of Slots with
        a startTime Time
        an endTime Time

actions
    createBooking(user: User, roomID: String, startTime: Time, endTime: Time): Booking
        requires: roomID is a valid id, startTime occurs before endTime, the room associated with the roomID does not have a Booking that overlaps with the specified time slot
        effects: create a Booking with the specified User and a Slot with the specified start/end time. Add the Booking to the set of Bookings associated with that Room
    
    deleteBooking(roomID: String, booking: Booking): Booking
        requires: roomID is a valid id, the room associated with the roomID contains the specified booking
        effects: remove the booking from the set of Bookings associated with the Room with the specified roomID

    addSlot(roomID: String, startTime: Time, endTime: Time): 
        requires: roomID is a valid id, startTime occurs before endTime
        effects: adds a Slot consisting of the specified start/end time to the set of Slots associated with roomID

    deleteSlot(roomID: String, startTime: Time, endTime: Time):
        requires: roomId is a valid id, startTime occurs before endTime
        effects: removes the Slot consisting of the specified start/end time from the specified room. 
```
Notes: The CSAIL reservation system doesn't seem to have a room capacity or amenities feature. That could be an extension of this basic RoomReservation system. Another extension could be to edit a booking. Currently, that can be done by deleting and rebooking. For deleteSlot, I didn't require bookings to be associated with a slot to be deleted because I think it's justifiable for a space (eg. a library) to delete slots even if they are already reserved, maybe for a special event or a holiday. A separate sync would be needed to notify users that the slot was deleted.

### 3. Electronic Boarding Pass
```
concept ConferenceRoomBooking
purpose reserves conference rooms in a shared institution space
principle a user selects a conference room to reserve, along with a time block during which the user wants to occupy the room. The room is then booked/associated with the user for that time period

state
    a set of Airlines with
        an airlineID: String
        a set of Flights
    a set of Flights with
        a flightID: String
        a gate: String
        a boardingTime: Time
        a departureTime: Time
        a departureLocation: Location
        an arrivalLocation: Location
    a set of BoardingPasses with
        a passID: String
        a flightID: String
        a ticketNumber: String
        a passenger: Passenger {String/Number fields associated with a human passenger}
        a seatNumber: String
        a boardingGroup: String
        a checkedIn Flag
        an active Flag

actions
    issuePass(airlineID: airlineID, flightID: String, passenger: Passenger, ticketNumber: String): BoardingPass:
        requires airlineID is valid, flightID exists in the set of Flights associated with the specified Airline and the passenger has purchased a ticket for the plane (reflected in ticketNumber)
        effects create an active BoardingPass with the specified passenger and flightID. The boarding pass will therefore contain all the information about a flight associated with that particular flightID. The checkedIn flag should be initialized to False.

    updateFlight(airlineID, flightID, gate: String, boardingTime: Time, departureTime: Time): Flight
        requires airlineID is valid, flightID exists in the set of Flights associated with the specified Airline 
        effects update the information for the Flight associated with the flightID. All boarding passes with that flightID will then be able to reflect the modified information.

    updatePass(airlineID: String, passID: String, seatNumber: String, boardingGroup: String): BoardingPass
        requires airlineID is valid, the BoardingPass with the specified passID has a flightID that exists in the set of Flights associated with the specified Airline 
        effects: update the boarding information in the BoardingPass accordingly.

    checkIn(flightID: String, passID: String): BoardingPass
        requires that the passID is a valid passID
        effects sets the checkedIn flag for the BoardingPass associated with the passID to True
        
    scanPass(flightID: String, passID: String): True/False
        requires that flightID be valid
        effects: successful scan if the BoardingPass associated with the passID contains the matching flightID and the checkedIn Flag is True, otherwise, failed scan.
```
Notes: The scanPass action could also return a more complex output, maybe containing more information on the result of a scan (icluding passenger info), but the basic functionality is that it should either pass the scan at the gate or fail.