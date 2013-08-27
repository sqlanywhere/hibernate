/**
 * This library is free software; you can redistribute it and/or
 * modify it under the terms of the GNU Lesser General Public
 * License as published by the Free Software Foundation; either
 * version 2.1 of the License, or (at your option) any later version.
 *
 * This library is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
 * Lesser General Public License for more details.
 *
 * You should have received a copy of the GNU Lesser General Public
 * License along with this library; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
 *
 * Support: http://sqlanywhere-forum.sybase.com/
 *          http://www.sap.com/services-and-support/
 *
 */

SQL Anywhere Dialects for Hibernate 4.x
=======================================

  - Supports Hibernate 4.x (4.0.0.Final and up)

  - Supports SQL Anywhere 'Sequence' generators in SQL Anywhere 12 and 16
  
  - Supports SQL Anywhere 12 and 16, using one of the following Dialects (*):
  
  	- org.hibernate.dialect.SQLAnywhereDialect12
    - org.hibernate.dialect.SQLAnywhereDialect12SnapTran
  	- org.hibernate.dialect.SQLAnywhereDialect16
  	- org.hibernate.dialect.SQLAnywhereDialect16SnapTran
  	 	
    with the SQL Anywhere JDBC4 driver, "sajdbc4.jar", after CR #730023

      - Available in SQL Anywhere 12.0.1.3847 and up
      - Available in SQL Anywhere 16, all versions
    
    or the jConnect 7.07 JDBC4 driver "jconn4.jar", on all SQL Anywhere 12 and SQL Anywhere 16 versions
  
  - Supports SQL Anywhere 10 and 11, using the Dialects (*):
  
    - org.hibernate.dialect.SQLAnywhereDialect10
    - org.hibernate.dialect.SQLAnywhereDialect10SnapTran
    - org.hibernate.dialect.SQLAnywhereDialect11
    - org.hibernate.dialect.SQLAnywhereDialect11SnapTran
  	
   with the jConnect 7.07 JDBC4 driver "jconn4.jar", after CR #680196
   
      - Available as a separate download ( http://downloads.sybase.com/swd/summary.do?client=support&baseprod=63&contentOnly=true )
      - Available to support SQL Anywhere 11.0.1.2654 and up
      - Available to support SQL Anywhere 10.0.1.4255 and up

(*) The "SnapTran" dialects are intended for those customers who are using
    snapshot isolation in SQL Anywhere (see the notes about this support below)


Installation Instructions
=========================

  1. Unzip the archive contents to your project directory
  2. Include 'SQLAnywhereDialect.jar' on your Java project CLASSPATH
  3. Configure Hibernate to use the new dialect (see 'Sample Hibernate.cfg.xml' below)
     with the appropriate JDBC driver


Sample Hibernate.cfg.xml
========================

		<?xml version='1.0' encoding='utf-8'?>
		
		<!DOCTYPE hibernate-configuration PUBLIC
		      "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		      "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd" >
		
		<hibernate-configuration>
		  <session-factory>
				<!-- Dialect selection -->
		    <property name="hibernate.dialect">org.hibernate.dialect.SQLAnywhere16Dialect</property>
		  
		    <!-- SQL Anywhere JDBC Definition -->
		    <property name="hibernate.connection.driver_class">sybase.jdbc4.sqlanywhere.IDriver</property>
		    <property name="hibernate.connection.url">jdbc:sqlanywhere:dsn=SQL Anywhere 16 Demo;uid=dba;pwd=sql</property>
		    
		    <!-- jConnect Definition -->
		    <!--
		    <property name="hibernate.connection.driver_class">com.sybase.jdbc4.jdbc.SybDriver</property>
		    <property name="hibernate.connection.url">jdbc:sybase:Tds:localhost:2638</property>
		    <property name="hibernate.connection.username">dba</property>
		    <property name="hibernate.connection.password">sql</property>
		    <property name="hibernate.connection.JCONNECT_VERSION">7</property>
		    <property name="hibernate.connection.DYNAMIC_PREPARE">true</property>
		    <property name="hibernate.jdbc.batch.builder">org.hibernate.dialect.sqlanywhere.JConnBatchBuilderImpl</property>
		    -->
		    
		    <!-- Batch statement support (available in SQL Anywhere 11 and up)-->
		    <property name="hibernate.jdbc.batch_size">15</property>   
		
				<!-- Mappings -->    
		    <mapping resource="com/myorganization/MyClass.hbm.xml" />
		    <mapping class="org.myorganization.MyJPAClass" />
		    <!-- ... -->
		  </session-factory>
		</hibernate-configuration>
  
jConnect Support Notes
======================
  
  - java.math.BigInteger (NUMERIC) and java.math.BigDecimal (DECIMAL) fields are not bound correctly if mapped using the
    Hibernate 'org.hibernate.type.BigIntegerType' / 'org.hibernate.type.BigDecimalType' types.
    (e.g. You may receive SQLCODE -158: 'Value %1 out of range for destination' errors).
    
    Instead, these fields must be mapped as 'org.hibernate.type.StringType' types.
    
  - If you intend to use batched statements with jConnect, be aware that the row counts returned from Statement.executeBatch()
    does not reflect the Hibernate default configuration. After attempting to set a batch size, you may encounter errors such as:
    
    ============================
    org.hibernate.jdbc.BatchedTooManyRowsAffectedException:
       Batch update returned unexpected row count from update [0]; actual row count: X; expected: 1
    ============================

		where 'X' is the number of statements executed in the batch.
		
		To workaround this issue, the 'hibernate.jdbc.batch.builder' property setting must be overridden to a new
		'org.hibernate.engine.jdbc.batch.internal.BatchBuilderImpl' implementation that uses the 'Expectations.NONE'
		expectation rather than 'Expectations.BASIC' to verify column counts.
		
		A sample implementation that overrides these settings is provided in the Dialect JAR as
		'org.hibernate.dialect.sqlanywhere.JConnBatchBuilderImpl'
		
		e.g.
		
		============================
		<property name="hibernate.jdbc.batch.builder">org.hibernate.dialect.sqlanywhere.JConnBatchBuilderImpl</property>
		============================
		

Dialect Language Notes
======================
  - See 'http://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch08.html' for Hibernate Type Mappings
    
  - Isolation Levels / Snapshot Isolation Support
    
    * By default, the SQLAnywhereDialect supports ANSI isolation levels and not snapshot isolation
    
      e.g. In 'hibernate.cfg.xml':
      
         <!-- Set Isolation level to 'READ_COMMITTED' -->
         <property name="connection.isolation">2</property>
    
   * To use snapshot isolation instead:

     a) Change the PUBLIC or USER settings of the 'isolation_level' connection option to one of SQL Anywhere's supported SNAPSHOT
        isolation levels. (This is necessary because Hibernate configuration files only directly support 0-3 as isolation levels,
        as per the JDBC specification).
        
        e.g. Run the following SQL:
        
           SET OPTION PUBLIC.allow_snapshot_isolation = 'On';
           SET OPTION PUBLIC.isolation_level = 'snapshot';
           
           or 

           SET OPTION PUBLIC.allow_snapshot_isolation = 'On';
           SET OPTION PUBLIC.isolation_level = 'statement-snapshot';
           
           or
           
           SET OPTION PUBLIC.allow_snapshot_isolation = 'On';
           SET OPTION PUBLIC.isolation_level = 'readonly-statement-snapshot';

   
     b) Specify 'org.hibernate.dialect.SQLAnywhere12DialectSnapTran' or 'org.hibernate.dialect.SQLAnywhere16DialectSnapTran'
        (depending upon your SQL Anywhere version) as your target dialect in hibernate.cfg.xml instead
    
	- Sequence Support
	
   	* Example JPA definition:
	
		  ----------------
		  @Id
		  @SequenceGenerator(name="CustomerSequence", sequenceName="customers_sequence", initialValue=20, allocationSize=1)
		  @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "CustomerSequence")
		  @Column(name = "id")
		      private int id;
		  ----------------
	
	  * The example above uses the sequence named 'customers_sequence' in the database
	
	  * If the sequence does not already exist in the database, it must be created manually or as part of the
	    Hibernate schema 'auto schema-update' feature at run-time - in 'hibernate.cfg.xml':
	
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="hibernate.id.new_generator_mappings">true</property>
	
	   This uses the fields 'initialValue' / 'allocationSize' to create the sequence (otherwise these fields are ignored).
	   
	   (Note: hbm2ddl.auto='update' is not recommended for production scenarios and is only recommended in development).
  
  - UUID Generation Support

   	* Example JPA definition:
	
		  ----------------
      @Id
      @GeneratedValue(generator = "hibernate-uuid")
      @GenericGenerator(name = "hibernate-uuid", strategy = "uuid2")
      @Column(name = "uuidField")
          private UUID uuidField;
		  ----------------

  - LIMIT is not a keyword by default in SQL Anywhere 12/16
  
    * TOP / START AT are used for range queries (available since SQL Anywhere 10)

    * LIMIT should not be a reserved word unless explicitly set in 'reserved_keywords':
         http://dcx.sybase.com/index.html#1201/en/dbadmin/reserved-keywords-option.html
         
    * LIMIT reserved keyword support can be changed in 'SQLAnywhere12Dialect.java' or 'SQLAnywhere16Dialect.java'
      and the Dialect must be recompiled to enable support:
      
      - Uncomment the following line underneath 'registerSA12Keywords()' / 'registerSA16Keywords()':
         
        		//registerKeyword( "limit" );
        		
      - Recompile the SQL Anywhere Dialect classes (see the notes below)
  
  - Database case sensitivity support
  
    * By default, the dialect assumes a case-insensitive database is being used. If this is not true, you must edit the file
      'SQLAnywhere10Dialect.java'
      
      - Uncomment the following line underneath 'areStringComparisonsCaseInsensitive()':
      
      	    //return false;
      
      - Comment the following line underneath 'areStringComparisonsCaseInsensitive()':
      
           return true;
      
      - Recompile the SQL Anywhere Dialect classes (see the notes below)
  
  - LOB Data Types

    * Example JPA definitions for CLOB/BLOB:

		  ----------------
 	    @Lob
		  @Column(name = "ClobField")
  		  private Clob ClobField;				// SA Type: "LONG VARCHAR"

	  	@Lob
  		@Column(name = "BlobField")
	  	  private Blob BlobField;				// SA Type: "LONG BINARY"
		  ----------------

      An 'ImageType' can also be used to be bound as a JDBC LONGVARBINARY

	    ----------------
		  @Lob
		  @Column(name = "BlobField")
		  @Type(type="org.hibernate.type.ImageType")
        private byte[] BlobField;		 // SA Type: "IMAGE" or "LONG BINARY"
		  ----------------
 
   * When using the SQL Anywhere JDBC driver, it is expected to see the INFO message on start-up:

      <TIMESTAMP> org.hibernate.engine.jdbc.internal.LobCreatorBuilder useContextualLobCreation
      INFO: HHH000424: Disabling contextual LOB creation as createClob() method threw error : java.lang.reflect.InvocationTargetException

   * The LOB merge strategy is provided by the 'NEW_LOCATOR_LOB_MERGE_STRATEGY', via a NonContextualLobCreator (SQL Anywhere JDBC4 driver)
     or via a ContextualLobCreator (jConnect 7.07 JDBC4 driver)


Dialect Recompilation Instructions
==================================

   1. Unzip the Java source files to an empty current folder, make any neccessary source changes
   2. javac -cp path\to\hibernate-core.jar *.java
   3. mkdir org\hibernate\dialect
   4. move *.class org\hibernate\dialect
   5. cd sqlanywhere
   5. javac -cp path\to\hibernate-core.jar;path\to\jboss-logging.jar *.java
   6. mkdir ..\org\hibernate\dialect\sqlanywhere
   7. move *.class ..\org\hibernate\dialect\sqlanywhere
   8. cd ..
   9. jar cvf SQLAnywhereDialect.jar org
   10. Re-include 'SQLAnywhereDialect.jar' in your CLASSPATH
