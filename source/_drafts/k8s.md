## API对象

API对象是Kubernetes集群中的管理操作单元

每引入一项新技术，一定会新引入对应的API对象

四大类属性：

- ### TypeData

  引入了GKV模型定义一个对象

  Group:group定义了非常多的对象，将对象依据其功能范围归入不同的分组

  Kind：对象基本类型，比如Node，Pod，Deployment

  Version:版本

- ### MetaData

  最重要的属性：NameSpace和Name，这两个属性唯一定义了某个对象实例

  1.Lable 打标签，key不能超过63个字节，value可以为空，不超过253个字符串

  2.Annotation 当属性不够用的时候，可以放到annotation

  3.finalizer 是一种资源锁，当他不为空时，让对象只能被逻辑删除

  4.ResoureceVersion 乐观锁，确保分布式系统重任意多线程能够安全访问对象

- ### Status 实际状态

- ### Spec 期望状态

  环境变量：

  ​	直接设置值;

  ​	读取Pod Spec的某些属性；

  ​	从ConfigMap和Sercet读取

  挂载存储卷:volumn

  Pod网络

  资源限制：resources

  健康检查：LivenessProbe\ReadinessProbe\statusupProbe

  三种方式：Exec\TCP socket\HTTP

## 对象

Node

Namespace

Pod 是k8s的基础调度单元

job

PV和PVC

