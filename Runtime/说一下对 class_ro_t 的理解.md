# 说一下对 class_ro_t 的理解
存储了当前类在编译期就已经确定的属性、方法以及遵循的协议。

```
struct class_ro_t {  
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
    uint32_t reserved;

    const uint8_t * ivarLayout;

    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;
};

```
在编译期间类的结构中的 `class_data_bits_t *data` 指向的是一个 `class_ro_t * `指针：

