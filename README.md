![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 查看SQL.bak文件，例如Sys.Master_Files
#### View SQL .bak Files Like Sys.Master_Files
**发布-日期: 2016年10月13日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
这是一些采用文件夹路径（填充.bak文件）的SQL逻辑，并生成类似于`sys.master_files`的结果集。
那么这是做什么的呢？这将创建一些变量/临时表，然后编译所有数据库名称，逻辑名称和物理名称（路径）的列表，包括所有数据文件名* .mdf，* .ldf等。

它基本上是根据指定路径下找到的备份文件构建列表。该脚本将假设备份文件只是以自己的数据库命名，这是一种非常简化的约定，但如果你认为需要，则会相应地进行更改。然后，它将创建一个包含数据库名称（从文件名中推断）的“更好的filelistonly”表，并为你提供以下内容：

- 数据库名称
- 备份路径
- 备份文件
- 文件类型

然后，你可以根据所找到的文件类型生成脚本。



## English
Here’s some SQL logic that takes a folder path (filled with .bak files), and produces a result set similar to `sys.master_files`.
Ok; so what does this do. This creates a few variable/temp tables then compiles a list of all database names, logical names, and physical names(paths) including all data file names .mdf, .ldf, etc.

It essentially builds the list based on the backup files found under the given path. The script will assume backup files are simply named after the database it’s self which is an extremely simplified convention, but change accordingly if you see the need. It then goes about creating a ‘better filelistonly’ table incorporating the database names (extrapolated from the file names), and presents you with the following:

- Database Name
- Backup Path
- Backup File
- Type of file

You can then produce a script in accordance with the type of file that is located. 


---
## Logic
```SQL
use master;
set nocount on
 
declare @path       varchar(255)    = '\\MyBackupServerName\E$\MyBackupFolder'
declare @filelist   table
(
    [subdirectory]  varchar(255)
,   [depth]     int
,   [file]      int
)
insert  into        @filelist
exec master..xp_dirtree @path, 1, 1
 
if object_id('tempdb..#backupfiles') is not null
        drop table  #backupfiles
create  table       #backupfiles
(
    [id]        int identity(1,1)
,   [database]  varchar(255)
,   [backup_path]   varchar(255)
,   [backup_file]   varchar(255)
)
 
insert  into #backupfiles ([database], [backup_path], [backup_file])
select  upper(replace([subdirectory], '.bak', '')), @path, [subdirectory]
from    @filelist where [subdirectory] not in ('master.bak', 'msdb.bak', 'model.bak')
 
select * from #backupfiles
 
if object_id('tempdb..#better_filelistonly') is not null
    drop table  #better_filelistonly
create  table   #better_filelistonly
(
    [id]            int identity(1,1)
,   [database]      varchar(255)
,   LogicalName     nvarchar(128)
,   PhysicalName        nvarchar(260)
,   [Type]          char(1)
,   FileGroupName       nvarchar(128)
,   Size            numeric(20,0)
,   MaxSize         numeric(20,0)
,   FileID          bigint
,   CreateLSN       numeric(25,0)
,   DropLSN         numeric(25,0)
,   UniqueID        uniqueidentifier
,   ReadOnlyLSN     numeric(25,0)
,   ReadWriteLSN        numeric(25,0)
,   BackupSizeInBytes   bigint
,   SourceBlockSize     int
,   FileGroupID     int
,   LogGroupGUID        uniqueidentifier
,   DifferentialBaseLSN numeric(25,0)
,   DifferentialBaseGUID    uniqueidentifier
,   IsReadOnly      bit
,   IsPresent       bit
,   TDEThumbprint       varbinary(32) 
)
 
declare @get_logical_names  varchar(max)
set @get_logical_names  =''
select  @get_logical_names  =@get_logical_names +
'declare @baseid    int =(select isnull(max(id), 0) from #better_filelistonly)' + char(10) +
'insert into #better_filelistonly' + char(10) +
'(
    LogicalName
,   PhysicalName
,   [Type]
,   FileGroupName
,   Size
,   MaxSize
,   FileID
,   CreateLSN
,   DropLSN
,   UniqueID
,   ReadOnlyLSN
,   ReadWriteLSN
,   BackupSizeInBytes
,   SourceBlockSize
,   FileGroupID
,   LogGroupGUID
,   DifferentialBaseLSN
,   DifferentialBaseGUID
,   IsReadOnly
,   IsPresent
,   TDEThumbprint
)'  + char(10) + 
 
'exec (''restore filelistonly from disk = ''''' + [backup_path] + [backup_file] + ''''''');' + char(10) +
'declare    @lastid int =   (select (max(id) + 1) from #better_filelistonly)' + char(10) +
'update     #better_filelistonly    set [database] = ''' + [database] + ''' where [id] between @baseid and @lastid' + char(10) + 'go' + char(10) + char(10)
from    #backupfiles
 
exec    (@get_logical_names)
 
select * from #better_filelistonly


```



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

