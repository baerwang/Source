# Consul

## 启动问题

这个只能自己本机访问

```cmd
./consul agent -dev
```

如果要机器可以任何访问

```cmd
./consul agent -dev -ui -node=consul-dev -client=本机的私有ip地址 -bind=本机的私有ip地址
```

```cmd
consul agent -server -bootstrap -ui -data-dir=/data/soft/consul_1.4/consul-data -bind=本机的私有ip地址 -client=本机的私有ip地址  -node=本机的私有ip地址
```







