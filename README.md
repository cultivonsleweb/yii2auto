# yii2auto
Script installation, automatic update Yii2 and its extensions.

Yii2Auto use Phing (https://www.phing.info/)

## requierment

Composer (https://getcomposer.org/) must be installed. 

## install

Clone project.

Edit `phingyii2/build.properties` and customize your data

run :

`composer update`

phing is then installed.
The populated database must be created before running phing.
Now run phing:

`phing`

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
    
        <phing 
        phingfile="${ext.path}/dektrium/yii2-user.xml" 
        target="dektrium/yii2-user" />
        
    </target>
    
</project>
```

## LICENSE 

GNU GENERAL PUBLIC LICENSE Version 3, please read LICENSE.md
