# Spring Data 2
07/12/2020

## Chapter 1 - Spring Data Commons
### Mission
* Legacy of providing superior data access frameworks
* Vast collection of enterprise solutions

### Spring Data Commons
https://projects.spring.io/spring-data
1. Java business entities <-> Persistent target datastore records
2. Lookup records
3. Update records
4. Delete records

#### Repository Pattern
* Based on Repository Pattern
* Create, read, update and delete (CRUDRepository)
* JpaRepository
* MongoRepository
* GemfireRepository

## Chapter 2 - Understanding JPA for Object-Relational Mapping
### Object-Relational Mapping (ORM)
* Physical model to the Logical model
* Physical model = Relational database
* Logical model = Java domain objects

### ORM with Standard Java
1. Open a transaction
2. Make a SQL Query
3. Iterate through each records
4. Iterate through each fiels in a records
5. Extract field, respecting data type
6. Map to the Java object/attribute
7. Close the transactions
7a. For Insert/Update query - commit/rollback transaction

### JPA Is Just a Spectification
* Implementation frameworks: Hibernate, TopLink, and Java EE application servers
* Metadata mapping (XML or Java annotations)
	Java entities <===> Tables
	Java attributes <===> Column/fields
* EntityManager
	Create, read, update, and delete entities
* Still requires boilerplate code to make it work
	
### Map Multiple Tables to Java Classes
* Enrollment is a join table

### Java Persistence Query Language (JPQL)
* Interact with entities and their persistent state
* Portable to any database management system
* Syntax similar to SQL
	* JPQL - entites and attributes
	* SQL - tables and columns
	
### JPA Examples
* Create


	@PersistenceContext
	EntityManager entityManager;
	
	Student create(String name, boolean isFullTime, int age) {
		entityManager.getTransaction().begin();
		Student newStudent = new Student(name, isFullTime, age);
		entityManager.persist(newStudent);
		entityManager.getTransaction().commit();
		entityManger.close();
		return newStudent;
	}
	
* Update
    
    
    @PersistenceContext
	EntityManager entityManager;
	
	void updateAge(int studentId, int age) {
		entityManager.getTransaction().begin();
		Student student = entityManager.find(Student.class, studentId);
		entityManager.persist(student);
		entityManager.getTransaction().commit();
		entityManger.close();
	}`
	
* Read/Lookup

* Update


	@PersistenceContext
	EntityManager entityManager;
	
	List<Student> read(String nameLike) {
		Query query = entityManager.createQuery(
		"select s from Student s where s.name LIKE :someName", Student.class);
		query.setParameter("someName", "%" + nameLike + "%");
		List<Student> result = query.getResultList();
		entityManger.close();
		return result;
	}
	
## Chapter 3 - Introduction to Spring Data JPA
### Spring Data Repository Interfaces

	public interface Repository<T, ID>
	
	T	Domain type the repository manages
	ID	Type of the entity id
	
	
	public interface CrudRepository<T, ID> extends Repository<T, ID>
	
### Create/Update Methods

	T save(T entity);
	
	Iterable<T> saveAll(Iterable<T> entity);
	
### Delete Methods
* Spring Data v2


	void deleteById(ID id)
	void deleteAll(Iterable<? extends T>)
	void delete(T var1)
	void deleteAll()
	
* Spring Data v1.x


	void delete(ID id)
	
### Read Methods
* Spring Data v2


	Optional<T> findById(ID id)
	Iterable<T> findAllById(Iterable<ID> ids)
	Iterable<T> findAll()
	long count()
	boolean existsById(ID id)
	
* Spring Data v1.x

	
	T findOne(ID id)
	Iterable<T> findAll(Iterable<ID> ids)
	
### JpaRepository
All features of _CrudRepository_ plus:
    
     void flush();
     Department saveAndFlush(Department department);
     void deleteInBatch(Iterable<Department> iterable);
     void deleteAllInBatch();
     
Benefits
* No need to acces EntityManagerFactory to flush, for example. We could just call the method, like above. 
* Differentiate from other data repositories. It helps applications to access many types of data stores:
    * MongoRepository: MongoDB
    * SolrCrudRepository: Apache Solr
    * GemfireRepository: Pivotal GemFire
    
### Simple Query Method Property Expression Rules
1. Return type
2. findBy
3. Entity attribute name (use camel case)
4. Optionally, chain subattribute names (i.e., findByAttendeeLastName)

## Chapter 4 - Query by Example
### Query by Example
* User-friendly alternative to SQL
* Lookup objects similar to another object
* Independent of underlying datastore
* Frequently refactored code
* Code requiring nested property constraints or complex string matching


    JPARepository<Department, Integer> extends QueryByExampleExecutor<Department>
    
####Available methods

    List<Department> findAll(Example<Department> example);
    List<Department> findAll(Example<Department> example, Sort sort);
    Optional<Department> findOne(Example<Department> example);
    Page<Department> findAll(Example<Department> example, Pageable pageable);
    long count(Example<Department> example);
    boolean exists(Example<Department> example);
    
### Example<T> example = Example.of(T probe);
Given the following Constructors:

    Department(String name, Staff chair)
    Staff(Person member)
    Person(String firstName, String lastname)

Find the department with the name "Humanities":

    departmentRepository.findOne(Example.of(new Department("Humanities", null)));
    
Find all departments whose chair has the first name of "John":

    departmentRepository.findAll(Example.of(new Department(null, 
                                            new Staff(new Person("John", null)))));
    
### Example<T> example = Example.of(T probe, ExampleMatcher matcher);
Find all departments with the name ending in sciences; ignore case:

    departmentRepository.findAll(Example.of(new Department("sciences", null),
                    ExampleMatcher.matching().
                            withIgnoreCase().
                            withStringMatcher(ExampleMatcher.StringMatcher.ENDING)))
                            
## Chapter 5 - More Repository Types
### Spring Data JDBC
#### Java Persistent API (Hibernate)
**Pros**
* Lazy loading, caching, dirty tracking

**Cons**
* Expensive SQL statements (using lazy loading) and unexpected exceptions
* External database updates not in cache (caching hides recent updates to the database)
* Point of operator persistent not obvious (dirty entity makes it difficult to locate the point of operation persistence)

#### Spring Data JDBC Repositories
**Pros**
* Simpler model, SQL issued when needed, fully loaded object 
    * bypasses lazy loading, caching, dirty tracking
    * SQL queries are issued when and only when you invoke a repository method and return a fully loaded object.
    
**Cons**
* Many-to-one and many-to-many relationships not supported

* Parent and child object lifecycles coupled (following the principles of DDD)

## Chapter 6 - Special Features
### QueryDSL Spring data extension
Spring data query methods are static. To filter by age, fulltime and age, it requires 7 static methods with some possibilities on StudentRepository, for example.

QueryDSL is a dynamic search criteria framework that works with many data sources, as JPA and MongoDB. Spring Data provides an extension by the _QueryDslPredicateExecutor\<Entity\>_.

The available methods are:

    // Predicate is the search criteria
    Student findOne(Predicate predicate);
    Iterable<Student> findAll(Predicate predicate);
    Iterable<Student> findAll(Predicate predicate, Sort sort);
    Page<Student> findAll(Predicate predicate, Pageable pageable);
    long count(Predicate predicate);
    boolean exists(Predicate predicate);

The QueryDSL annotation processors generates Q-classes from entities. Q-classes are search criteria helpers to create BooleanExpression, the building blocks of predicates:

    public class StudentExpressions {
        public static BooleanExpression hasLastName(String lastName) {
            return QStudent.student.attendee.lastName.eq(lastName);
        }
        public static BooleanExpression isFullTime() {
            return QStudent.student.fullTime.eq(true);
        }
        public static BooleanExpression isOlderThan(int age) {
            return QStudent.student.age.gt(age);
        }
    }
    
We can use calls from the repository as following:

    studentRepository.findAll(hasLastName("Smith").and(isFullTime()).and(isOlderThan(20));
    studentRepository.findAll(isFullTime().or(isOlderThan(20));
    studentRepository.findAll(hasLastName("Smith").and(isOlderThan(20));
    
### Auditing
Spring Data provides many ways to implement Auditing in our applications:
#### Entity Annotations
One is as following:

    @CreatedDate
    @Column
    private ZoneDataTime createdAt;
    
    @LastModifiedBy
    @Column
    private User updatedBy;
    
    @CreatedBy
    @Column
    private User createdBy;
    
    @LastModifiedBy
    
#### Implementing Auditable and Extending AbstractAuditable
Or using this:

    public class Staff extends AbstractAuditable<User, Integer> implements Auditable<User, Integer> {}
    
    public Abstract class AbstractAuditable<U, PK extends Serializable> extends AbstractPersistable<PK> implements Auditable<U, PK> {
        private static final long serialVersionUID = 1417182213232323L;
        @ManyToOne
        private U createdBy;
        
        @Temporal(TemporalType.TIMESTAMP)
        private Date createdDate;
        
        @ManyToOne
        private U lastModifiedBy;
        
        @Temporal(TemporalType.TIMESTAMP)
        private Date lastModifiedDate;
    }
    
Either way, we need to pull the user of the session and inject into the entity. Implemented by the AuditorAware:

    public class SpringSecurityAuditorAware implements AuditorAware<User> {
        Public User getCurrentAuditor() {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            if (authentication == null || !authentication.isAuthenticaded()) {
                return null;
            }
            return ((MyUserDetails) authentication.getPrincipal()).getUser();
        }
    }