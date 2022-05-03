Sysrepo - Control & Config
----

We will use `sysrepo-netopeer` container (see References).

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

Get started with projects:

- libyang https://github.com/CESNET/libyang
- sysrepo https://github.com/sysrepo/sysrepo
- netopeer https://github.com/CESNET/Netopeer2

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
$ cd ${PROJECT_DIR}
$ mkdir build; cd build
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
gcc -o oper_data_push_example.out oper_data_push_example.c -I ${SYSREPO_DIR}/src/ -lsysrepo

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
$./oper_pull_push_example.out
```