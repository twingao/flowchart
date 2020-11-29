# 几种常用的重定向<<和管道|的用法

执行一个shell命令行时通常会自动打开三个标准文件，即标准输入文件（stdin），通常对应终端的键盘；标准输出文件（stdout）和标准错误输出文件（stderr），这两个文件都对应终端的屏幕。进程将从标准输入文件中得到输入数据，将正常输出数据输出到标准输出文件，而将错误信息送到标准错误文件中。


| 文件描述符 | 文件 | 文件 | 对应的设备 |
| :-- | :-- | :-- | :-- |
| 0 | 标准输入文件 | standard input | keyboard, mouse|
| 1 | 标准输出文件 | standard output| monitor |
| 2 | 标准错误输出文件 | standard error| monitor |


### 输出重定向

可以将标准输出重定向到文件中。

    command > file1

如将echo的输出重定向到一个文件中。

    echo "hello world" > h.txt

    cat h.txt
    hello world

">"会覆盖文件中的原来内容。

    echo "你好" > h.txt

    cat h.txt
    你好

">>"会追加在源文件的后面，不会覆盖原有内容

    echo "hello world" >> h.txt

    cat h.txt
    你好
    hello world

### 输入重定向

同样，可以将标准输入重定向到文件上，将文件的内容作为标准输入的内容。

    command < file1

如cat也可以接受标准输入的内容，以ctrl+D作为结束。

    cat
    hello world
    hello world
    你好
    你好

### Here Document（重点）

Here Document是Shell中的一种特殊的重定向方式，用来将输入重定向到一个交互式Shell脚本或程序。

它的基本的形式如下：

    command << delimiter
        document
    delimiter

它的作用是将两个delimiter之间的内容(document)作为输入传递给command。

> 结尾的delimiter一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和tab缩进。
> 开始的delimiter前后的空格会被忽略掉。

    cat << EOF
    hello world
    你好
    EOF
    hello world
    你好

"<<"常用的用法有是创建新文件。将delimiter之间的内容重定向到一个文件中。

    cat << EOF > config.yml
    key1: value1
    key2: value2
    key3: value3
    EOF

    cat config.yml
    key1: value1
    key2: value2
    key3: value3

delimiter是任何字符都行，不是必须EOF，但EOF表示End Of File，通常使用EOF作为delimiter。

    cat << eof > config.yml
    key1: value1
    key2: value2
    key3: value3
    eof

    cat config.yml
    key1: value1
    key2: value2
    key3: value3

或者

    cat << aaa > config.yml
    key1: value1
    key2: value2
    key3: value3
    aaa

    cat config.yml
    key1: value1
    key2: value2
    key3: value3

也可以将文件名放在前面。

cat > config.yml << EOF
key1: value1
key2: value2
EOF

可以追加文件。

    cat << EOF >> config.yml
    key4: value4
    key5: value5
    EOF
    
    cat config.yml
    key1: value1
    key2: value2
    key3: value3
    key4: value4
    key5: value5

### 管道（重点）

通过echo '...'将内容输出到标准输出stdout上，然后通过管道|将内容又重定向到标准输入stdin，而`kubectl apply -f -`中的-表示从标准输入获取输入，所有刚好将echo的内容传递给kubectl。这种做法很常用。

    echo '
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config1
    data:
      apploglevel: "info"
      appdatadir: /var/data
    ' | kubectl apply -f -
    configmap/config1 created

    kubectl get configmap/config1 -oyaml
    apiVersion: v1
    data:
      appdatadir: /var/data
      apploglevel: info
    kind: ConfigMap
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          {"apiVersion":"v1","data":{"appdatadir":"/var/data","apploglevel":"info"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"config1","namespace":"default"}}
      creationTimestamp: "2020-04-08T15:17:37Z"
      managedFields:
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:data:
            .: {}
            f:appdatadir: {}
            f:apploglevel: {}
          f:metadata:
            f:annotations:
              .: {}
              f:kubectl.kubernetes.io/last-applied-configuration: {}
        manager: kubectl
        operation: Update
        time: "2020-04-08T15:17:37Z"
      name: config1
      namespace: default
      resourceVersion: "108937"
      selfLink: /api/v1/namespaces/default/configmaps/config1
      uid: c4173032-8a0e-4020-88cd-bded26000de0

