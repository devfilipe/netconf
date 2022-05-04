
## Part I - sysrepo
Sysrepo - Control & Config
----

We will use `sysrepo-netopeer2` container (see References > docker in this document).

```bash
$ docker pull sysrepo/sysrepo-netopeer2
$ docker run -it --name sysrepo -p 830:830 --rm sysrepo/sysrepo-netopeer2:latest
$ docker exec -it sysrepo /bin/bash
```

Let's see how to install the `openconfig-platform` model:

```bash
$ cd tmp; git clone ${OPENCONFIG_URL}
$ sysrepoctl -l
$ sysrepoctl -s /tmp/public/release/models/:/tmp/public/release/models/types/:/tmp/public/release/models/platform/ -i /tmp/public/release/models/platform/openconfig-platform.yang
```

And now, how to `create` some config and `get` it:

```bash
# create an instance on running config
$ sysrepocfg --import=/tmp/platform-linecard.xml -f xml -d running -x '/openconfig-platform:components'

# export running config to a file
$ sysrepocfg --export=/tmp/platform-linecard-exported.xml -f xml -d running -m 'openconfig-platform'
```

`platform-linecard.xml` file:

```xml
<components xmlns="http://openconfig.net/yang/platform">
  <component>
    <name>linecard0_1</name>
    <config>
      <name>linecard0_1</name>
    </config>
  </component>
</components>
```

References
----
- docker https://hub.docker.com/r/sysrepo/sysrepo-netopeer2

- useful commands https://blog.fearcat.in/a?ID=01700-da42805a-a9bd-4a80-81b8-ae7d32b903df


Development Environment
----

Get started with the projects:

- libyang https://github.com/CESNET/libyang
- sysrepo https://github.com/sysrepo/sysrepo


Main dependencies to build projects:

```bash
$ gcc --version
gcc (Debian 10.2.1-6) 10.2.1 20210110

$ cmake --version
cmake version 3.18.4

$ pcre2-config --version
10.36
```

For each package (libyang, sysrepo, ...) you can use:

```bash
$ git clone ${PROJECT_URL}
$ cd ${PROJECT_DIR}; mkdir build; cd build
$ cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
$ make
$ make install
```

Check it.

```bash
# ldconfig -p | grep <LIBNAME>
$ ldconfig -p | grep libyang
libyang.so.1 (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libyang.so.1
libyang.so (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libyang.so
```

References

https://boboob.tistory.com/entry/NETCONFYANG-0-How-to-install-the-development-environment-for-YANG-and-NETCONF

Build & Run
----

We will use the  `Development Environment`.

The `sysrepo` examples can be found at `sysrepo/examples` folder.

In the next 4 topics we will learn how to subscribe to data change and how to get data via pull and push modes:

- Subscription to data change
- Get data (pull)
- Get data (push)
- Get data (pull&push)

### Data change

In this example, our application will be notified when a change occurs on the monitored module.

Main functions:

- `sr_connect`
- `sr_start_session`
- `sr_module_change_subscribe`
- `sr_disconnect`

Terminal #1:

```bash
$ gcc -o application_changes_example.out application_changes_example.c -I${SYSREPO_DIR}/src/ -l sysrepo
$ chmod a+x application_changes_example.out
$ ./application_changes_example.out examples
Application will watch for changes in "examples".

 ========== READING RUNNING CONFIG: ==========

/examples:cont (container)


 ========== LISTENING FOR CHANGES ==========
```

Terminal #2:

```bash
$ sysrepocfg --import=/tmp/test-sysrepo/examples.xml -f xml -d running -x '/examples:cont'
```

In Terminal #1:

```bash
 ========== EVENT change CHANGES: ====================================

CREATED: /examples:cont/l = my_leaf

 ========== END OF CHANGES =======================================
```

Exercise: modify the xml and use `sysrepocfg --edit`.

### Data pull

In this example, data is mocked and returned by the subscriber.

Main functions:

- `sr_connect`
- `sr_start_session`
- `sr_get_oper_subscribe`
- `sr_disconnect`

Terminal #1

```bash
# compile
$ gcc -o oper_data_pull_example.out oper_data_pull_example.c -I${SYSREPO_DIR}/src/ -I${LIBYANG_DIR}/src -lsysrepo -lyang

# run
$ chmod a+x oper_data_pull_example.out
$ ./oper_data_pull_example.out examples /examples:stats
Application will provide data "/examples:stats" of "examples".



 ========== LISTENING FOR REQUESTS ==========
```

Terminal #2

```bash
$ sysrepocfg --export=/tmp/test-sysrepo/examples-oper-exported.xml -f xml -d operational -m 'examples'

$ cat /tmp/test-sysrepo/examples-oper-exported.xml
<stats xmlns="urn:examples">
  <counter>852</counter>
  <counter2>1052</counter2>
</stats>
```

### Data push

In this example there is no subscriber an data lifetime is limited by the lifetime of the connection.

```bash
# compile
$ gcc -o oper_data_push_example.out oper_data_push_example.c -I ${SYSREPO_DIR}/src/ -lsysrepo

# run
$ chmod a+x oper_data_push_example.out
$ ./oper_data_push_example.out
```

### Data pull & push

In this example both methods are visited.

```bash
# compile
$ gcc -o oper_pull_push_example.out oper_pull_push_example.c -I${SYSREPO_DIR}/src/ -I${LIBYANG_DIR}/src -lsysrepo -lyang

# run
$ chmod a+x oper_pull_push_example.out
$ ./oper_pull_push_example.out
```

## Part II - netopeer2

Get started with the projects:

- netopeer project https://netopeer.liberouter.org
- libnetconf2 https://github.com/CESNET/libnetconf2
- libyang https://github.com/CESNET/libyang
- sysrepo https://github.com/sysrepo/sysrepo
- netopeer2 https://github.com/CESNET/Netopeer2

Requirements for `netopeer2`:

- libnetconf2
- libyang
- sysrepo

Let's make it happen:

* Install `libssh` for _netconf over SSH_:

```bash
git clone http://git.libssh.org/projects/libssh.git
cd libssh; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
make
make install
```

* Install `libnetconf2`:

```bash
git clone https://github.com/CESNET/libnetconf2.git
cd libnetconf2; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
make
make install
```

* Install `libyang`:

```bash
git clone https://github.com/CESNET/libyang.git
cd libyang; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
make
make install
```

* Install `sysrepo`:

```bash
git clone https://github.com/sysrepo/sysrepo.git
cd sysrepo; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
make
make install
```

* Install `netopeer2`:

```bash
git clone https://github.com/CESNET/netopeer2.git
cd netopeer2; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
make
make install
```

Using _netopeer2_
----

* Start `netopeer-server`

```
$ netopeer2-server
```

* Check ssh

```bash
ssh <USERNAME>@localhost -p 830 -s netconf
```

WIP: skip it! Go to **Start netopeer2-cli**.

```
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<get-config>
  <source>
    <running />
  </source>
</get-config>
</rpc>

<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <capabilities>
    <capability>urn:ietf:params:netconf:base:1.0</capability>
    <capability>urn:ietf:params:netconf:base:1.1</capability>
    <capability>urn:ietf:params:netconf:capability:writable-running:1.0</capability>
    <capability>urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring?module=ietf-netconf-monitoring&amp;revision=2010-10-04</capability>
  </capabilities>
</hello>
]]>]]>
##
```

* Start `netopeer2-cli`

```bash
$ netopeer2-cli
```

```
> status
Client is not connected to any NETCONF server
```

```
> connect --ssh --port 830 --host localhost --login ${USER}
Interactive SSH Authentication
Type your password:
>
```

```
> status
Current NETCONF session:
  ID          : 6
  Host        : 127.0.0.1
  Port        : 830
  Transport   : SSH
  Capabilities:
        urn:ietf:params:netconf:base:1.0
        urn:ietf:params:netconf:base:1.1
        urn:ietf:params:netconf:capability:writable-running:1.0
        urn:ietf:params:netconf:capability:candidate:1.0
        urn:ietf:params:netconf:capability:confirmed-commit:1.1
        urn:ietf:params:netconf:capability:rollback-on-error:1.0
        urn:ietf:params:netconf:capability:validate:1.1
        urn:ietf:params:netconf:capability:startup:1.0
        urn:ietf:params:netconf:capability:xpath:1.0
        urn:ietf:params:netconf:capability:with-defaults:1.0?basic-mode=explicit&also-supported=report-all,report-all-tagged,trim,explicit
        urn:ietf:params:netconf:capability:notification:1.0
        urn:ietf:params:netconf:capability:interleave:1.0
        urn:ietf:params:netconf:capability:url:1.0?scheme=scp,http,https,ftp,sftp,ftps,file
        urn:ietf:params:xml:ns:yang:ietf-yang-metadata?module=ietf-yang-metadata&revision=2016-08-05
        urn:ietf:params:xml:ns:yang:1?module=yang&revision=2021-04-07
        urn:ietf:params:xml:ns:yang:ietf-inet-types?module=ietf-inet-types&revision=2013-07-15
        urn:ietf:params:xml:ns:yang:ietf-yang-types?module=ietf-yang-types&revision=2013-07-15
        urn:ietf:params:netconf:capability:yang-library:1.1?revision=2019-01-04&content-id=1
        urn:sysrepo:plugind?module=sysrepo-plugind&revision=2022-03-10
        urn:ietf:params:xml:ns:yang:ietf-netconf-acm?module=ietf-netconf-acm&revision=2018-02-14
        urn:ietf:params:xml:ns:netconf:base:1.0?module=ietf-netconf&revision=2013-09-29&features=writable-running,candidate,confirmed-commit,rollback-on-error,validate,startup,url,xpath
        urn:ietf:params:xml:ns:yang:ietf-netconf-with-defaults?module=ietf-netconf-with-defaults&revision=2011-06-01
        urn:ietf:params:xml:ns:yang:ietf-netconf-notifications?module=ietf-netconf-notifications&revision=2012-02-06
        urn:examples?module=examples
        urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring?module=ietf-netconf-monitoring&revision=2010-10-04
        urn:ietf:params:xml:ns:netmod:notification?module=nc-notifications&revision=2008-07-14
        urn:ietf:params:xml:ns:netconf:notification:1.0?module=notifications&revision=2008-07-14
        urn:ietf:params:xml:ns:yang:ietf-x509-cert-to-name?module=ietf-x509-cert-to-name&revision=2014-12-10
        urn:ietf:params:xml:ns:yang:iana-crypt-hash?module=iana-crypt-hash&revision=2014-08-06
>
```

Open a second terminal and write to `examples` data (see Part I):

Initial config `examples.xml`:

```xml
<cont xmlns="urn:examples">
  <l>my_leaf</l>
</cont>
```

Write directly to `running` datastore:

```bash
# set a value
$ sysrepocfg --import=/tmp/test-sysrepo/examples.xml -f xml -d running -x '/examples:cont'

# check it
$ sysrepocfg --export=/tmp/test-sysrepo/examples-exported.xml -f xml -d running -m 'examples'

$ cat /tmp/test-sysrepo/examples-exported.xml
<cont xmlns="urn:examples">
  <l>my_leaf</l>
</cont>
```

Back to `netopeer2-cli`:

```
> get-config --source running --filter-xpath '/examples:cont/l'
DATA
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <cont xmlns="urn:examples">
    <l>my_leaf</l>
  </cont>
</data>
```

Go to the second terminal end edit `examples.xml`:

```xml
<cont xmlns="urn:examples">
  <l>my_leaf_updated</l>
</cont>
```

Back to `netopeer2-cli` and try `edit-config`:
```
> edit-config --target running --config=examples.xml --defop merge
ERROR
        type:     protocol
        tag:      access-denied
        severity: error
        path:     /examples:cont/l
        message:  Access to the data model "examples" is denied because ${USER} NACM authorization failed.
```

Lets fix it (disable NACM).

Go to the second terminal and create a XML file named `nacm.xml`:

```xml
<nacm xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-acm">
  <enable-nacm>false</enable-nacm>
</nacm>
```

Then populate the `nacm` config:
```bash
$ sysrepocfg --import=/tmp/test-sysrepo/nacm.xml -f xml -d running -x '/ietf-netconf-acm:nacm'
```

Back to `netopeer2-cli`.

Check `nacm`:

```
> get-config --source running --filter-xpath '/ietf-netconf-acm:nacm'
DATA
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <nacm xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-acm">
    <enable-nacm>false</enable-nacm>
  </nacm>
</data>
```

Try `edit-config`:
```
> edit-config --target running --config=examples.xml --defop merge
OK
> get-config --source running --filter-xpath '/examples:cont/l'
DATA
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <cont xmlns="urn:examples">
    <l>my_leaf_updated</l>
  </cont>
</data>
```
