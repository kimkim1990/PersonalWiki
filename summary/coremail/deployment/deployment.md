## Deployment of Coremail system	[Back](./../coremail.md)

Configuration files of Coremail systems are always formatted with `.ini` extension name, in which items are always set as `[xxx]`, and commented with `#`.

- **coremail.cf**
    - host id
    - information of **adminsvr**
    - environmental parameters
- **global.cf**
    - information of license
    - switches of global functions
- **datasources.cf**
    - types of databases
    - host/port
    - user/password
    - name of databases
    - character set
    - connection counts/timeout
- **userschema.cf**
    - custom character field (separated with comma)
- **hosts.cf**
    - host id of devices
    - IP
    - program list
    - program id (UDID, MSID)
    - program weight (MSWeight)
    - configuration overriding
- **programs.cf**
    - the name of configuration items
    - server list
    - information of logs
    - server to connect other programs
    - specific configuration of programs
- **services.cf**
    - the name of configuration items
    - server side servers
        - types of service
        - information of connection (size of queues, timeout)
        - connection limitation
    - client side servers
        - which server side service to connect
        - information of connection (maximum of connection, timeout)
        - version of protocols
- **iplimit.cf**
    - the name of configuration items
    - amounts of IP
    - IP addreses
- **resources.cf**
    - host lists
- **policy.cf**
    - weak password policy
    - users grouping
        - user lists
        - COS lists, COS id
- **language.cf**, **res.cf**, **res_{locale}.cf**
    - style of languages
    - attribute name
    - attribute alternative
    - tags