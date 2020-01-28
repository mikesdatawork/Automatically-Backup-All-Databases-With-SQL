![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")       

# Automatically Backup All Databases With SQL
**Post Date: October 3, 2014**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>The following SQL logic will perform the following actions.

1. Automatically create a backup folder structure on a network share ( you just provide the share name )
2. Automatically delete old Full Database Backups if they exist ( current retention is 2 days. feel free to modify )
3. Automatically perform Full Database Backups on all Databases.
4. Automatically delete old Transaction Log Backups if they exist ( current retention is 2 days. feel free to modify )
5. Automatically perform Transaction Log Backups on all Databases.
6. Automatically shrink Transaction Log Data Files down to the lowest 8kb increment.
7. Automatically perform DBCC CheckDB across all Databases.
8. Automatically shrink Data Files down to the lowest 8kb increment.

I recommend copying the logic and creating 3 Agent Jobs in the following construct:

Job1:
DATABASE BACKUP FULL ALL DATABASES
Step 1. Delete old full backups.
Step 2. Run Full Database Backups

Job2:
DATABASE BACKUP TLOG ALL DATABASES
Step 1. Delete old transaction log backups.
Step 2. Run Transaction Log Backups.
Step 3. Shrink Transaction Log Data Files.

Job3:
DATABASE MAINTENANCE
Step 1. Run DBCC CheckDB
Step 2. Shrink Data Files.
You will need to ensure the SQL Service Account has the appropriate rights on the network share to create Folders and Files.
Hope this is useful.</p>    


## SQL-Logic
```SQL
use master;
set nocount on
 
-- create backup folder structure
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- full database backups to network share
declare @backup_all_databases varchar(max)
declare @dayname varchar(255)
declare @timestamp varchar(255)
declare @type varchar(255)
set @type = 'FULL'
set @dayname = ( select DATENAME(dw, GETDATE()) )
set @timestamp = ( select replace(replace(replace(REPLACE(CONVERT(char, getdate()), ':', '-'), 'AM', 'am'), 'PM' ,'pm'), ' ', ''))
set @backup_all_databases = ''
select @backup_all_databases = @backup_all_databases + 
'backup database [' + name + '] to disk = ''' + @complete_path + '\' + @dayname + ' ' + @timestamp + ' ' + @type + ' ' + name + '.bak' + ''';' + CHAR(10)
from sys.databases where name not in ('tempdb') and state_desc = 'online'
exec (@backup_all_databases)
go
 
-- create backup folder structure
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- delete old full backups
declare @olderthan datetime
declare @deletepath varchar(255)
set @deletepath = @complete_path + '\'
set @olderthan = dateadd(day, -0, getdate());
execute master.dbo.xp_delete_file 0, @deletepath, 'bak', @OlderThan;
go
 
-- create backup folder structure
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- transaction log backups to network share
use master;
set nocount on
set quoted_identifier off
declare @backup_all_logs varchar(max)
declare @dayname varchar(255)
declare @timestamp varchar(255)
declare @type varchar(255)
set @type = 'TLOG'
set @dayname = ( select DATENAME(dw, GETDATE()) )
set @timestamp = ( select replace(replace(replace(REPLACE(CONVERT(char, getdate()), ':', '-'), 'AM', 'am'), 'PM' ,'pm'), ' ', ''))
set @backup_all_logs = ''
select @backup_all_logs = @backup_all_logs + 
'backup log [' + name + '] to disk = ''' + @complete_path + '\' + @dayname + ' ' + @timestamp + ' ' + @type + ' ' + name + '.trn' + ''';' + CHAR(10)
from sys.databases where name not in ('master', 'model', 'msdb', 'tempdb') and state_desc = 'online' and recovery_model_desc = 'FULL'
exec (@backup_all_logs)
go
 
-- shrink transaction log backups
declare @shrink_files varchar(max)
set @shrink_files = ''
select @shrink_files = @shrink_files +
'use [' + name + ']; dbcc shrinkfile(2,8);' + char(10)
from sys.databases where name not in ('tempdb') and state_desc = 'online' and recovery_model_desc = 'FULL'
exec (@shrink_files)
 
-- create backup folder structure
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- delete old transaction log backups
declare @olderthan datetime
declare @deletepath varchar(255)
set @deletepath = @complete_path + '\'
set @olderthan = dateadd(day, -0, getdate());
execute master.dbo.xp_delete_file 0, @deletepath, 'trn', @OlderThan;
go

```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)


      
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

