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

SQL Anywhere Dialects for Hibernate 3.2.2+, 3.5.x, 3.6.x distribution
=====================================================================

  - Supports Hibernate 3.2.2+, 3.5.x, 3.6.x distributions
 
  - Supports SQL Anywhere 12 and 16, using one of the following Dialects (*):
  
  	- org.hibernate.dialect.SQLAnywhereDialect12
    - org.hibernate.dialect.SQLAnywhereDialect12SnapTran
  	- org.hibernate.dialect.SQLAnywhereDialect16
  	- org.hibernate.dialect.SQLAnywhereDialect16SnapTran
  	 	
    with the SQL Anywhere JDBC4 driver, "sajdbc4.jar", after CR #730023

      - Available in SQL Anywhere 12.0.1.3847 and up
      - Available in SQL Anywhere 16, all versions
    
    or the jConnect 7.07 JDBC4 driver "jconn4.jar", on all versions
  
  - Supports SQL Anywhere 10 and 11, using the Dialects (*):
  
    - org.hibernate.dialect.SQLAnywhereDialect10
    - org.hibernate.dialect.SQLAnywhereDialect10SnapTran
    - org.hibernate.dialect.SQLAnywhereDialect11
    - org.hibernate.dialect.SQLAnywhereDialect11SnapTran
  	
   with the jConnect 7.07 JDBC4 driver "jconn4.jar", after CR #680196
   
      - Available in SQL Anywhere 11.0.1.2654 and up
      - Available in SQL Anywhere 10.0.1.4255 and up

(*) The "SnapTran" dialects are intended for those customers who are using
    snapshot isolation in SQL Anywhere (see the notes about this support below)


Installation Instructions
=========================

  1. Unzip the archive contents to your project directory
  2. Include 'SQLAnywhereDialect.jar' on your Java project CLASSPATH
  3. Configure Hibernate to use the new dialect (see 'Sample Hibernate.cfg.xml' below)
     with the appropriate JDBC driver


Sample Hibernate.cfg.xml (with XML Entity mappings)
===================================================

<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD//EN"
    "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
  <session-factory>
  
    <!-- SQL Anywhere JDBC Definition -->
    <property name="connection.driver_class">sybase.jdbc4.sqlanywhere.IDriver</property>
    <property name="connection.url">jdbc:sqlanywhere:dsn=SQL Anywhere 12 Demo;</property>
    
    <!-- jConnect Definition -->
    <!--
    <property name="connection.driver_class">com.sybase.jdbc4.jdbc.SybDriver</property>
    <property name="connection.url">jdbc:sybase:Tds:localhost:2638</property>
    -->
    
    <property name="connection.username">dba</property>
    <property name="connection.password">sql</property>
    
    <property name="dialect">org.hibernate.dialect.SQLAnywhere12Dialect</property>
    
    <mapping resource="path/to/Entity.hbm.xml" />
    
  </session-factory>
</hibernate-configuration>
  

Dialect Language Notes
======================
  - See 'http://docs.jboss.org/hibernate/orm/3.6/reference/en-US/html/types.html' for Hibernate Type Mappings
    
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
  

Dialect Recompilation Instructions
==================================

   1. Unzip the Java source files to an empty current folder, make any neccessary source changes
   2. javac -cp path\to\hibernate-core.jar *.java
   3. mkdir org\hibernate\dialect
   4. move *.class org\hibernate\dialect
   5. jar cvf SQLAnywhereDialect.jar org
   6. Re-include 'SQLAnywhereDialect.jar' in your CLASSPATH
