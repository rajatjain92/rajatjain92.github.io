# Backup And Restore Configuration in OpenBMC

## Description

Create a Redfish API which provides functionality to backup and restore BMC configurations. Having a backup of desired configurations can be useful after a factory reset or in case of BMC replacement. It helps to get system back to known settings.


## Configurations to be BackedUp

- **Network Service** -  hostname, network interface, MAC address, default gateway,
                         IPv4 address, IPv4 gateway, IPv4 network prefix length,
                         DNS Servers.
- **SNMP Service** - Manager Host address, Manager Host IP.
- **NTP Service** - BMC Host time settings, BMC Host Date Settings.
- **LDAP Service** - Server URI, BIND DN , BIND password, Role group settings etc.
- **UserAccess Service** - Local user settings, username, password, SSL certificates.


## Requirements

- Provide a consolidated backup file containing all desired BMC configurations.
- Provide the checklist option to the API user to select which settings he wants to store e.g. SNMP, Network, LDAP, ALL etc.
- The backup and restore functionality must be accessible over multiple interfaces , means it should work not only with redfish but other interfaces like IPMI also.
- At the time of restore , all BMC parameters should be set properly as provided by backup file.
- Concept of versioning - Ability to discard restore functionality by a particular app due to changes across application versions. Application versions may include changes to Redfish schemas and available parameters. A particular service might accept certain backed up data or not based on versioning.
- The new Backup and Restore API requires that the Redfish session is running under an admin role in order to ensure the data being retrieved or uploaded is coming from a secure source.
- When "old" configurations are applied to a newer firmware revision that contains new parameters, new parameters will take on their default value.


## High Level Design Diagram

![alt text](/docs/assets/HighLevelDesign.PNG)


## High Level Design

- First the user will make a POST call on redfish endpoint /redfish/v1/backupConfiguration . User will also provide service names which he want to backup. There will also be option to backup all services.
- BmcWeb will then call Backup method of BackupAndRestore service manager with arguments as service names.
- BackupAndRestore service manager acts as orchestrator. It is responsible for encryption/decryption of backup file. It is also responsible for creating manifest of all backup files received from desired services.
- Now for each service received in argument , BackupAndRestore service calls their backup method .
- Now in each service like Network service will backup its own data and provide its path to main orchestrator service.
- After receiving all backup file paths from desired services , orchestrator service will combine them into a single file and provide it to the user.
- Even if the new service is added there will be no changes to be made in orchestrator service. Its responsibility of new service to implement BackupAndRestore functionality.
- Versioning, encryption/decryption is responsibility of orchestrator service.


## Backup Configuration

![alt text](https://github.com/rajatjain92/BackupAndRestoreConfiguration/blob/3ed478b571bfefd1be5a33323a0b9981a92f3a4f/Backup.PNG)

### Flow of backup configuration

- WebUI(User) will do a POST call on redfish end point /redfish/v1/BackupConfiguration to the BMCWeb. User will also provide list of desired services to be backed up.
- BMCWeb will call to backup method in BackAndRestore Manager service(dbus service). BMCWeb will also provide list of desired service names to be backed up. This call will be ASYNCHRONOUS in nature. We need to find a way to tell Redfish service that backup is done. (DISCUSS)
- Once the BR service recieves request for backup along with desired service names as argument, it calls backup method defined in each desired service. BR service will trigger individual service backup calls in parallel. These calls will be SYNCHRONOUS in nature.  
- After each service e.g. network recieves request for backup, it returns unixfd or filePath of desired backup service configuration file. 
- As soon as the BR orchestrator service recieves all files unixfd's/paths of desired services , BR orchestrator service will try to package all these files for user. 
- There are 2 ways in which we can build this file. 
    - **Single file** - Either it can be a single yaml file containing service name as 
    its key and contents to be backed up for that service as value. 
    - **Multiple files** - Another way is to provide a zip file which contains files related to each desired service separately. As a result we do not have to do any extra work at the time of restore of separating backup content for each respected service.
- After the backup file is ready ,BR service will provide unixfd/path to BMCWeb. It can then do a GET call to get the file and provide to the user. 

### Constraints in Backup Configuration

- Decide where should the backup file be stored (RAM or flash) ?
- How big the backup file can get in size? Can we run out of space in tmpfs? 
- Does individual services should have a cap on amount of data to be backed up.
- Does we package backup of all desired services in 1 file or provide zip of individual service files ? what package mechanism should we use. ??
- How does Redfish will know that whole backup is done ?
- Does backup method should return unixfd or path of file.
- Synchronous/Asynchronous nature of calls.

## Restore Configuration

![alt text](https://github.com/rajatjain92/BackupAndRestoreConfiguration/blob/2313bfdf65a051b10cbdc86a5d920c0760745ef7/Restore.PNG)

### Flow in Restore Configuration

- WebUI(User) will do a POST call on redfish end point /redfish/v1/RestoreConfiguration to the BMCWeb. User will also provide backup file along with it.
- BMCWeb will call to restore method in BackAndRestore Manager service(dbus service). BMCWeb will provide backup file as an argument. This call will be ASYNCHRONOUS in nature. We need to find a way to tell Redfish service that end to end restore is done. (DISCUSS)
- Once the BR service recieves request for restore along with backup file, it calls restore method defined in each desired service. BR service will then separate each service configurations and pass it along with restore method call of indivial service.BR service will trigger individual service restore calls in parallel. These calls will be SYNCHRONOUS in nature. 
- After each service has complete restore method , we need to reboot the system.

### Constraints in Restore Configuration

- How does BR app know service restore is done - D-Bus API impact?
- How does Redfish know end to end restore is done?
- How are Restore errors communicated? Look at D-Bus errors in pdi.
- Async and progress behavior for Restore?
- Reboot of BMC after Restore - needs a trigger? BMC reboots itself - check?
- Malformed backup data needs to be handled at the service level
- BR needs to log errors if after reboot specific services do not start?

## Make Service/App available for Backup/Restore

- To make network service available for backup and restore service needs to implement 
BackupAndRestore.interface.yaml .
- Under objectPath “/xyz/openbmc_project/network”

```yaml
{
  methods:
    - name: Backup
      description: >
        method to create file which contains configurations of desired services.
      returns:
        - name: FilePath/unixfd
          type: string
          description: >
            it returns path/unixfd of single consolidated config backup file.

    - name: Restore
      description: >
        method to restore configurations of desired services from user provided backup file.
      parameters:
        - name: FilePath/unixfd
          type: string
          description: >
            The user provides backup conf file.

}
```


## BackupAndRestore manager service

- New BackupAndRestore service will be created. This service will act as an orchestrator .
- Responsible for creating single consolidated backup file containing all desired service and provide to the user.
- Responsible for creating manifest and encryption/decryption.
- Implements interface BackupAndRestoreManager.interface.yaml .
- If any new service is added in future no change is 
Needed in this service.
- Should we create 2 interfaces or not.

BackupAndRestore Manager service **"xyz.openbmc_project.BackupAndRestore.Manager"**
- /xyz/openbmc_project/BackupAndRestoreConfiguration:
    - xyz.openbmc_project.BackupAndRestoreConfiguration:
        - Methods: backup, restore

```yaml
{
  methods:
    - name: Backup
      description: >
        method to create file which contains configurations of desired services.
      parameters:
        - name: ServiceNames
          type: array[string]
          description: >
            The user provides desired service names we want to backup.
      returns:
        - name: FilePath/unixfd
          type: string
          description: >
            it returns path/unixfd of single consolidated config backup file.

    - name: Restore
      description: >
        method to restore configurations of desired services from user provided backup file.
      parameters:
        - name: FilePath/unixfd
          type: string
          description: >
            The user provides backup conf file.

}
```

## Configurations files used for different services

To make a service ready for backup and restore , its service implementer responsibility to create a persistent file containing desired configurations and this configuration file should be used to restore or even to regain state of service in case of reboot. All services like Network service , SNMP, User Manager, LDAP etc uses same approach. e.g.

- **Network Service**
    - NETWORK_CONF_DIR = **"/etc/systemd/network"**
    - firstBootPath = **"/var/lib/network/firstBoot_"**
    - configFile = **"/usr/share/network/config.json”**
- **SNMP Service**
    - Serialize and persist SNMP manager/client D-Bus object
    - SNMP_CONF_PERSIST_PATH = **"/var/lib/phosphor-snmp/managers/"**
    - File present at this path is used at time of reboot.
- **User Manager Service**
    - “/etc/passwd”
    - “/etc/shadow” 
- **LDAP Service**
    - LDAP_MAPPER_PERSIST_PATH = **"/var/lib/phosphor-ldap-mapper/groups"**
    - LDAP_CONF_PERSIST_PATH = **"/var/lib/phosphor-ldap-conf"**
    - Both these paths are used to restore the state.
- **Time Manager Service**
    - Manual Set time.
    - Use NTP protocol to set time.
    - Phosphor-settingsd service is used to populate. “Settings.yaml” file is used to store mode.
    - SETTINGS_PERSIST_PATH = **"/var/lib/phosphor-settings-manager/settings"**


```bash
root@dgx:/var/lib/phosphor-settings-manager/settings/xyz/openbmc_project# ls
control  time
root@dgx:/var/lib/phosphor-settings-manager/settings/xyz/openbmc_project# cd time/
root@dgx:/var/lib/phosphor-settings-manager/settings/xyz/openbmc_project/time# ls
sync_method__
root@dgx:/var/lib/phosphor-settings-manager/settings/xyz/openbmc_project/time# cat sync_method__
{
    "value0": {
        "cereal_class_version": 1,
        "value0": 1
    }
```

If there are some properties which we want to backup but are not present in persistent file then 
we need to add code in desired service to append property to persistent file.  


## Alternatives

- one alternative is backup data flash? We care more about settings. Doesn't work for service level backup?
