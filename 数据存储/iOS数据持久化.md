# iOS数据持久化
### iOS沙盒机制
#### iOS 沙盒机制简介
沙盒也叫沙箱，英文`standbox`，其原理是通过重定向技术，把程序生成和修改的文件定向到自身文件夹中。在沙盒机制下，每个程序之间的文件夹不能互相访问。iOS系统为了保证系统安全，采用了这种机制
iOS 应用程序在安装时，会创建属于自己的沙盒文件，应用程序不能直接访问其他应用程序的沙盒文件，当应用程序需要向外部请求或接收数据时，都需要经过权限认证，否则，无法获取到数据。
应用程序中所有的非代码文件都保存在沙盒中，比如图片、声音、属性列表，`sqlite`数据库和文本文件等。
#### 获取沙盒路径
通过`NSHomeDirectory()`获取沙盒路径并输出：

```
NSLog(@"%@",NSHomeDirectory());
```
#### 沙盒文件组成
沙盒的的根目录有四个文件夹，分别是 `Documents，Library，tmp，SystemData`
![](https://upload-images.jianshu.io/upload_images/1840444-ef584d21f251c8e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### Documents/
Documents中一般保存应用程序本身产生文件数据，例如游戏进度，绘图软件的绘图等， iTunes备份和恢复的时候，会包括此目录。
`注意：在此目录下不要保存从网络上下载的文件，否则app无法上架！`
获取Documents文件路径

```
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).lastObject;
```
#### Library/
`Library`目录下有两个子目录：`Caches` 和 `Preferences`**现在系统自动清理功能会将此缓存自动清理掉**。
![](https://upload-images.jianshu.io/upload_images/1840444-57be1ce787c8de8a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
获取Library路径

```
NSString *path = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).lastObject;
```
#### Library/Caches/
此目录用来保存应用程序运行时生成的需要持久化的数据，这些数据一般存储体积比较大，又不是十分重要，比如网络请求数据等。这些数据需要用户负责删除。`iTunes`同步设备时不会备份该目录。**`YYCache`保存的数据就是放这里的**
获取`Library/Caches`文件路径

```
NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).lastObject;
```
#### Library/Preferences/
此目录保存应用程序的所有偏好设置，iOS的`Settings`(设置)应用会在该目录中查找应用的设置信息。`iTunes`同步设备时会备份该目录
在`Preferences/`下不能直接创建偏好设置文件，而是应该使用`NSUserDefaults`类来取得和设置应用程序的偏好,`NSUserDefaults`默认存放在此文件夹
获取`Library/Preferences/`文件路径

```
NSString *path = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).lastObject stringByAppendingPathComponent:@"Preferences"];
```
#### tmp/
此目录保存应用程序运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录

```
NSString *path = NSTemporaryDirectory();
```
#### SystemData
新加入的一个文件夹
### iOS常见数据持久化方法

1. Property List
2. Preference（偏好设置）
3. NSKeyedArchiver归档(NSCoding)
4. SQLite3 
5. Core Data
#### Property List
如果对象是`NSString`、`NSDictionary`、`NSArray`、`NSData`、`NSNumber`等类型，就可以使用`writeToFile:atomically:`方法直接将对象写到属性列表文件中。存储在沙盒的`Documents`目录下
将一个`NSDictionary`对象归档到一个`plist`属性列表中。例如：

```
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
[dict setObject:@"母鸡" forKey:@"name"];
[dict setObject:@"15013141314" forKey:@"phone"];
[dict setObject:@"27" forKey:@"age"];
将字典持久化到Documents/stu.plist文件中
[dict writeToFile:path atomically:YES]; 
读取Documents/stu.plist的内容，实例化NSDictionary
NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:path];
NSLog(@"name:%@", [dict objectForKey:@"name"]);
NSLog(@"phone:%@", [dict objectForKey:@"phone"]);
NSLog(@"age:%@", [dict objectForKey:@"age"]);
```
#### Preference（偏好设置）
很多iOS应用都支持偏好设置，比如保存用户名，密码，字体大小等设置，iOS提供了一套标准的解决方案来为应用提供偏好设置，每个应用都有一个`NSUserDefaults`实例，通过它来保存偏好设置。只能存储系统自带的数据类型，自定义的对象无法存储。

```
NSUserDefaults *default= [NSUserDafaults  standardUserDefault]；
[default setObject:@"itcast" forKey:@"userName"];
[default setFloat:18.0f forKey:@"text_size"];
```
**注意**：`UserDefaults`设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。出现以上问题，可以通过调用`synchornize`方法强制写入

```
[defaults synchornize];
```
#### 偏好设置底层实现原理
底层其实就是利用一个字典，存储一些键值对。存储在沙盒的`Library/Preferences`目录下
偏好设置好处：能快速存储一些键值对，如果用字典去存储，还需要获取文件名比较麻烦。
偏好设置坏处：不能及时存储，需要做同步操作，把内存中的数据同步到硬盘上。

### NSKeyedArchiver（归档）
如果对象是`NSString`、`NSDictionary`、`NSArray`、`NSData`、`NSNumber`等类型，可以直接用`NSKeyedArchiver`进行归档和恢复
不是所有的对象都可以直接用这种方法进行归档，只有遵守了`NSCoding`协议的对象才可以。
#### NSCoding协议有2个方法
`encodeWithCoder:`
每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用`encodeObject:forKey:`方法归档实例变量
`initWithCoder:`
每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用`decodeObject:forKey`方法解码实例变量。例如

```
@interface Person : NSObject<NSCoding>
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) int age;
@property (nonatomic, assign) float height;
@end
@implementation Person
- (void)encodeWithCoder:(NSCoder *)encoder {
    [encoder encodeObject:self.name forKey:@"name"];
    [encoder encodeInt:self.age forKey:@"age"];
    [encoder encodeFloat:self.height forKey:@"height"];
}
- (id)initWithCoder:(NSCoder *)decoder {
    self.name = [decoder decodeObjectForKey:@"name"];
    self.age = [decoder decodeIntForKey:@"age"];
    self.height = [decoder decodeFloatForKey:@"height"];
    return self;
}
- (void)dealloc {
    [super dealloc];
    [_name release];
}
@end
 
归档(编码)
Person *person = [[[Person alloc] init] autorelease];
person.name = @"MJ";
person.age = 27;
person.height = 1.83f;
[NSKeyedArchiver archiveRootObject:person toFile:path];
恢复(解码)
Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
```
#### NSKeyedArchiver-归档对象的注意
如果父类也遵守了`NSCoding`协议，请注意：
应该在`encodeWithCoder:`方法中加上一句
`[super encodeWithCode:encode]`;
确保继承的实例变量也能被编码，即也能被归档
应该在`initWithCoder:`方法中加上一句
`self = [super initWithCoder:decoder]`;
确保继承的实例变量也能被解码，即也能被恢复。
#### NSData-归档2个Person对象到同一文件中
使用`archiveRootObject: toFile:`方法可以将一个对象直接写入到一个文件中，但有时候可能想将多个对象写入到同一个文件中，那么就要使用`NSData`来进行归档对象。
`NSData`可以为一些数据提供临时存储空间，以便随后写入文件，或者存放从磁盘读取的文件内容
。可以使用`[NSMutableData data]`创建可变存储空间。例如
 新建一块可变数据区

```
NSMutableData *data = [NSMutableData data];
 将数据区连接到一个NSKeyedArchiver对象
NSKeyedArchiver *archiver = [[[NSKeyedArchiver alloc] initForWritingWithMutableData:data] autorelease];
 开始存档对象，存档的数据都会存储到NSMutableData中
[archiver encodeObject:person1 forKey:@"person1"];
[archiver encodeObject:person2 forKey:@"person2"];
 存档完毕(一定要调用这个方法)
[archiver finishEncoding];
 将存档的数据写入文件
[data writeToFile:path atomically:YES]; 
NSData-从同一文件中恢复2个Person对象
恢复（解码）
从文件中读取数据
NSData *data = [NSData dataWithContentsOfFile:path];
根据数据，解析成一个NSKeyedUnarchiver对象
NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
Person *person1 = [unarchiver decodeObjectForKey:@"person1"];
Person *person2 = [unarchiver decodeObjectForKey:@"person2"];
 恢复完毕
[unarchiver finishDecoding];
```
#### 归档可实现深复制

```
比如对一个Person对象进行深复制
//临时存储person1的数据
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:person1];
//解析data，生成一个新的Person对象
Person *person2 = [NSKeyedArchiver unarchiveObjectWithData:data];
//分别打印内存地址
NSLog(@"person1:0x%x",person1);//person1:0x7177a60
NSLog(@"person2:0x%x",person2);//person2:0x7177cf0
```
### SQLite3
SQLite3是一款开源的嵌入式关系型数据库，可移植性好，易使用，内存开销小
SQLite3是无类型的，意味着你可以保存任意类型的数据到任意表的任意字段中
SQLite3常用的5种数据类型：`text`，`integer`，`float`，`boolean`，`blob`
在iOS中使用SQLite3，首先要添加libsqlite3.dylib和导入主头文件
### Core Data
Core Data框架提供了对象-关系映射的功能，即能够将OC对象转化成数据，保存在SQlite3数据库文件中，也能够将保存在数据库中的数据还原成OC对象。在此数据操作期间不需要编写任何SQL语句。使用此功能，要添加CoreData.framework头文件<CoreData/CoreData.h>

