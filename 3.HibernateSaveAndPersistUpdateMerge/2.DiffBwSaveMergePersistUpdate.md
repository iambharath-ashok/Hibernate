#  Hibernate saveOrUpdate vs Update vs Save Persist

![Hibernate-Architecture](https://thoughts-on-java.org/wp-content/uploads/2017/08/Entity-LifeCycle-State-tiny.png)

## Serializable save()

-	save() method is used to insert the new entity in the database
-	It will Persist the given transient instance, first assigning a generated identifier
- 	It returns the id of the entity created
-	The save method is an “original” Hibernate method that does not conform to the JPA specification
-	We can invoke this method outside a transaction
-	If we use this without transaction and we have cascading between entities, then only the primary entity gets saved unless we flush the session
-	The effect of saving an already persisted instance is the same as with persist. Difference comes when you try to save a detached instance:

		Person person = new Person();
		person.setName("John");
		Long id1 = (Long) session.save(person);
		 
		session.evict(person);
		Long id2 = (Long) session.save(person);
		
-	In the above code, the id2 variable will differ from id1
-	The call of save on a detached instance creates a new persistent instance and assigns it a new identifier, which results in a duplicate record in a database upon committing or flushing

-	save() method does not guarantee that it will not execute an INSERT statement, it returns an identifier, and if an INSERT has to be executed to get the identifier (e.g. "identity" generator), this INSERT happens immediately, no matter if you are inside or outside of a transaction
	

## void persist()

-	persist() is used to insert a new entity in the database
-	persist() method return type is void
-	It’s safe and takes care of any cascaded objects
-	persist() method basically states that:

	-	a transient instance becomes persistent
	-	if an instance is already persistent, then this call has no effect for this particular instance
	-	if an instance is detached, you should expect an exception, either upon calling this method, or upon committing or flushing the session
	
-	id will not be generated right away, regardless of the id generation strategy
-	id will be generated at the time of commit or flush

-	The second call to session.persist() causes an exception, so the following code will not work:

			Person person = new Person();
			person.setName("John");
			session.persist(person);
			 
			session.evict(person);
			 
			session.persist(person); // PersistenceException!	


-	persist() method guarantees that it will not execute an INSERT statement if it is called outside of transaction boundaries



##	Object merge()

-	Hibernate merge() can be used to update existing values, however this method create a copy from the passed entity object and return it
-	The returned object is part of persistent context and tracked for any changes, passed object is not tracked
-	merge method:

	-	finds an entity instance by id taken from the passed object
		-	either an existing entity instance from the persistence context is retrieved, or a new instance loaded from the database
	-	copies fields from the passed object to this instance;
	-	returns newly updated instance
	

-	In the following example we evict (detach) the saved entity from context, change the name field, and then merge the detached entity	
			
			Person person = new Person(); 
			person.setName("John"); 
			session.save(person);
			 
			session.evict(person);
			person.setName("Mary");
			 
			Person mergedPerson = (Person) session.merge(person);


-	 It is the mergedPerson object that was loaded into persistence context and updated, not the person object that you passed as an argument
-	Those are two different objects, and the person object usually needs to be discarded


##	void update()

-	update() method is the original hibernate method(), that present long before merge() method
-	it acts upon passed object
-	its return type is void
-	the update method transitions the passed object from detached to persistent state
-	this method throws an exception if you pass it a transient entity

	
-	In the following example we save the object, then evict (detach) it from the context, then change its name and call update. 
-	Notice that we don’t put the result of the update operation in a separate variable, because the update takes place on the person object itself. 
-	Basically we’re reattaching the existing entity instance to the persistence context — something the JPA specification does not allow us to do


			Person person = new Person();
			person.setName("John");
			session.save(person);
			session.evict(person);
			 
			person.setName("Mary");
			session.update(person);
			
-	Trying to call update on a transient instance will result in an exception. The following will not work:

			Person person = new Person();
			person.setName("John");
			session.update(person); // PersistenceException!




##	void saveOrUpdate()

-	Main difference between save and saveOrUpdate method is that save() generates a new identifier and INSERT record into database 
-	while saveOrUpdate can either INSERT or UPDATE based upon existence of record. 
-	Clearly saveOrUpdate is more flexible in terms of use but it involves an extra processing to find out whether record already exists in table or not
-	In summary  save() method saves records into database by INSERT SQL query
	-	Generates a new identifier and return the Serializable identifier back

-	On the other hand  saveOrUpdate() method either INSERT or UPDATE based upon existence of object in database
	-	If persistence object already exists in database then UPDATE SQL will execute and if there is no corresponding object in database than INSERT will run

	




















