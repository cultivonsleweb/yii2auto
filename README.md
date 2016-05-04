# yii2auto
Script installation, automatic update Yii2 and its extensions.

Yii2Auto use Phing (https://www.phing.info/)

## requierment

Composer (https://getcomposer.org/) must be installed. 

## install

1. Clone project.

2. Edit `phingyii2/build.properties` and customize your data

3. run :

```sh

$ composer update

```

phing is then installed.
The populated database must be created before running phing.
4. Now run phing:

```sh 

$ phing

```

or 

````

$ vendor/bin/phing

```

## build.properties

File default :

```
# PROJECT PROPERTIES

# WARNING : DON'T CHANGE basedir
basedir= . 

# Project name
project=Yii2Auto

# Action : install or update
#	   After installation, set to 'install' automatically becomes 'update'
action=install

# general paths
build=${basedir}/build
tmp=${basedir}/tmp
vendor=vendor
yii=${basedir}/yii
php=/opt/local/bin/php
composer=/opt/local/bin/composer
ext.path=${basedir}/phingyii2/ext

# Set the template project structure: 'basic' or 'advanced'
yii2.template=advanced

# 1/ basic
# Define the data template for the 'basic'.
yii2.basic.db.host=localhost
yii2.basic.db.user=root
yii2.basic.db.pwd=root
yii2.basic.db.database=yii2auto_basic

# 2/ advanced
# common database, set to 'true'. Otherwise 'false'
yii2.advanced.db.common=true

# if common is 'true', define the variables 'yii2.advanced.db.common. *'.
yii2.advanced.db.common.host=localhost
yii2.advanced.db.common.user=root
yii2.advanced.db.common.pwd=root
yii2.advanced.db.common.database=yii2auto_advanced_common

# if common is false, define the variables 'yii2.advanced.db.backend. *' and 'yii2.advanced.db.frontend. *'.
yii2.advanced.db.backend.host=localhost
yii2.advanced.db.backend.user=root
yii2.advanced.db.backend.pwd=root
yii2.advanced.db.backend.database=yii2auto_advanced_backend

yii2.advanced.db.frontend.host=localhost
yii2.advanced.db.frontend.user=root
yii2.advanced.db.frontend.pwd=root
yii2.advanced.db.frontend.database=yii2auto_advanced_frontend
```

Change `project` by your project name

Change `general paths` for `composer` and `php` by yours paths.

Choose between 'basic' and 'advanced' to `yii2.template`

If you choose `basic`, changes the values of the following lines:

```
yii2.basic.db.host=localhost
yii2.basic.db.user=root
yii2.basic.db.pwd=root
yii2.basic.db.database=yii2auto_basic
```

If you choose `advanced`, you must set if you use the same database for the frontend and the backend. If so, then the variable définissiez `yii2.advanced.db.common` `true`, if not `false`.

If `yii2.advanced.db.commontrue` = `true`, define: 

```
yii2.advanced.db.common.host=localhost
yii2.advanced.db.common.user=root
yii2.advanced.db.common.pwd=root
yii2.advanced.db.common.database=yii2auto_advanced_common
```

if not, define:

```
yii2.advanced.db.backend.host=localhost
yii2.advanced.db.backend.user=root
yii2.advanced.db.backend.pwd=root
yii2.advanced.db.backend.database=yii2auto_advanced_backend

yii2.advanced.db.frontend.host=localhost
yii2.advanced.db.frontend.user=root
yii2.advanced.db.frontend.pwd=root
yii2.advanced.db.frontend.database=yii2auto_advanced_frontend
```

## Add extension

For that extensions can be installed with auto yii2, create a file and add phing its appeal in ext.xml.

### 1. phing create the file extension

Create directory in `phingyii2/ext` with your namespace, example `dektrium` and create a xml file with name of your extension, example : `yii2-user.xml`.

Start your file with :

```xml

<?xml version="1.0" encoding="utf-8"?>
<project basedir="${basedir}" default="namspace/name-extension">

    <property file="${basedir}/phingyii2/build.properties" />
    
``` 

and create tag `target` with attribut `name` equal to `namespace/name-extension`.

In this tag, add your command for install.

example :

```xml

<?xml version="1.0" encoding="utf-8"?>
<project basedir="${basedir}" default="dektrium/yii2-user">

    <property file="${basedir}/phingyii2/build.properties" />
    
    <target name="dektrium/yii2-user">
        <echo>install dektrium/yii2-user extension...</echo>
        
        <echo msg="Update configuration"></echo>
        <copy file="${basedir}/common/config/main-local.php"
                tofile="${src}/common/config/main-local.php.dist" overwrite="true" />
                
        <copy file="${src}/common/config/main-local.php.dist" tofile="${basedir}/common/config/main-local.php" overwrite="true">
            <filterchain>
            <replacetokens begintoken="'" endtoken="'">
                    <token key="components" value="'modules' => [  
    'user' => [ 
        'class' => 'dektrium\user\Module',  
    ],  
],  
'components'" />
                </replacetokens>
             </filterchain>
        </copy>
        
        <echo>composer require "dektrium/yii2-user:0.9.*@dev"</echo>
        <composer command="require" composer="${composer}">
            <arg value="dektrium/yii2-user:0.9.*@dev" />
        </composer>
        
        <echo>composer update</echo>
        <composer command="update" composer="${composer}" />
        
        <echo msg="Migration"></echo>
        <exec command="${php} ${yii} migrate/up --migrationPath=${vendor}/dektrium/yii2-user/migrations --interactive=0" dir="${basedir}/"  />
        <echo>${php} ${yii} migrate/up --migrationPath=${vendor}/dektrium/yii2-user/migrations --interactive=0</echo>
        
        <echo>...dektrium/yii2-user installed</echo>
    </target>
    
</project>

```

### 2. Call the phing file extension

Edit `phingyii2/ext.xml` and added to the target tag and whose value of the attribute "name", "run":

```xml
<phing 
        phingfile="${ext.path}/namespace/phing-file-extension.xml" 
        target="namespace/name-extension" />
``` 

example :

```xml
<?xml version="1.0" encoding="utf-8"?>
<project basedir="${basedir}" default="install">

    <property file="${basedir}/phingyii2/build.properties" />
  
    <target name="run">
    
        <echo>extensions, install,update..start</echo>
        
        <phing 
        phingfile="${ext.path}/dektrium/yii2-user.xml" 
        target="dektrium/yii2-user" />
        
        <echo>extensions, install,update..end</echo>
        
        <echo>composer update</echo>
        <composer command="update" composer="${composer}" />
        
    </target>
    
</project>
```

## LICENSE 

GNU GENERAL PUBLIC LICENSE Version 3, please read LICENSE file.

