# IPSec部署性能优化脚本

## **IPSec部署性能优化脚本**

在某些情况下，当IPsec中的明文或ESP数据包具有大量2层填充时，NP IPsec引擎可能无法处理它们并且产生丢包情况。

在部署IPSec项目时，强烈建议上线前在IPSec设备上的CLI下敲入如下命令（**敲入后，重启生效。建议在项目上线前实施。**）：

```
config system npu
    set strip-esp-padding enable
    set strip-clear-text-padding enable
end
```
