#m2s3
###mongodump straight to Amazon S3


*m2s3* is a small program who performs a number of **mongodb database backups via mongodump**, compresses them into a gzipped tarball and finally sends them to an **AWS S3 bucket**.    
       
This program was made as part of [Blanclink](http://www.blanclink.com)'s IT Infrastructure strategy to preserve essential data used by [Blancride](http://www.blancride.com)  and other applications developed by the company.

##Requirements
* [python](http://python.org) >= 3.3
* [boto](http://docs.pythonboto.org/en/latest/) >= 2.33
* mongodump >= 2.4

##Usage

    m2s3 [options]

##Options
* `--version` show version number and exit
* `-h | --help` show a help message and exit
* `-c [file] | --config=[file] | --config [file]` specify configuration file to use
* `-d | --dry-run` don't actually do anything
* `-s | --stdout` log messages to stdout too
* `--ll | --log-level=[num]` set logging output level

#Installation
Once the source distribution has been downloaded, installation can be made via **setuptools** or **pip**, whichever you prefer.

	$ # setuptools installation
	$ cd blancride-is-m2s3
	$ python setup.py install
	$ # from this point, you can create your configuration file
	$ cat > /path/to/my/config.conf
	$ {
	$	"aws": {
	$		"aws_id" : "SDF73HSDF3663KSKDJ",
	$		"aws_access_key" : "d577273ff885c3f84dadb8578bb41399"
	$	},
	$	"mongodb": {
	$		"hosts": [
	$			{
	$				"address": "myserver.com",
	$				"user_name": "bob",
	$				"password": "robert",
	$				"dbs" : ['clients', 'seo_stats']
	$			}
	$		]
	$	}
	$ }
	$ # Once installed, you can try it
	$ m2s3 -c /path/to/myconfig.conf

If everything went well, you can then check out your S3 bucket to see the backup.

##Configuration file
The configuration is handled through a simple [JSON](http://www.json.org) file including a series of *sections* (which are JSON objects), each one composed by *directives* (JSON numbers, strings and arrays) which determine the behavior on **m2s3**.  If **m2s3** does not receive any configuration file on command line, it will try to read `/etc/m2s3.conf`.  
           
The following is an example of what a configuration file looks like:

    $ cat /etc/m2s3.conf
    {
      "debug": true,
      "log": {
      },
      "aws": {
        "aws_id": "SDF73HSDF3663KSKDJ",
        "aws_access_key": "d577273ff885c3f84dadb8578bb41399"
      },
      "mongodb": {
		"mongodump" : "/opt/bin/mongodump",
		"output_dir" : "/opt/tmp/mydir",
		"host_defaults" : {
			"port" : 666,
			"user_name" : "satan",
			"password" : "14mh4x0r",
		},
		"hosts": [
			{
				"port": 34127,
				"dbs": ["app", "sessions", "another_one"]
			},
			{
				"name" : "bar",
				"address" : "bar.example.com",
				"password" : "1AmAn07h3rh4x0r"
			}
		]
      }
    }
Through this configuration file, you can set key variables about the databases you want to backup and the AWS S3 bucket you wish to send them to.
###Configuration file: sections and directives
####Root section
#####Directives
    "debug" : true | false
* Type: **boolean**
* *Default value: false*
* **Role: Debug mode is activated if `true`**

####`log` section
Directives regarding logging output

#####Directives
*FOR THE MOMENT RESERVED*
 
####`mongodb` section
This section holds directives regarding the [**mongodb**](http://mongodb.org) server where **`m2s3`** is going to connect to and also the databases that are going to be backed up through *mongodump*.

***
    "output_dir" : <directory>
* Type: **string**
* *Default value : /tmp/m2s3*
* **Role: directory where m2s3 is going to temporarily save the backup files**
* **Examples:** 
>`"output_dir": "/path/to/my/dir"`

***
    "mongodump" : <path_to_executable>
* Type: **string**
* *Default value : "mongodump"*
* **Role: mongodump executable used by m2s3**
* **Examples:** 
>`"mongodump": "/opt/bin/mongodump"`

####`host_defaults` subsection
Many directives (such as the user name and/or password) could be common among the databases that are going to be backed up. For this reason, it is best to simply put those common directives under a single section, this is entirely optional but the best for easily manageable configuration files in order to avoid redundance, the supported directives are `user_name`, `password`, `dbs` and `port` . See **`hosts`** section.

####`hosts` section

This is an array of objects, each containing the following a series of directives relative to a mongodb database located at a server, its specs and databases themselves contained within it, these are the main values used by `mongodump` when it does its magic. For each entry inside the `hosts` section, these are its valid directives:

#####Directives
***
    "address" : <hostname> | <fqdn> | <ip_address> 
* Type: **string**
* Required: YES
* Default value : "localhost"
* **Role: mongodb server location**
* **Examples:**
>With a hostname : `"host": "my-hostname"`
>With a FQDN: `"host": "my.host.name"`
>With an IPv4 address: `"host": "192.168.1.1"`

***
    "port" : <number>
* Type: **integer**
* Required: NO
* Default value : `host_defaults["port"]` | 27017
* **Role: mongodb server listening port**
* **Examples:**
>`"port": 27412`

***
    "user_name" : <user>
* Type: **string**
* Required: NO
* Default value : `host_defaults['user_name']` | "m2s3"
* **Role: user name used for authentication against the mongodb server**
* **Examples:** 
>`"user_name": "matt"`

***
    "password" : <password>
* Type: **string**
* Required: NO
* Default value : `host_defaults['pass']` |  "pass"
* **Role: password used for authentication against the mongodb server**
* **Examples:**
>`"password": "mySup3rS3cr37P455w0rd"`

***
	"dbs" : <array>
* Type: **array**
* Required: NO
* Default value : `host_defaults['dbs']` | []
* **Role: password used for authentication against the mongodb server**
* **Examples:**
>`"dbs": ["jack", "wendy", "danny"]`

**NOTE: particular `dbs` on one host will be merged with those of `host_defaults`**
####`aws` section
This sections holds directives regarding AWS credentials that **`m2s3`** is going to use in order to upload the *mongodump backups* to S3.
#####Directives
***
    "aws_id" : <id>
* Type: **string**
* Required: NO
* *Default value : ""*
* **Role: AWS access key ID**
* **Examples:**
>`"aws_id": "HAS6NBASD8787SD"`

***
    "aws_access_key" : <key>
* Type: **string**
* Required: NO
* *Default value : ""*
* **Role: AWS access key ID**
* **Examples:**
>`"aws_access_key": "d41d8cd98f00b204e9800998ecf8427e"`

***
	"s3_bucket"  : <bucket_name>
* Type: **string**
* Required: NO
* *Default value: "m2s3"*
* **Role: name of the S3 bucket where m2s3 is going to upload the compressed backup**
* **Examples:**
> `"s3_bucket" : "mybucket"`
