# Empower your SAP BPC standard edition with SQL inside the logic script editor

SAP BPC standard is still alive, and support has been extended until 2027.  Therefore I would like to highlight some interesting possibilities you have when working with logic script in SAP BPC.  Performance of logic script has improved with BPC versions running on HANA, however the code is not executed completely in the database, except for the RUN_ALLOCATION statement. Logic script also offers the possibility to call Business Add-Ins (BAdIs) which allow you to custom develop your own logic using ABAP. With the creative use of BAdIs, one can write HANA SQL directly in the logic script editor or even store it in a flat file on the BPC file server.
In order to start writing SQL on top of BPC data, you want to have decent database views to work with as a start.  When creating a SAP BPC standard model, the system automatically creates the infoproviders in the BW back-end with random technical names, within the ‘CPMB’ namespace.  In order to code SQL, the first thing you consider is to activate the HANA calculation view for this infoprovider, but in that case the view will also contain these technical names. It is not convenient to query the columns with the technical names directly, therefore you would prefer to create another view on top of that containing the actual dimension names as columns.  Creating the view manually in the HANA database,  comes with the disadvantage that the view might get invalidated when you would create a new property in the dimension , or even the technical names of the fields might change.  Therefore , ideally, you set up a stored procedure on HANA which dynamically creates these views for you.  This stored procedure can e.g. be invoked automatically every time you process a dimension, by calling this stored procedure in the appropriate dimension process BAdI.
When these views have been activated, you are able to query BPC models with easy-to-read SQL statements inside HANA or ABAP, but not yet directly inside your logic script editor.  In order to achieve this, you will need to setup a custom logic BAdI which has the SQL statement as an input parameter , and passes this SQL string to an ABAP class handling ABAP database connectivity, example of this code is given here : https://help.sap.com/doc/abapdocu_751_index_htm/7.51/en-us/abenadbc_query_abexa.htm
The result of the SQL statement needs to be passed to the output table of the BAdI, in order to have BPC process the result into the model. Therefore you need to pay attention that the result set of your SQL statement respects a specific order of your dimensions ( you can check the correct order by debugging a custom logic BAdI ). 
 
Source : diagrams.net
Now this BAdI can be called from any logic script inside your BPC environment, and passing the SQL directly to the BAdI. Below a simple example where you have created a database view called “DV_BPC_SALES” for a sales model in SAP BPC, and want to use the SQL to copy actual data to plan data.
“
*START_BADI CALL_SQL
	QUERY = OFF
	WRITE = ON
	SQL = SELECT AUDIT_TRAIL, COMPANY_CODE, CUSTOMER, PLANT,  PRODUCT, SALES_ACC
        ,‘2020’ + right(TIME,3) as TIME, ‘PLAN’ AS VERSION FROM “DV_BPC_SALES”
 , SUM( SIGNEDDATA) AS SIGNEDDATA 
WHERE VERSION = ‘ACTUAL’ AND TIME LIKE %2019%
GROUP BY AUDIT_TRAIL, COMPANY_CODE, CUSTOMER, PLANT,  PRODUCT, SALES_ACC, TIME, VERSION
*END_BADI
“
Alternatively, if you work with large SQL statements, you can write the SQL statement in your notepad and upload the file to your BPC file system. You can use the appropriate classes inside the BPC development package UJ to read the file from the file server, and pass the content as a SQL statement to the HANA database. 
Setting up this framework might cause some risks when working with a larger user base using the logic script editor.  Everyone is able to pass any SQL statement to the HANA database, therefore it is advisable to build some checks in your BAdI that blocks forbidden statements like DELETE, TRUNCATE, ROLE, etc..

Conclusion
BPC standard will continue to play an important role as an on-premise planning and consolidation solution, and with some creativity you can upgrade your calculation engine with the power of HANA.  SQL is a widely used and known query language, and enabling it to be used directly into the logic script editor of BPC standard will increase the adoption of HANA SQL for your business calculations.  It is not always ideal to keep up with the different scripting languages (logic script, fox, advanced formulas), therefore this framework allows the use of a widely known standard SQL.  It is also comes with the big advantage that your calculations will be directly executed inside the HANA database, which will increase the performance of your calculations drastically.  When looking at SAP Analytics Cloud, it is a nice to see that advanced formulas are available to code in business logic, but it would be wonderful if SAP could add an SQL editor inside the application. This way, users could choose between another scripting language , or falling back to SQL. It would also allow to move more complex calculations inside SAC. 


