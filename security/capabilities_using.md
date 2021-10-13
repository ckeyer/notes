# Capability 使用

## 资料

* 

在上一篇文章中已经讲到，Capabilities的主要思想在于分割root用户的特权，即将root的特权分割成不同的能力，每种能力代表一定的特权操作。

## 相关命令

### setcap

`setcap [-q] [-v] (capabilities|-|-r) filename [ ... capabilitiesN fileN ]`

给指定的文件设置特权能力

### getcap

### setpcap

## 示例

### CAP_CHOWN

```golang
    flag.Parse()
	args := flag.Args()
	if len(args) != 2 {
		return
	}
	uid := args[0]
	file := args[1]
	uidn, _ := strconv.Atoi(uid)
	err := os.Chown(file, uidn, -1)
	if err != nil {
		fmt.Println(err)
	}
```

### CAP_NET_BIND_SERVICE

