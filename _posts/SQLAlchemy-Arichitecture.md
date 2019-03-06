Step By Step on SQLAlchemy State Transition
1. create domain object - transient
```
Entity 'user1' is instantiated
Attribute 'name' on entity 'user1' is set to value 'ed'
Create empty collection 'addresses' on entity 'user1'
Entity 'address1' is instantiated
Attribute 'email' on entity 'address1' is set to value 'ed@ed.com'
Entity 'address2' is instantiated
Attribute 'email' on entity 'address2' is set to value 'ed@gmail.com'
Entity 'address3' is instantiated
Attribute 'email' on entity 'address3' is set to value 'edward@python.net'
Append entity 'address1' to collection 'addresses' on entity 'user1'
Append entity 'address2' to collection 'addresses' on entity 'user1'
Append entity 'address3' to collection 'addresses' on entity 'user1'
```
2. Create Session 
```
session starts a transaction container
Entity 'user1' moves from transient to pending
Entity 'address1' moves from transient to pending
Entity 'address2' moves from transient to pending
Entity 'address3' moves from transient to pending
```
3. Flush
```
flush begins
Connection established
a 'snapshot' is established in the database session
execute INSERT statement on table 'user'
Attribute 'id' on entity 'user1' is set to value 1
Attribute 'user_id' on entity 'address1' is set to value 1
Attribute 'user_id' on entity 'address2' is set to value 1
Attribute 'user_id' on entity 'address3' is set to value 1
execute INSERT statement on table 'address'
Attribute 'id' on entity 'address1' is set to value 1
execute INSERT statement on table 'address'
Attribute 'id' on entity 'address2' is set to value 2
execute INSERT statement on table 'address'
Attribute 'id' on entity 'address3' is set to value 3
Entity 'address3' moves from pending to persistent
Entity 'address2' moves from pending to persistent
Entity 'user1' moves from pending to persistent
Entity 'address1' moves from pending to persistent
flush ends
```
4. Invalidate Domain Objects - Expired
```
the 'snapshot' is committed
Expire attributes 'id', 'email', 'user_id' on entity 'address3'
Expire attributes 'id', 'email', 'user_id' on entity 'address1'
Expire attributes 'addresses', 'id', 'name' on entity 'user1'
Expire attributes 'id', 'email', 'user_id' on entity 'address2'
Entity 'address3' is garbage collected
Entity 'address2' is garbage collected
Entity 'address1' is garbage collected
Connection removed
session's transaction container ended
```
5. Query Started New Transaction
```
session starts a transaction container
Connection established
a 'snapshot' is established in the database session
SELECT rows from tables 'user'
SELECT rows from tables 'address'
Entity 'address1' is loaded
Entity 'address2' is loaded
Entity 'address3' is loaded
Create empty collection 'addresses' on entity 'user1'
Append entity 'address1' to collection 'addresses' on entity 'user1'
Append entity 'address2' to collection 'addresses' on entity 'user1'
Append entity 'address3' to collection 'addresses' on entity 'user1'
```
6. Update an Object
```
Attribute 'email' on entity 'address2' is set to value 'edward@gmail.com'
Entity 'address2' is added to the session.dirty list
flush begins
execute UPDATE statement on table 'address'
flush ends
the 'snapshot' is committed
```
7. Expire Domain Objects
```
Expire attributes 'addresses', 'id', 'name' on entity 'user1'
Entity 'address3' is garbage collected
Entity 'address2' is garbage collected
Entity 'address1' is garbage collected
Connection removed
session's transaction container ended
session starts a transaction container
Entity 'user1' is garbage collected
```
![sqlalchemy-session-flow](https://user-images.githubusercontent.com/6065072/53859039-59692800-4017-11e9-884b-4bfb0669f897.png)

**Comments on Session Mode**
* Explicit transaction required, which provides the sqlalchemy when to refresh domain object
* Maintain transaction state in cache including domain object
* Only exist in span of transaction, new query will start new transaction.
* Same primary key, then same domain object in a session
* Auto flush just before the next query
* Unit of work: no need to care about the primary/foreign key record insertion order
* Don't share data between different sessions
![screen shot 2019-03-06 at 8 00 31 pm](https://user-images.githubusercontent.com/6065072/53879905-96024700-404a-11e9-81e7-b0c0b5c6d0ad.png)

**Comments on Active Record Persistence mode**
* thread local and auto commit
* user1 = User.get(id=5); user2=User.get(id=5); user1 is user2 => False
* operations are committed immediately.
issues:
* you need to make sure they are inserted in the right order
* different function need to make sure fetch and refetch carefully.
* uncommited can be leaked into another transaction

ACID:
* Atomic: all or none, no other state
* Consistency: not null/foreign key/primary key
* Isolation: locking/multi-version concurrency control
* Durable: safe after commit

Object as Row Proxy:
* Object is result of SELECT/INSERT statement
* Expired: With no transaction present - no value available in object
* Transient: object outside of session, no row is coresponding to it
* Pending: inside session, but not pushed/coresponding to any row
* Detached: a previously persistent object, but no longer attached to a session.

Benefits of using SQLAlchemy:
* unit of work: track changes on domain object and auto flush out
* identity map: same primary key, you get the same instance
* lazy loading: one-> many relation. load on access
* eager loading: used when lazy loading is not what you want - foreach then access the many relationship
* method chaining to compose sql string.
