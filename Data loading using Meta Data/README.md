# How to load data without having to develop multiple data copy activities?

If a requirement arises to move data from one database to another, a simple way is to create a copy activity for each table. For example, if you have ten tables to copy data from, you may have to create 10 copy activities. Is there any other simple way to do this? Yes, use a meta data-driven approach. Meta data-driven approach has been around for a long time, and SSIS used the same process. In Azure, implementation is slightly different, but the idea remains the same.

Steps;

1. Create a load configuration table.
If you are new to ETL frameworks, load configuration is a table that holds control information about your ETL activities. 
There are the fields usually (may not be exhaustive) a load configuration tables hold 
Source table names -List of all tables that provide source data
Destination table names - List of all tables that that is going to be written into
Enabled - A boolean type field to say whether a particular source data is used in ETL activity or not. If you set this field to 'No' that particular table is skipped.

2. Create a pipeline in Azure data factory and add three linked services, one to read the configuration table and the other two for source and destination respectively. 
3. Create a lookup activity to read data from Configuration then add a For each activity to go through the output of the lookup activity. Since every record represents one ETL activity add a copy activity and set Source and Destination dataset's table names as the Source table name and Destination Table name coming from the configuration table.


Tip #1: always parameterize your datasets, without that your datasets cannot be re-used.
Tip #2: suppose you want to copy activity to another pipeline because you don't want to build everything from scratch. You can go to code view and copy the code for the activity over to the pipeline that you wish to the copy of the activity to be in. Afterwards, you can do all changes. 
Tip #3: In for each activity, you can set (settings->sequential) whether you want activities inside for each to run in parallel or sequentially.
