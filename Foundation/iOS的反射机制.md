# iOS的反射机制

```
// SEL和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromSelector(SEL aSelector);
FOUNDATION_EXPORT SEL NSSelectorFromString(NSString *aSelectorName);
// Class和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromClass(Class aClass);
FOUNDATION_EXPORT Class __nullable NSClassFromString(NSString *aClassName);
// Protocol和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromProtocol(Protocol *proto) NS_AVAILABLE(10_5, 2_0);
FOUNDATION_EXPORT Protocol * __nullable NSProtocolFromString(NSString *namestr) NS_AVAILABLE(10_5, 2_0);
```

