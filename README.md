# Device42 Integration Pack

Integration pack that provides support for Device42, a self-documenting CMDB and single source of truth for all things
IT infrastructure

## Configuration

Copy the example configuration in [device42.yaml.example](./device42.yaml.example)
to `/opt/stackstorm/configs/device42.yaml` and edit as required.

* `d42_server` - Device42 instance address (with protocol and without trailing slash)
* `d42_username` - Device42 username
* `d42_password` - Device42 password
* `verify_certificate` - Set to `false` in case of self-signed SSL certificate

You can also use dynamic values from the datastore. See the
[docs](https://docs.stackstorm.com/reference/pack_configs.html) for more info.

**Note** : When modifying the configuration in `/opt/stackstorm/configs/` please
           remember to tell StackStorm to load these new values by running
           `st2ctl reload --register-configs`

## Supported Actions
```
+---------------------------------------------------+----------+------------------------------------------+
| ref                                               | pack     | description                              |
+---------------------------------------------------+----------+------------------------------------------+
| device42.device_name_list                         | device42 | Returns list of devices names            |
| device42.get_device_by_id                         | device42 | Get a device with full details from D42  |
|                                                   |          | by its ID                                |
| device42.get_dns_zone                             | device42 | Returns DNS zone file                    |
| device42.get_lifecycle_events                     | device42 | Get lifecycle events from D42 with       |
|                                                   |          | optional filtering parameters.           |
|                                                   |          | https://api.device42.com/#asset-device-  |
|                                                   |          | life-cycle                               |
| device42.suggest_next_ip                          | device42 | Suggest next available IP Address        |
| device42.udpate_object_category_by_lifecycle_id   | device42 | Update a devices object category and     |
|                                                   |          | more  on D42 based on an incoming        |
|                                                   |          | lifecycle event ID.                      |
| device42.update_device                            | device42 | Update a device on D42                   |
+---------------------------------------------------+----------+------------------------------------------+
```

## Examples

#### Get `device42.device_name_list` for FreeBSD
```sh
# st2 run device42.device_name_list os=freebsd

id: 57e2dad51d41c80479ff1de6
status: succeeded
parameters:
  os: freebsd
result:
  exit_code: 0
  result:
  - freebsd
  - freebsd-93-001
  - Unknown
  stderr: ''
  stdout: ''
```

#### Get `device42.suggest_next_ip` for subnet
```sh
# st2 run device42.suggest_next_ip subnet=10.15.24.0/22

id: 57e2d9741d41c80479ff1dc8
status: succeeded
parameters:
  subnet: 10.15.24.0/22
result:
  exit_code: 0
  result: 10.15.24.1
  stderr: ''
  stdout: ''
```

#### Get `device42.get_dns_zone` for domain
```sh
# st2 run device42.get_dns_zone domain=domain.com

id: 581c7c441d41c804d9067e59
status: succeeded
parameters:
  domain: domain.com
result:
  exit_code: 0
  result: ''@ 3600 IN SOA nh-win2k8r2-vm-03 hostmaster. 107493 900 600 86400 3600       
    @ 600 IN A 192.168.11.161       
    @ 3600 IN NS nh-win2k8r2-vm-03            
    gc._msdcs 600 IN A 192.168.11.161                  
    _ldap._tcp.gc._msdcs 600 IN SRV 0 100 3268 nh-win2k8r2-vm-03'
  stderr: ''
  stdout: ''
```

#### Get `device42.get_device_by_id`  
```sh 
# st2 run device42.get_device_by_id device_id=15 

id: 5a09b77bf2fdc607ac43f206
status: succeeded
parameters: 
  device_id: '15'
result: 
  exit_code: 0
  result:
    aliases: []
    asset_no: ''
    category: Inventoried_LC
    cpucore: 2
    cpucount: 2
    cpuspeed: 1900.0
    custom_fields:
    - key: Puppet Node ID
      notes: Puppet Server 10.42.7.60
      value: puppet.device42.pvt
    - key: test_field
      notes: null
      value: 22 days
    - key: test_field2
      notes: null
      value: Linux
    - key: node_classes
      notes: ''
      value: '{"classes": {"example": { "param": "example_param" } }, "environment": "production" }'
    - key: lifecycle
      notes: Delivered, Installed, Configured, Deployed, InService
      value: Delivered
    - key: Salt Node ID
      notes: null
      value: null
    customer: null
    device_external_links: []
    device_id: 15
    device_purchase_line_items: []
    hdd_details: null
    hddcount: 3
    hddraid: null
    hddraid_type: null
    hddsize: 18.25
    ...
  stderr: ''
  stdout: ''
```

#### Get `device42.get_lifecycle_events` with optional filtering. 
```sh
# st2 run  device42.get_lifecycle_events device_name="example.device42" date_gt="2017-11-11"  

id: 5a09b915f2fdc607ac43f209
status: succeeded
parameters: 
  date_gt: '2017-11-11'
  device_name: example.device42
result: 
  exit_code: 0
  result:
    lifecycle_events:
    - date: '2017-11-13T15:20:42Z'
      device: example.device42
      id: 30
      notes: ''
      type: Checked In
    limit: 1000
    offset: 0
    total_count: 1
  stderr: ''
  stdout: ''
```

### Put changes on device `device42.update_device` 
```sh
 st2 run device42.update_device identifier=15 identifier_type="device_id" changes='{"tags":"example_tag"}'
.................................
id: 5a09bb4bf2fdc607ac43f20c
status: succeeded
parameters: 
  changes:
    tags: example_tag
  identifier: '15'
  identifier_type: device_id
result: 
  exit_code: 0
  result:
    code: 0
    msg:
    - 'device added or updated '
    - 15
    - example.device42
    - true
  stderr: ''
  stdout: ''
  

```

## Debuging / General Tips 

- rules engine log: tail -f /var/log/st2/st2rulesengine.log

- Review the input parameters for a given action in its {action_name}.yaml file in /opt/stackstorm/packs/device42/actions

- An example json file is included to show the structure for importing data directly into the key value store for use with the device42fork.lifecycle_triggered_object_category_change rule. Associate Lifecycle type IDs from Device42 to the name of the new object category to apply to the device.    
