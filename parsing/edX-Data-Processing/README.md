Data Analytics Platform
====

A modularized system for managing data collected through McGill's online courses offered via the edX platform. 

Populating Database
----
We first need to populate databases with data from the edX data package. MongoDB is used to store the data. 

The following data is stored in the databases:
* Tracking Logs
* Course Structure
* Forum
* Student Course Enrollment
* Certificates
* User
* User Profile

Each course will have its own database and there is one master database which will contain all tracking log data. All data above, except for tracking logs, is course specific and will be stored in its own collection of the course database. Tracking log data will be first stored in the master database; from this data, course specifc tracking logs are extraced and stored in the course specific database. 

### Master Collection for Tracking Logs
All tracking logs are first stored in a master collection (tracking) in a master database (tracking_logs). The reason we first store the tracking logs in a master database is because the tracking log data provided by edX is logged on a daily basis (not course specific). Tracking logs can be stored in the master database on a daily or weekly basis. 

### Course Specific Collection for Tracking Logs
Tracking log data for a course is defined as the range of data between the date of course enrollment to the date of course completion. Once a course ends, we run the script that extracts the course specific logs from the master collection of the master database to the course specific collection

Before extracting the tracking logs of a course, make sure the course structure data has been migrated to the course specific database. This is because a subset of the course structure data is appended to the corresponding record in the tracking log. 

### Other Data
Apart from the tracking logs, all other data provided by edX is course specific and is provided in any of the .json, .sql and .mongo data formats. 