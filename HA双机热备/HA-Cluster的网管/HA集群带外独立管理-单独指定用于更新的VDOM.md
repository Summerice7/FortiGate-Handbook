# HA集群带外独立管理-独立管理VDOM

## 组网需求

FortiGate工作于多VDOM场景，管理VDOM无法访问Internet，业务VDOM是可以访问Internet的，客户想使用业务VDOM来实现Fortiguard更新。

## 解决方案

**在7.2.3之后**版本可以通过非管理VDOM实现Fortiguard更新：

```
config system fortiguard
	set vdom "root"
end
```

具体请参考：https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-use-non-management-VDOM-for-Fortiguard/ta-p/231010
