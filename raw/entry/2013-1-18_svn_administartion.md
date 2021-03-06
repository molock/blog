#Subversion Administration 
##Administrator's Toolkit
安装Subversion后，查看下svn提供的实用工具：

	[root@server ~]# svn [Tab][Tab]
	svn   svnadmin  svndumpfilter  svnlook   svnserve    svnsync   svnversion    

简介：

* svn：命令行客户端程序
* svnversion：查看当前项目的修订版本的工具
* svnadmin：Subversion版本库管理程序，可以建立、调整和修复版本库等
* svndumpfilter：过滤Subversion版本库转储数据流的工具
* svnlook：直接查看版本库的工具
* svnserve：一个单独运行的服务器程序，可以作为守护进程或者由SSH调用
* svnsync：一个通过网络增量镜像版本库的程序

这里参考《Version Control with SUbversion》的“Chapter 5. Repository Administration”进行整理，并进行简单的测试，主要面向fsfs类型的版本库的备份和恢复；有Subversion的概念、组织、授权等请参考本书其他章节。

##1. svn
svn is the official command-line client of Subversion. Its functionality is offered via a collection of task-specific subcommands, most of which accept a number of options for fine-grained control of the program's behavior.  

##2. svnversion
svnversion is a program for summarizing the revision mixture of a working copy. The resultant revision number, or revision range, is written to standard output.

##3. svnadmin
The svnadmin program is the repository administrator's best friend. Besides providing the ability to create Subversion repositories, this program allows you to perform several maintenance operations on those repositories.

##4. svnserve
svnserve allows access to Subversion repositories using Subversion's custom network protocol.

You can run svnserve as a standalone server process (for clients that are using the `svn://` access method); you can have a daemon such as inetd or xinetd launch it for you on demand (also for `svn://`), or you can have sshd launch it on demand for the `svn+ssh://` access method.

##5. svnlook
svnlook is a tool provided by Subversion for examining the various revisions and transactions (which are revisions in the making) in a repository. No part of this program attempts to change the repository. svnlook is typically used by the repository hooks for re- porting the changes that are about to be committed (in the case of the pre-commit hook) or that were just committed (in the case of the post-commit hook) to the repository.  

#6. svndumpfilter
While it won't be the most commonly used tool at the administrator's disposal, svndumpfilter provides a very particular brand of useful functionality—the ability to quickly and easily modify streams of Subversion repository history data by acting as a path- based filter.

#7. svnsync
The svnsync program provides all the functionality required for maintaining a read-only mirror of a Subversion repository. The program really has one job—to transfer one repository's versioned history into another repository. And while there are few ways to do that, its primary strength is that it can operate remotely—the “source” and “sink”6 repositories may be on different computers from each other and from svnsync itself.

svnsync is the Subversion remote repository mirroring tool. Put simply, it allows you to replay the revisions of one repository intoanother one.

In any mirroring scenario, there are two repositories: the source repository, and the mirror (or “sink”) repository. The source repository is the repository from which svnsync pulls revisions. The mirror repository is the destination for the revisions pulled from the source repository. Each of the repositories may be local or remote—they are only ever addressed by their URLs.

The svnsync process requires only read access to the source repository; it never attempts to modify it. But obviously, svnsync requires both read and write access to the mirror repository.

#Migration Repositories
##1. svnadmin dump
	[root@server repository]# svnadmin help dump
	dump: usage: svnadmin dump REPOS_PATH [-r LOWER[:UPPER] [--incremental]]
	
	Dump the contents of filesystem to stdout in a 'dumpfile'
	portable format, sending feedback to stderr.  Dump revisions
	LOWER rev through UPPER rev.  If no revisions are given, dump all
	revision trees.  If only LOWER is given, dump that one revision tree.
	If --incremental is passed, the first revision dumped will describe
	only the paths changed in that revision; otherwise it will describe
	every path present in the repository as of that revision.  (In either
	case, the second and subsequent revisions, if any, describe only paths
	changed in those revisions.)
	
	Valid options:
	  -r [--revision] ARG      : specify revision number ARG (or X:Y range)
	  --incremental            : dump incrementally
	  --deltas                 : use deltas in dump output
	  -q [--quiet]             : no progress (only errors) to stderr

###Examples

转储存整个版本库：

	[root@server ~]# svnadmin dump /data/repository/Test > /data/backup/Test_full.dump
	* Dumped revision 0.
	* Dumped revision 1.
	* Dumped revision 2.
	* Dumped revision 3.
	* Dumped revision 4.

基于修订版本进行增量转储：

	[root@server ~]# svnadmin dump /data/repository/Test --incremental > /data/backup/Test_base.dump
	* Dumped revision 0.
	* Dumped revision 1.
	* Dumped revision 2.
	* Dumped revision 3.
	* Dumped revision 4.
	* Dumped revision 5.
	* Dumped revision 6.
	* Dumped revision 7.
	* Dumped revision 8.

检出版本库：

	[root@server Workspace]# pwd
	/root/Workspace
	[root@server Workspace]# svn checkout --username 'dylanninin@gmail.com' --password 'Us*+2&:9L/C,8^.?' http://172.29.88.162/Test .

	[root@server Workspace]# ls
	DBA  Notes  svn  Test

提交更新：

	[root@server Workspace]# mkdir Incremental
	[root@server Workspace]# cd Incremental/
	[root@server Incremental]# ls
	[root@server Incremental]# touch 1
	[root@server Incremental]# touch 2
	[root@server Incremental]# cd ..
	[root@server Workspace]# svn add Incremental/
	A         Incremental
	A         Incremental/1
	A         Incremental/2
	[root@server Workspace]# svn commit -m "add incremental"
	Adding         Incremental
	Adding         Incremental/1
	Adding         Incremental/2


增量转存：

	[root@server Workspace]# svnlook history /data/repository/Test / --show-ids
	REVISION   PATH <ID>
	--------   ---------
	       9   / <0.0.r9/713>
	       8   / <0.0.r8/4674>
	       7   / <0.0.r7/710>
	       6   / <0.0.r6/112>
	       5   / <0.0.r5/201>
	       4   / <0.0.r4/174>
	       3   / <0.0.r3/144851702>
	       2   / <0.0.r2/138>
	       1   / <0.0.r1/106>
	       0   / <0.0.r0/17>

	[root@server ~]# svnadmin dump /data/repository/Test -r 9 --incremental > /data/backup/Test_r9.dump
	* Dumped revision 9.

导入初始增量：

	[root@server ~]# svnadmin create /data/repository/Inc
	[root@server ~]# svnadmin load /data/repository/Inc/ < /data/backup/Test_base.dump 

查看目录结构：	

	[root@server ~]# svnlook tree /data/repository/Inc/ --full-paths
	/
	Test/
	DBA/
	DBA/LaTeX and WinEdit Tutorial.pdf
	... ...
	svn/
	svn/svn.png
	svn/svn.dot
	Notes/
		
导入版本为9的增量：

	[root@server ~]# svnadmin load /data/repository/Inc/ < /data/backup/Test_r9.dump 
	<<< Started new transaction, based on original revision 9
	     * adding path : Incremental ... done.
	     * adding path : Incremental/1 ... done.
	     * adding path : Incremental/2 ... done.
	
	------- Committed revision 9 >>>

查看目录结构：

	[root@server ~]# svnlook tree /data/repository/Inc/ --full-paths
	/
	Test/
	Incremental/
	Incremental/1
	Incremental/2
	DBA/
	DBA/LaTeX and WinEdit Tutorial.pdf
	... ...
	svn/
	svn/svn.png
	svn/svn.dot
	Notes/

注：

查看版本库的目录结构，：

	[root@server ~]# svnlook tree /data/repository/Test --full-paths --show-ids
	/ <0.0.r4/174>
	Test/ <0-4.0.r4/0>
	DBA/ <0-1.0.r3/144851472>
	DBA/LaTeX and WinEdit Tutorial.pdf <1-3.0.r3/144826285>
	... ...


以相应目录的修订版本历史：

	[root@server ~]# svnlook history /data/repository/Test / --show-ids
	REVISION   PATH <ID>
	--------   ---------
	       4   / <0.0.r4/174>
	       3   / <0.0.r3/144851702>
	       2   / <0.0.r2/138>
	       1   / <0.0.r1/106>
	       0   / <0.0.r0/17>
	[root@server ~]# 


##2. svnadmin load

	[root@server ~]# svnadmin help load
	load: usage: svnadmin load REPOS_PATH

	Read a 'dumpfile'-formatted stream from stdin, committing
	new revisions into the repository's filesystem.  If the repository
	was previously empty, its UUID will, by default, be changed to the
	one specified in the stream.  Progress feedback is sent to stdout.
	
	Valid options:
	  -q [--quiet]             : no progress (only errors) to stderr
	  --ignore-uuid            : ignore any repos UUID found in the stream
	  --force-uuid             : set repos UUID to that found in stream, if any
	  --use-pre-commit-hook    : call pre-commit hook before committing revisions
	  --use-post-commit-hook   : call post-commit hook after committing revisions
	  --parent-dir ARG         : load at specified directory in repository

###Examples

导入转储的版本库：

	[root@server ~]# svnadmin load /data/repository/Test_load < /data/backup/Test_full.dump 
	svnadmin: Can't open file '/data/repository/Test_load/format': No such file or directory

需先创建目标版本库：

	[root@server ~]# svnadmin create /data/repository/Test_load
	[root@server ~]# svnadmin load /data/repository/Test_load < /data/backup/Test_full.dump 
	<<< Started new transaction, based on original revision 1
	     * adding path : DBA ... done.
	
	------- Committed revision 1 >>>
	
	<<< Started new transaction, based on original revision 2
	     * adding path : Notes ... done.
	
	------- Committed revision 2 >>>
	
	<<< Started new transaction, based on original revision 3
	     * adding path : DBA/CTeX FAQ.pdf ... done.
	     ... ...
	     * adding path : DBA/template ... done.
	     * adding path : DBA/template/CASthesis-v0.1j.zip ... done.
	     * adding path : DBA/template/CASthesis-v0.2.zip ... done.
	
	------- Committed revision 3 >>>
	
	<<< Started new transaction, based on original revision 4
	     * adding path : Test ... done.
	
	------- Committed revision 4 >>>


##3. svndumpfilter

	[root@server ~]# svndumpfilter help
	general usage: svndumpfilter SUBCOMMAND [ARGS & OPTIONS ...]

	Type 'svndumpfilter help <subcommand>' for help on a specific subcommand.
	Type 'svndumpfilter --version' to see the program version.

	Available subcommands:
	   exclude
	   include
	   help (?, h)

###Examples

使用svndumpfilter对转储文件进行过滤，`include`模式，其他的都排除：

	[root@server repository]# svndumpfilter include "DBA/template" "DBA/pdf" < /data/backup/Test_full.dump > Test_filter.dump
	Including prefixes:
	   '/DBA/template'
	   '/DBA/pdf'
	
	Revision 0 committed as 0.
	Revision 1 committed as 1.
	Revision 2 committed as 2.
	Revision 3 committed as 3.
	Revision 4 committed as 4.
	
	Dropped 72 nodes:
	   '/DBA'
	   '/DBA/CTeX FAQ.pdf'
	   '/DBA/LaTeX and WinEdit Tutorial.pdf'
	   '/DBA/LaTeX2ε 插图指南.pdf'
	   ... ...

`--pattern`模式，使用正则表达式：

	[root@server ~]# svndumpfilter exclude --pattern "*template" < /data/backup/Test_full.dump > Test_filter.dump
	svndumpfilter: invalid option: --pattern
	Type 'svndumpfilter help' for usage.

Note: 

Beginning in Subversion 1.7, `svndumpfilter` can optionally treat the PATH_PREFIXs not merely as explicit substrings, but as file patterns instead。

也就是说，在1.7以上版本的subversion中，才能使用基于正则表达式的路径语法；在低版本中，使用正则表达式语法，只会被当做普通的字符串处理。

查看当前subversion版本信息：

	[root@server ~]# svnadmin --version
	svnadmin, version 1.6.11 (r934486)
	   compiled Sep 27 2011, 15:29:25
	Copyright (C) 2000-2009 CollabNet.

	Subversion is open source software, see http://subversion.tigris.org/
	This product includes software developed by CollabNet (http://www.Collab.Net/).

	The following repository back-end (FS) modules are available:
	* fs_base : Module for working with a Berkeley DB repository.
	* fs_fs : Module for working with a plain file (FSFS) repository.

##4. svnadmin hotcopy

	[root@server repository]# svnadmin help hotcopy
	hotcopy: usage: svnadmin hotcopy REPOS_PATH NEW_REPOS_PATH
	Makes a hot copy of a repository.
	Valid options:
	  --clean-logs             : remove redundant Berkeley DB log files
	                             from source repository [Berkeley DB]

###Examples

热拷贝一个版本库：

	[root@server ~]# svnadmin hotcopy /data/repository/Test /data/repository/Test_hotcopy

使用svnlook tree输出源版本库、新版本库的目录结构，并用diff比较

	[root@server ~]# svnlook tree /data/repository/Test_hotcopy/ > Test_hotcopy.tree
	[root@server ~]# svnlook tree /data/repository/Test/ > Test.tree
	[root@server ~]# diff Test_hotcopy.tree Test.tree 

新版本库路径加到Apache的`subversion.conf`中，调整authz配置即可正常访问。

##5. svnsync

	[root@server ~]# svnsync help
	general usage: svnsync SUBCOMMAND DEST_URL  [ARGS & OPTIONS ...]
	Type 'svnsync help <subcommand>' for help on a specific subcommand.
	Type 'svnsync --version' to see the program version and RA modules.
	
	Available subcommands:
	   initialize (init)
	   synchronize (sync)
	   copy-revprops
	   info
	   help (?, h)

###Examples

	[root@server ~]# svnsync init file:///data/repository/Test_mirror file:///data/repository/Test
	svnsync: Unable to open an ra_local session to URL
	svnsync: Unable to open repository 'file:///data/repository/Test_mirror'

	[root@server ~]# svnadmin create /data/repository/Test_mirror

	[root@server ~]# svnsync init file:///data/repository/Test_mirror file:///data/repository/Test
	svnsync: Repository has not been enabled to accept revision propchanges;
	ask the administrator to create a pre-revprop-change hook

	[root@server ~]# cd /data/repository/Test_mirror/hooks
	[root@server hooks]# cp pre-revprop-change.tmpl pre-revprop-change

	[root@server ~]# svnsync init file:///data/repository/Test_mirror file:///data/repository/Test
	svnsync: Revprop change blocked by pre-revprop-change hook (exit code 255) with no output.
	
	[root@server ~]# chmod +x /data/repository/Test_mirror/hooks/pre-revprop-change
	[root@server ~]# svnsync init file:///data/repository/Test_mirror file:///data/repository/Test
	svnsync: Revprop change blocked by pre-revprop-change hook (exit code 1) with output:
	Changing revision properties other than svn:log is prohibited

有待继续测试... ...

##6.svnadmin upgrade

	[root@server repository]# svnadmin help upgrade
	upgrade: usage: svnadmin upgrade REPOS_PATH
	
	Upgrade the repository located at REPOS_PATH to the latest supported
	schema version.
	
	This functionality is provided as a convenience for repository
	administrators who wish to make use of new Subversion functionality
	without having to undertake a potentially costly full repository dump
	and load operation.  As such, the upgrade performs only the minimum
	amount of work needed to accomplish this while still maintaining the
	integrity of the repository.  It does not guarantee the most optimized
	repository state as a dump and subsequent load would.


##7. svnadmin verify

	[root@server ~]# svnadmin help verify
	verify: usage: svnadmin verify REPOS_PATH

	Verifies the data stored in the repository.

	Valid options:
	  -r [--revision] ARG      : specify revision number ARG (or X:Y range)
	  -q [--quiet]             : no progress (only errors) to stderr


#Sumary

Backup or migrate a Whole Repository：

* hotcopy

	`svnadmin hotcopy src_repo_path dist_repo_path`

* dump with increment

	`svnadmin dump -> dump file --> svndumpfilter --> filtered dump file --> svnadmin load`

* sync

	`svnsync`

![](file:///F:\Workspace\lab\g\svn_backup_and_recovery.png)


#Reference
* Version Control with Subversion
