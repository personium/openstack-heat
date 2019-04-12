# HeatTemplate for creating Personium 3server unit

## Overview
This HeatTemplate automatically constructs the network and server configuration for Personium 3 Server unit in OpenStack.

#### Precondition
 * The operation confirmation is done by FUJITSU Cloud Service K5 of OpenStack base IaaS.
 * The network connecting to the Internet and the SSL-VPN connection for server access are excluded from the creation of this HeatTemplate.
 * Operation confirmation is done with CentOS 7.2.

## Construction environment
The configuration that can be created with this HeatTemplate is shown below.

<img src="3-server_heat.jpg" title="3-server_heat" style="width:70%;height:auto;">  

## File structure
| File name | Contents |
|:---:|:---:|
| 01_personium_network.yaml  |Create networks and firewalls.|
| 02_personium_security_group.yaml   |Create security group|
| 03_personium_3server.yaml   |Create servers for Personium.|


## Flow of creation
I will explain the flow of creation using this HeatTemplate.

### 1: Create Keypair (Manual)
Create a KeyPair to log in to the server for Personium.

### 2: Preparation for network creation
Edit 01_personium_network.yaml.

 *  Set the Availability Zone used for the default of availability_zone in the Parameters section.
```
  availability_zone:
    type: string
    description: Availability zone
    default: { Availability zone } # set Availability zone
```

### 3: Create network
Use 01_personium_network.yaml to create the network.

### 4: Connect with external network
Connect DMZ network, Secure network, Management network to external network.

### 5: Create security group
Use 02_personium_security_group.yaml to create a security group.

 *  change the Availability Zone from jp-west-1a as suitable for your environment.
```
 availability_zone:
    type: string
    description: Availability zone
    default: jp-west-1a
```

### 6: Preparation for server creation
Edit 03_personium_3server.yaml.

 *  Set the availability zone you are using to the default of availability_zone in the Parameters section.
```
  availability_zone:
    type: string
    description: Availability zone
    default: { Availability zone } # set Availability zone
```

 *  Network ID setting  
Obtain the ID of the created network, and in the Parameters section Set to default of dmz_network_id, secure_network_id, mng_network_id.  
```
dmz_network_id:
  type: string
  description: ID of the dmz network.
  default: { dmz_network_id } #set dmz network id.
```
```  
  secure_network_id:
  type: string
  description: ID of the secure network.
  default: { secure_network_id } #set secure network id.
```
```
mng_network_id:
  type: string
  description: ID of the management network.
  default: { management_network_id } #set management network id.
```

 *  Security group setting
Obtain the ID of the created security group, and in the Parameters section Set to default of dmz_secgroup_id, secure_secgroup_id, mng_secgroup_id.
```
dmz_secgroup_id:
  type: string
  description: Security group name of dmz network.
  default: { dmz_secgroup_id } #set DMZ security group ID.
```
```
secure_secgroup_id:
  type: string
  description: SSecurity group name of secure network
  default: { secure_secgroup_id } #set secure security group ID.
```
```
mng_secgroup_id:
  type: string
  description: Security group name of management network
  default: { mng_secgroup_id } #set management security group ID.
```

 *  KeyPair setting
Obtain the name of the created KeyPair and in the Parameters section
Set to web_server_key_name, ap_server_key_name, es_server_key_name default.
```
web_server_key_name:
  type: string
  description: Name of web server key.
  default: { your_server_keyname } #set server KeyPair name.
```
```
ap_server_key_name:
  type: string
  description: Name of ap server key.
  default: { your_server_keyname } #set server KeyPair name.
```
```
es_server_key_name:
  type: string
  description: Name of es server key.
  default: { your_server_keyname } #set server KeyPair name.
```

 *  Certificate setting
Edit the user_data of the web_server in the resources section according to the certificate you are creating.
```
  web_server:
    type: OS::Nova::Server
    properties:

        --Omission--

    user_data_format: RAW
    user_data:
      str_replace:
        template: |
          #!/bin/bash -v

    　　--Omission--

          openssl genrsa 2048 > /root/ansible/resource/web/opt/nginx/conf/server.key
          openssl req -new -key /root/ansible/resource/web/opt/nginx/conf/server.key << EOF > /root/ansible/resource/web/opt/nginx/conf/server.csr
          { Country Name } # set country name
          { State or Province Name } # set state name
          { Locality Name } # set locality name
          { Organization Name } # set oganization name
          { Organizational Unit Name } # set oganization unit name
          { Common Name } # set common name
          { Email Address } # set mail address
          { A challenge password } # set password
          { An optional company name } # set company name
          EOF
          openssl x509 -days 3650 -req -signkey  /root/ansible/resource/web/opt/nginx/conf/server.key < /root/ansible/resource/web/opt/nginx/conf/server.csr > /root/ansible/resource/web/opt/nginx/conf/server.crt

          openssl genrsa -out /root/ansible/resource/ap/opt/x509/unit.key 2048 -outform DER
          openssl req -new -key /root/ansible/resource/ap/opt/x509/unit.key -out /root/ansible/resource/ap/opt/x509/unit.csr << EOF
          { Country Name } # set country name
          { State or Province Name } # set state name
          { Locality Name } # set locality name
          { Organization Name } # set oganization name
          { Organizational Unit Name } # set oganization unit name
          { Common Name } # set common name
          { Email Address } # set mail address
          { A challenge password } # set password
          { An optional company name } # set company name
          EOF

          openssl x509 -req -days 3650 -signkey /root/ansible/resource/ap/opt/x509/unit.key -out /root/ansible/resource/ap/opt/x509/unit-self-sign.crt < /root/ansible/resource/ap/opt/x509/unit.csr
```

### 7: Create SSL-VPN (Manual)
Create an SSL-VPN connection to the Management network.

### 8: Setup Personium
Follow the steps below to set up Personium.
[ansible/3-server_unit](https://github.com/personium/ansible/tree/master/3-server_unit "3-server_unit")
