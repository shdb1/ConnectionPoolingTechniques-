# ConnectionPoolingTechniques-

Connection pooling is a way to create basic minimal number of connection at the application startup time . Also to decide following things:-

a) minimum number of connection
b) maximum number of connection
c) life of connection in idle case
d) time to evict an idle connection
e) validation query
f) test while borrow
g) minimum idle 
h) maximum idle

and many more.

Connection pooling can be achieved in any appication wherever we do have database requirement either it is web application or stand alone application.

I have seen following CP so far :-

a) Tomcat-JDBC CP.
b) Apache DBCP/DBCP2 
c) Hikari CP

We have seen always DBCP2 was more effecient but when java 8 came , tomcat-jdbc CP has beaten it . here are the details if want to go thru 
https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html .

How to Configure any CP framework with our application:-

To configure any CP framework we should understand first data source.

Being a java developer we have seen DriverManager to get connection always but what is DataSource then.

Data source also a way to get DB connection how we do get from DrivareManger . While we create the connection we do provide connection URL and request DriverManager to provide the connections. on each request we do get a connection from DriverManager. But if we want to think something as factory of connections then DataSource is the option.

DriverManagers are implementaed almost by all DB vendors , for DataSource we do have vendors like apache,hikari and c3p0 etc.

Under the package javax.sql we do have DataSource Interface with following methods

 Public MethodgetConnection() : Connection
 Public MethodgetConnection(String, String) : Connection
 Public MethodInheritedgetLoginTimeout() : int
 Public MethodInheritedgetLogWriter() : PrintWriter
 Public MethodInheritedisWrapperFor(Class) : boolean
 Public MethodInheritedsetLoginTimeout(int) : void
 Public MethodInheritedsetLogWriter(PrintWriter) : void
 Public MethodInheritedunwrap(Class) : T
 
 
 So whatever vendor's API we use for connection pooling will have to give implementation of this interface and they add their new methods also as per their convience to make their API better than others.
 
 So what i can say now Hikari,dbcp,dbcp2 or tomcat jdbc cp are the implementation of javax.sql.DataSource implementation having few additional facilities.
 
 let see one by one how to configure them with our application:-
 
1.Java Web Application:-

If you are on a java webapplication and using Tomcat Server then you can configure using context.xml file. 

<?xml version='1.0' encoding='utf-8'?>
...
  <GlobalNamingResources>
    ...
    <Resource name="jdbc/shdbDB" 
			  global="jdbc/shdbDB"
			  factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
			  auth="Container"
              type="javax.sql.DataSource"              
			  username="test"
			  password="test"
			  driverClassName="com.mysql.jdbc.Driver"
			  description="JCG Example MySQL database."
			  url="jdbc:mysql://localhost:3306/shdbDB"
			  maxTotal="10"
			  maxIdle="10"
			  maxWaitMillis="10000"
			  removeAbandonedTimeout="300"			  
			  defaultAutoCommit="true" />
     ...
  </GlobalNamingResources>'
  
  
We have configured a resource which will be available globally throughout the application.
Note:- here we have used tomcat jdbc pooling

Now we need to use this for getting the connection:-

first of all get the resource from context using JNDI

Context environmentContext = (Context) initialContext
				.lookup("java:comp/env");
        
 Now feth the data source from context object
 
 /**
		 * Name of the Resource we want to access.
		 */
		String dataResourceName = "jdbc/shdbDB";
    
    DataSource dataSource = (DataSource) environmentContext
				.lookup(dataResourceName);
        
  now get the connection using DataSource
  
  Connection conn = dataSource.getConnection();
  
  This is common process for any type of CP Implementation , we just need to change factory in XML config according to the vendor's API.
  
  
 2) Spring Application:-
 
 Either its and Spring web application or standalone application , basic things is we need to have ready object of datasource . We can can ask spring container to create datasource object and then we can use the same data source object anywhere :
 i could see following cases where we could use the DataSource object directly:-
 
 a) it can be passed to jdbc template(if we are using sprinh jdbc template for connections)
 b) it can be passed to hibernate configurataion if we are uing hibernate as ORM.
 c) It can be used as raw object to get DB connection directly.
 
 Following is the sample configuration how we can create the bean for datasource:-
 
 <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/shdbDB" />
		<property name="username" value="root" />
		<property name="password" value="password" />
	</bean>
  
  Note: here we have used apche dbcp2 for connection pooling.
  
  Note: Also connection pooling built on top of java.sql.Connection
