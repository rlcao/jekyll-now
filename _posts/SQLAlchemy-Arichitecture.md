Step By Step on SQLAlchemy State Transition
1. create transient domain object
2. create session from engine 
3. add domain object in session
4. flush(auto flush before next query)
5. connection established
6. flush ended
7. expire domain objects
8. load domain objects
9. ...
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
session starts a transaction container
Entity 'user1' moves from transient to pending
Entity 'address1' moves from transient to pending
Entity 'address2' moves from transient to pending
Entity 'address3' moves from transient to pending
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
Attribute 'email' on entity 'address2' is set to value 'edward@gmail.com'
Entity 'address2' is added to the session.dirty list
flush begins
execute UPDATE statement on table 'address'
flush ends
the 'snapshot' is committed
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

**Comments on Active Record Persistence mode** 
