# 概述

Todo

## Playbook编辑执行
先进行一下Shell模块操作,测试建立目录与删除目录
先查看一下客户端的server-test是否存在,如果没有我们就创建

```
root@hadoopmaster:~# ansible hadoop -m shell -a "ls -l /tmp/server-test" -f 5
192.168.1.76 | FAILED | rc=2 >>
ls: cannot access /tmp/server-test: No such file or directory

192.168.1.166 | FAILED | rc=2 >>
ls: cannot access /tmp/server-test: No such file or directory

192.168.1.159 | FAILED | rc=2 >>
ls: cannot access /tmp/server-test: No such file or directory

root@hadoopmaster:~# ansible hadoop -m shell -a "mkdir -p  /tmp/server-test" -f 5
192.168.1.159 | SUCCESS | rc=0 >>


192.168.1.76 | SUCCESS | rc=0 >>


192.168.1.166 | SUCCESS | rc=0 >>
```

我们先来编写一下playbook

```
root@hadoopmaster:~# nano test.yml
root@hadoopmaster:~# cat test.yml
- hosts: hadoop
  remote_user: root
  tasks:
  - name: delete /tmp/server-test
    shell: rm -rf /tmp/server-test
```

我们运行一下

```
root@hadoopmaster:~# ansible-playbook test.yml  -f 5

PLAY [hadoop] ******************************************************************

TASK [setup] *******************************************************************
ok: [192.168.1.76]
ok: [192.168.1.159]
ok: [192.168.1.166]

TASK [delete /tmp/server-test] *************************************************
changed: [192.168.1.166]
 [WARNING]: Consider using file module with state=absent rather than running rm

changed: [192.168.1.159]
changed: [192.168.1.76]

PLAY RECAP *********************************************************************
192.168.1.159              : ok=2    changed=1    unreachable=0    failed=0
192.168.1.166              : ok=2    changed=1    unreachable=0    failed=0
192.168.1.76               : ok=2    changed=1    unreachable=0    failed=0
```

## Python调用API执行

使用Python调用API执行

```

```

## java调用Python程序进行Playbook的执行

```
import java.io.InputStreamReader;

import java.io.BufferedReader;

import java.io.InputStream;

import java.io.IOException;

public class Test{

  public static void main(String args[]) throws IOException{

    String command="ansible-playbook test.yml  -f 5";

    Process process = Runtime.getRuntime().exec(command);

    printStream(process.getInputStream());

    printStream(process.getErrorStream());

    System.out.println("finish");

  }

private static void printStream(InputStream inputStream) {

  if (inputStream == null){

    System.out.println("input null");

    return;

  }

  String line = "";

    try(BufferedReader input = new BufferedReader(new InputStreamReader(inputStream)))

    {

      while ((line = input.readLine()) != null) {

        System.out.println(line);

      }

    } catch (IOException e1) {

        System.out.println("输出流失败"+e1);

        e1.printStackTrace();

    }

  }

}
    
```




