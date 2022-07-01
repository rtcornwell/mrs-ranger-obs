overview
![image](https://user-images.githubusercontent.com/16845371/176891104-5151b6c9-7b5c-4d8e-9d0e-e99fb605f4c9.png)

1.ranger-obs-plugin: Provides a service definition plugin on the Ranger server. It provides OBS service permission control on the Ranger side; After the plug-in is deployed, users can fill in the appropriate permissions policy on the Control page of Ranger.

2. ranger-obs-service: The service provides an RPC interface that verifies permissions locally after receiving an authentication request from hadoop-obs/ranger-obs-client; It periodically synchronizes permission policies from the Ranger server

3.ranger-obs-client: Hadoop-obs integrates this plugin to forward requests that require permission validation to the ranger-obs-service.

Source code compilation
1. Make the ranger-obs directory

2.mvn clean package -Dmaven.test.skip=true

3. Generate the following components

ranger-obs/ranger-obs-client/target/ranger-obs-client-0.1.0.jar

ranger-obs/ranger-obs-plugin/target/ranger-obs-plugin-0.1.0.tar.gz

ranger-obs/ranger-obs-service/target/ranger-obs-service-0.1.0.tar.gz

II. .ranger-obs-plugin installation
(1) Unzip and extract the ranger-obs-plugin-0.1.0 .tar.gz, including the following:

ranger-obs-plugin-0.1.0.jar: ranger obs plugin package

ranger-obs.json: configuration file

(2) Put in the rangeradmin directory

Place the ranger-obs-plugin-0.1.0 .jar in the /ranger-admin/ews/webapp/WEB-INF/classes/ranger-plugins/obs directory

Note the permissions of the users and user groups of the obs directory and the ranger-obs-plugin-0.1.0-SNAPSHOT .jar

(3) Restart the Ranger service

(4) Register the OBS service on Ranger

curl -v -k -u<rangeradmin>:<password> -X POST -H "Accept:application/json" -H "Content-Type:application/json" -d @./ranger-obs.json https://ranger/service/plugins/definitions

(5) Create an obs service in the following directory

![image](https://user-images.githubusercontent.com/16845371/176890620-28513dd6-834d-4b5f-8a89-e8dd587acd04.png)

 ![image](https://user-images.githubusercontent.com/16845371/176890684-9957ec77-c89b-4dea-9511-08079018713c.png)
   

III. Ranger-obs-service installation
1. Add ranger-obs-service to start users

(1) Add kerberos users:

addprinc -randkey rangerobs/hadoop@NOVALOCAL

ktadd -k /etc/security/keytabs/rangerobs.keytab rangerobs/hadoop@NOVALOCAL

(2) Add the corresponding local user: useradd rangerobs -g hadoop -p rangerobs

2. Extract the ranger-obs-service-0.1.0 .tar.gz, including the following:

(1) bin: Script directory

star_rpc_server.sh: HUAWEI CLOUD MRS is integrated and used, which involves a lot of mr.-specific related information

start_server.sh: Open source big data cluster use

(2) conf: Profile directory

core-site .xml and hdfs-site .xml: Configuration files needed to access THE HDFS service (this service relies on the HDFS service)

Ranger-obs-security.xml and ranger-obs-audit .xml: access to the configuration file of the rangerAdmin service

ranger-obs.xml: The main configuration file of the ranger-obs-service service itself

Kdc.conf and rangerobs.keytab, etc.: Other optional configuration files

log4j.properties: Log configuration file

(3) lib: Dependent package directory

3. Configuration: Fill in the required options according to your own environment, and the others will remain at default values

(1) Core-site .xml and HDFS-site .xml configuration files:

You can copy it from the hadoop root /etc/hadoop/directory

Note: Configure-ranger.obs.xxx-site .xml and HDFs-site .xml configuration files for ranger-obs-service do not appear

(2) Ranger-obs .xml configuration file: required configuration items

<!--ranger-obs-service-kerberos -->
<property>
    <name>ranger.obs.service.kerberos.principal</name>
    <value>rangerobs/hadoop@NOVALOCAL</value>
</property>
<!-- ranger-obs-service-kerberos -->
<property>
    <name>ranger.obs.service.kerberos.keytab</name>
    <value>/etc/security/keytabs/rangerobs.keytab</value>
</property>
(3) ranger-obs-security.xml configuration file: ranger.plugin.obs.policy.cache.dir xxx/ranger-obs-service-0.1.0/cache ranger.plugin.obs.policy.rest.url http://rangerAdmin address: 6080 4. Launch

(1) Optional action: Edit the script start_server.sh and modify the following variables

native_path=hadoop root/lib/native/

(2) Start: nohup ./start_server.sh &

Note: The ranger-obs-service process will access the HDFS /user/directory when it starts, please give the ranger-obs-service process permission to start the user to access the HDFS /user/directory

IV. ranger-obs-client installation
1. Installation:

Place the ranger-obs-client-0.1.0-SNAPSHOT .jar and the ranger-obs-client-0.1.0-SNAPSHOT-tests .jar in the hadoop-obs directory

2. Configuration

Add the following configuration key to the core-site .xml file under the hadoop component directory:

ranger.obs.service.rpc.address: ranger-obs-service RPC service address xxxx:26900, supports configuring multiple addresses to be split by semicolons

ranger.obs.service.kerberos.principal：rangerobs/hadoop@NOVALOCAL

fs.obs.authorize.provider:org.apache.hadoop.fs.obs.security.RangerAuthorizeProvider: When this parameter is configured, the hadoop-obs module will walk the ranger authentication logic, otherwise it will not take the ranger authentication logic

User Manual
1.ranger-admin configuration

(1) Enter the obs service

![image](https://user-images.githubusercontent.com/16845371/176890933-f16c84a2-87dc-41d0-bb89-d17a291a6894.png)
    

(2) Create a policy

![image](https://user-images.githubusercontent.com/16845371/176891012-bb966e5d-3e62-4c4d-bac6-6a81c77aa581.png)

The relevant parameters are meanings as follows:

bucket: OBS bucket name
path: Object path, support wildcards, note that object paths do not start with /.
include: Indicates whether the set permission applies to path itself or to a path other than path.
recursive: Indicates that permissions apply not only to path, but also to child members under path path (i.e., recursive child members). Typically used when path is set to a directory.
Note: For bucket root directories, because the corresponding object path is an empty string, it can only be set by the * wildcard character at present, and it is recommended that you set the permissions of the root directory only for administrators

user/group: User name and user group. Here is the relationship of or, that is, the user name or user group satisfies one of them, and it can have the corresponding operation permissions.
Permissions：
Read: Read operation. Corresponding to get and HEAD class operations in object storage, including downloading objects and querying object metadata.
Write: Write operation. Corresponds to modification operations such as the PUT class in object storage, such as uploading objects.
2. HDFS command test

(1) Authenticate a user through kinit

(2) Execute various hadoop fs commands
