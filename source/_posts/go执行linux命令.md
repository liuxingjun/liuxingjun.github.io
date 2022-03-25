---
title: go执行linux命令进化史
date: 2020-03-25 14:06:46
categories: golang
---

起因是因为项目内使用了一个执行命令的 [deputy](https://github.com/natefinch/deputy)包,需要移除包，重构代码
```
func SyncRunTimeout(timeoutSeconds time.Duration, commandName string, params ...string) (string, error) {
	out := ""
	isTimeout := false
	cancel := make(chan struct{})
	go func() {
		<-time.After(time.Second * timeoutSeconds)
		isTimeout = true
		close(cancel)
	}()
	d := deputy.Deputy{
		Errors: deputy.DefaultErrs,
		StdoutLog: func(b []byte) {
			out = string(b)
			log.Println(out)
		},
		StderrLog: func(b []byte) {
			out = string(b)
			log.Println(out)
		},
		Cancel: cancel,
	}

	cmd := exec.Command(commandName, params...)
	if err := d.Run(cmd); err != nil {
		return out, err
	}

	if isTimeout {
		return "", errors.New("exec cmd timeout")
	}
	return out, nil
}
```
这个方法实现了执行超时错误，返回命令输出，命令执行中实时打印输出3大功能
检索资料借鉴了[GO语言执行命令超时的设置](https://linkscue.com/posts/2018-06-03-golang-exec-command-timeout-trace/)
得到如下代码
```
func SyncRunTimeout(timeoutSeconds time.Duration, commandName string, params ...string) (string, error) {
	cmd := exec.Command(commandName, params...)
	var stdout bytes.Buffer
	cmd.Stdout = &stdout
	err := cmd.Start()
	if err != nil {
		return "", err
	}
	done := make(chan error)
	go func() { done <- cmd.Wait() }()
	after := time.After(time.Second * timeoutSeconds)
	select {
	case <-after:
		err = cmd.Process.Signal(syscall.SIGINT)
		if err != nil {
			return "", err
		}
		time.Sleep(time.Second)
		err = cmd.Process.Kill()
		if err != nil {
			return "", err
		}

		return "", errors.New("exec cmd timeout")
	case <-done:
	}
	return stdout.String(), nil
}
```
当时如上方法有个问题，无输出时命令执行错误无法提示
例如 `echo "abc" | grep d`
```
$ echo "abc" | grep d
$ echo $?
1
```
如果用如上代码执行`echo "abc" | grep d`,既没有返回命令输出，也没有报错，这在单元测试是不通过的

既然 `cmd.Wait()` 执行后返回值放到了done chan， 那判断下就行
将select case 修改为如下
```
select {
case <-after:
    err = cmd.Process.Signal(syscall.SIGINT)
    if err != nil {
        return "", err
    }
    time.Sleep(time.Second)
    err = cmd.Process.Kill()
    if err != nil {
        return "", err
    }
    return "", errors.New("exec cmd timeout")
case err := <-done:
    if err != nil {
        return "", err
    }
}
```
到这里 执行超时错误，返回命令输出 2大功能已经实现了，只剩下实时日志输出了
看如下代码
```
cmd := exec.Command(commandName, params...)
var stdout bytes.Buffer
cmd.Stdout = &stdout
```
那是不是把 os.Stdout 赋值给 cmd.Stdout 就可以看到实时输出了？
我太天真的，虽然可以实时日志输出，但是没有返回值了，因为输出到了 `os.Stdout`,变量`var stdout bytes.Buffer`就没有数据，最后没办法return

我们需要一个变量保存 命令执行的输出，然后实时从这个变量里面取出日志打印，命令执行完之后，把这个变量返回出去
那我直接读取变量`stdout` 不就可以了吗？
在执行`done <- cmd.Wait()`的协程前增加读取程序并打印
```
go func() {
    for {
        tmp := make([]byte, 256)
        if _, err = stdout.Read(tmp); err == io.EOF {
            break
        }
        line := string(tmp)
        CMDLog.Debugf(line) // 打印日志
        log.Println(line)
        out += line         //追加命令执行输出
    }
    done <- cmd.Wait()
}() 
```
这里就完成了，实时日志打印

不过有个隐患，这里设置的定长块256，如果打印一些有格式的输出，超过256长度打印输出的时候会把格式打乱，比如命令为`ls -l`时
```
$ go run ./test.go 
2022/03/25 15:15:09 total 52
drwxr-xr-x 3 root root 4096 Jun  4  2021 build
drwxr-xr-x 3 root root 4096 Jun  4  2021 cmd
drwxr-xr-x 4 root root 4096 Mar 24 11:18 conf
-rw-r--r-- 1 root root 1024 Mar 24 11:18 go.mod
-rw-r--r-- 1 root root 9145 Mar 21 16:12 go.sum
-rw-r--r-- 1 
2022/03/25 15:15:09 root root 2005 Sep 28 21:03 Makefile
drwxr-xr-x 6 root root 4096 Mar 21 16:37 pkg
drwxr-xr-x 2 root root 4096 Jun  4  2021 script
drwxr-xr-x 2 root root 4096 Mar  7 20:29 test
-rw-r--r-- 1 root root 2198 Mar 25 15:15 test.go
-rwxr-xr-x 1 root root   45 Mar
2022/03/25 15:15:09  25 11:51 test.sh
```
在Makefile 上一行就发生了错位
需要使用按行读取
而且拿到返回值后对比也有问题
```
func main() {
	out, err := SyncExecShell(120, "echo", "hello")
	if err != nil {
		panic(err)
	}
	if out == "hello" {
		fmt.Println("一致")
	} else {
		fmt.Println("不一致")
	}
	fmt.Println(out, ":", []byte(out))
	fmt.Println("hello:", []byte("hello"))
}
```
```
$ go run ./test.go 
2022/03/25 15:27:52 hello

不一致
hello: [104 101 108 108 111 10 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
hello: [104 101 108 108 111]
```
这里可以看到，因为是定长的byte，所以后面用0来补位了
我们需要使用其他读取的方法，比如[bufio](https://pkg.go.dev/bufio)包的的[Scanner](https://pkg.go.dev/bufio#Scanner)类型
最终代码如下
```
func SyncExecShell(timeoutSeconds time.Duration, commandName string, params ...string) (string, error) {
	cmd := exec.Command(commandName, params...)
	//显示运行的命令
	out := ""
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		return "", err
	}
	if err = cmd.Start(); err != nil {
		return "", err
	}
	done := make(chan error)
	go func() {
		scanner := bufio.NewScanner(stdout)
		for scanner.Scan() {
			log.Println(scanner.Text())
			out += scanner.Text() //追加命令执行输出
		}
		done <- cmd.Wait()
	}()
	after := time.After(time.Second * 120)
	select {
	case <-after:
		err = cmd.Process.Signal(syscall.SIGINT)
		if err != nil {
			return "", err
		}
		time.Sleep(time.Second)
		err = cmd.Process.Kill()
		if err != nil {
			return "", err
		}
		return "", errors.New("exec cmd timeout")
	case err = <-done:
		if err != nil {
			return "", err
		}
		return out, nil
	}
}
```

至此，所有功能完美使用