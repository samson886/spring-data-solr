Spring Data Solr
======================

The primary goal of the [Spring Data](http://www.springsource.org/spring-data) project is to make it easier to build Spring-powered applications that use new data access technologies such as non-relational databases, map-reduce frameworks, and cloud based data services.

The Spring Data Solr project provides integration with the [Apache Solr](http://lucene.apache.org/solr/) search engine. 

Getting Help
------------

* [Reference Documentation](http://static.springsource.org/spring-data/data-solr/docs/current/reference/html/)
* [API Documentation](http://static.springsource.org/spring-data/data-solr/docs/current/api/)
* [Spring Data Project](http://www.springsource.org/spring-data)
* [Issues](https://jira.springsource.org/browse/DATASOLR)
* [Code Analysis](https://sonar.springsource.org/dashboard/index/org.springframework.data:spring-data-solr)

If you are new to Spring as well as to Spring Data, look for information about [Spring projects](http://www.springsource.org/projects).

Quick Start
-----------

### SolrTemplate
SolrTemplate is the central support class for solr operations.
 
 
### SolrRepository
A default implementation of SolrRepository, aligning to the generic Repository Interfaces, is provided. Spring can do the Repository implementation for you depending on method names in the interface definition.

The SolrCrudRepository extends PagingAndSortingRepository 

```java
    public interface SolrCrudRepository<T, ID extends Serializable> extends SolrRepository<T, ID>, PagingAndSortingRepository<T, ID> {
    } 
```
    
The SimpleSolrRepository implementation uses SolrJ converters for entity transformation and therefore requires fields to be annotated with org.apache.solr.client.solrj.beans.Field.

```java
    public interface SolrProductRepository extends SolrCrudRepository<Product, String> {

        //Derived Query will be "q=popularity:<popularity>&start=<page.number>&rows=<page.size>"
        Page<Product> findByPopularity(Integer popularity, Pageable page);

        //Will execute count prior to determine total number of elements
        //Derived Query will be "q=name:<name>*&start=0&rows=<result of count query for q=name:<name>>"
        List<Product> findByNameStartingWith(String name);

        //Derived Query will be "q=inStock:true&start=<page.number>&rows=<page.size>"
        Page<Product> findByAvailableTrue(Pageable page);
  
        @Query("inStock:?0")
        Page<Product> findByAvailableUsingAnnotatedQuery(boolean inStock, Pageable page);
        
        //Will execute count prior to determine total number of elements
        //Derived Query will be q=inStock:false&start=0&rows=<result of count query for q=inStock:false>&sort=name desc
        List<ProductBean> findByAvailableFalseOrderByNameDesc();
        
        //Execute faceted search 
        //Query will be q=name:<name>&facet=true&facet.field=cat&facet.limit=20&start=<page.number>&rows=<page.size>
        @Query(value = "name:?0")
        @Facet(fields = { "cat" }, limit=20)
        FacetPage<ProductBean> findByNameAndFacetOnCategory(String name, Pageable page)
    }
```

 SolrRepositoryFactory will create the implementation for you.

```java 
    public class SolrProductSearchRepositoryFactory {

        @Autwired
        private SolrOperations solrOperations;
  
        public SolrProductRepository create() {
  	        return new SolrRepositoryFactory(this.solrOperations).getRepository(SolrProductRepository.class);
        }
  
    }
```    
   
Furthermore you may provide a custom implementation for some operations.

```java
    public interface SolrProductRepository extends SolrCrudRepository<Product, String>, SolrProductRepositoryCustom {
    	
        @Query(fields = { "id", "name", "popularity" })
        Page<Product> findByPopularity(Integer popularity, Pageable page);
    	
    }
    
    public interface SolrProductRepositoryCustom {

        Page<Product> findProductsByCustomImplementation(String value, Pageable page)
	
    }

    public class SolrProductRepositoryImpl implements SolrProductRepositoryCustom {
	
        private SolrOperations solrTemplate;
	
        @Override
        public Page<Product> findProductsByCustomImplementation(String value, Pageable page) {
            Query query = new SimpleQuery(new SimpleStringCriteria("name:"+value)).setPageRequest(page);
            return solrTemplate.queryForPage(query, Product.class);
        }
        
        @Autowired
        public void setOperations(SolrOperations operations) {
            this.operations = operations;
        }

    }
```

### XML Namespace

You can set up repository scanning via xml configuration, which will happily create your repositories.
 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:solr="http://www.springframework.org/schema/data/solr"
	xsi:schemaLocation="http://www.springframework.org/schema/data/solr http://www.springframework.org/schema/data/solr/spring-solr-1.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<solr:repositories base-package="com.acme.repository" />
	<solr:solr-server id="solrServer" url="http://localhost:8983/solr" />
	
	<bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
		<constructor-arg ref="solrServer" />
	</bean>
	
</beans>
```

Maven
-----

```xml
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-solr</artifactId>
	<version>1.0.0.RC1</version>
</dependency> 

<repository>
	<id>spring-maven-milesone</id>
	<url>http://repo.springsource.org/libs-milestone</url>
</repository>  
```

Contributing to Spring Data
---------------------------
Please refer to [CONTRIBUTING](https://github.com/SpringSource/spring-data-solr/blob/master/CONTRIBUTING.md)
