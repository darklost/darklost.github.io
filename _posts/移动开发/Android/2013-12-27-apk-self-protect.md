---
layout: post  
title: APK自我保护 
category: 移动开发  
tags: Android 安全 
keywords: Android safe
description:   
---
/*

*  作者：MindMac

*  原文链接：[http://www.sanwho.com/445.html](http://www.sanwho.com/445.html)

*  转载请注明出处

*/

由于Android应用程序中的大部分代码使用Java语言编写，而Java语言又比较容易进行逆向，所以Android应用程序的自我保护具有一定的意义。本文总结了Android中可以使用的一些APK自我保护的技术，大部分都经过实际的代码测试。

## Dex

### Dex文件结构

classes.dex文件是Android系统运行于Dalvik Virtual Machine上的可执行文件，也是Android应用程序的核心所在，所以我们首先来看下DEX文件的结构，这样能够更好的理解后续的分析，需要更加详细的信息，可以参考[Google关于Dex的技术文档](http://source.android.com/devices/tech/dalvik/dex-format.html)。

从Java源文件（当然Android也支持JNI的调用方式）到生成Dex文件的基本映射关系如图 1所示，Java源文件通过Java编译器生成class文件，再通过dx工具转换为classes.dex文件。Dex文件从整体上来看是个索引的结构，类名、方法名、字段名等信息都存储在常量池中，这样能够充分减少存储空间，一个Dex文件的基本结构如图 2所示，相关结构声明定义在DexFile.h中，在AOSP中的路径为/dalvik/libdex/DexFile.h。

[![apk_protect_1](http://static.sanwho.com/uploads/2013/12/apk_protect_1.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_1.png)

图 1 Java源文件生成Dex文件的映射关系

   header: Dex文件头，包含magic字段、adler32校验值、SHA-1哈希值、string_ids的个数

 [![apk_protect_2](http://static.sanwho.com/uploads/2013/12/apk_protect_2.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_2.png)

图 2 Dex文件基本结构

以及偏移地址等。Dex文件头结构固定，占用0×70个字节，定义如下所示。




```C

    struct DexHeader {

        u1 magic[8]; /* includes version number */

        u4 checksum; /* adler32 checksum */

        u1 signature[kSHA1DigestLen]; /* SHA-1 hash */

        u4 fileSize; /* length of entire file */

        u4 headerSize; /* offset to start of next section */

        u4 endianTag;

        u4 linkSize;

        u4 linkOff;

        u4 mapOff;

        u4 stringIdsSize;

        u4 stringIdsOff;

        u4 typeIdsSize;

        u4 typeIdsOff;

        u4 protoIdsSize;

        u4 protoIdsOff;

        u4 fieldIdsSize;

        u4 fieldIdsOff;

        u4 methodIdsSize;

        u4 methodIdsOff;

        u4 classDefsSize;

        u4 classDefsOff;

        u4 dataSize;

        u4 dataOff;

};
```


   DexStringId: 定义了字符串数据的偏移， stringDataOff指向字符串数据；



```C
struct DexStringId {

        u4 stringDataOff; /* file offset to string_data_item */

    };
```



   DexTypeId: 表示应用程序代码中使用到的具体类型，如整型、字符串等，在Dalvik字节码中表示为I、Ljava/lang/String;，descriptorIdx指向DexStringId列表的索引；



```C
 struct DexTypeId {

        u4 descriptorIdx; /* index into stringIds list for type descriptor */

    };
```





   DexProtoId：表示方法声明的结构体，shortyIdx是方法声明字符串，格式为返回值类型后紧跟参数列表类型，如方法声明为VI，表示返回值为V（空，无返回值），参数为I（整型），所有的引用类型用L表示；returnTypeIdx指向DexTypeId列表的索引，表示返回值类型；parametersOff指向DexTypeList的偏移，表示参数列表类型；



```C
struct DexProtoId {

        u4 shortyIdx; /* index into stringIds for shorty descriptor */

        u4 returnTypeIdx; /* index into typeIds list for return type */

        u4 parametersOff; /* file offset to type_list for parameter types */

    };
```



   DexFieldId: 表示代码中的字段，classIdx指向DexTypeId列表索引，表示字段所属的类；typeIdx表示字段类型，nameIdx指向DexStringId列表索引，表示字段名；

```C
struct DexFieldId {

        u2 classIdx; /* index into typeIds list for defining class */

        u2 typeIdx; /* index into typeIds for field type */

        u4 nameIdx; /* index into stringIds for field name */

    };
```


   DexMethodId: 表示代码中使用的方法，classIdx表示方法所属的类，protoIdx指向DexProtoId列表索引，表示方法原型，nameIdx表示方法名；

```C
    struct DexMethodId {

        u2 classIdx; /* index into typeIds list for defining class */

        u2 protoIdx; /* index into protoIds for method prototype */

        u4 nameIdx; /* index into stringIds for method name */

    };
```


   DexClassDef: 该结构相对要复杂一些，定义了代码中的使用的类，以及相关的代码指令。





```C
    struct DexClassDef {

        u4 classIdx; /* index into typeIds for this class */

        u4 accessFlags;

        u4 superclassIdx; /* index into typeIds for superclass */

        u4 interfacesOff; /* file offset to DexTypeList */

        u4 sourceFileIdx; /* index into stringIds for source file name */

        u4 annotationsOff; /* file offset to annotations_directory_item */

        u4 classDataOff; /* file offset to class_data_item */

        u4 staticValuesOff; /* file offset to DexEncodedArray */

};

```

classIdx指向DexTypeId列表索引，表示该类的类型；accessFlags是类的访问标志，如public,private，static等；superclassIdx表示父类的类型；interfacesOff指向一个DexTypeList的偏移值，因为Java中可以实现多个接口，这里使用列表也就不难理解了；sourceFileIdx指向DexStringIdx列表的索引，表示类所在的源文件名称；annotationsOff指向注解目录结构；classDataOff指向DexClassData结构，表示类的数据部分；staticValuesOff表示类中的静态数据。



   DexClassData结构体定义在DexClass.h文件中，路径为/dalvik/libdex/DexClass.h，声明如下，header中包含静态字段个数，实例字段个数，直接方法（通过类直接访问的方法）个数，虚方法（通过类实例访问的方法）个数；

```C
 struct DexClassData {

        DexClassDataHeader header;

        DexField* staticFields;

        DexField* instanceFields;

        DexMethod* directMethods;

        DexMethod* virtualMethods;

    };

```

DexField表示字段的类型和访问标志， fieldIdx指向DexFieldId;

```C
struct DexField {

        u4 fieldIdx; /* index to a field_id_item */

        u4 accessFlags;

    };
```


DexMethod结构描述了方法的原型、名称、访问标志以及代码指令的偏移地址，methodIdx指向DexMethodId索引，需要注意的是在[Google的Dex文件文档](http://source.android.com/devices/tech/dalvik/dex-format.html)中对此的定义：



>index into the method_ids list for the identity of this method (includes the name and descriptor), represented as a difference from the index of previous element in the 
>list. The index of the first element in a list is represented directly.



注意红色字体部分，表示的是在Dex文件中，methodIdx是相对于前一个DexMethod中的methodIdx的增量，例如如果一个类中有两个directMethods，第一个directMethod的methodIdx值为0×13，表示指向索引为0×13的methodIdx，那么第二个directMethod的methodIdx的值是相对于前一个值的增量，例如0×01，表示指向索引为0×14的methodIdx；accessFlags为方法的访问标志，codeOff表示指令代码的偏移地址；

```C
  struct DexMethod {

        u4 methodIdx; /* index to a method_id_item */

        u4 accessFlags;

        u4 codeOff; /* file offset to a code_item */

    };
```


DexCode的结构体声明如下。



```C
   struct DexCode {

        u2 registersSize;

        u2 insSize;

        u2 outsSize;

        u2 triesSize;

        u4 debugInfoOff; /* file offset to debug info stream */

        u4 insnsSize; /* size of the insns array, in u2 units */

        u2 insns[1];

        /* followed by optional u2 padding */

        /* followed by try_item[triesSize] */

        /* followed by uleb128 handlersSize */

        /* followed by catch_handler_item[handlersSize] */

};

```

需要注意的是，在DexClass.h中，所有的u4类型，实际上是uleb128类型。每个uleb128类型是leb128的无符号类型，每个leb128类型的数据包含1-5个字节，表示一个32bit的数值。每个字节只有7位有效，最高一位用来表示是否需要使用到下一个字节，比如如果第一个字节最高位为1，表示还需要使用到第2个字节，如果第二个字节的最高位为1，表示会使用到第3个字节，以此类推，最多5个字节。对于一个2个字节的leb128类型数据，其结构如图 3所示。

[![apk_protect_3](http://static.sanwho.com/uploads/2013/12/apk_protect_3.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_3.png)



图 3 两字节的leb128类型数据格式

### Dex中方法的隐藏

此部分内容可参考[Playing Hide and Seek with Dalvik Executables](http://wikisec.free.fr/papers/hidex-hack.lu.pdf)。

前文分析了Dex文件的结构，根据Dex的文件结构，可以实现对Dex中特定方法的隐藏，这样在使用[baksamli](https://code.google.com/p/smali/)或者[apktool](https://code.google.com/p/android-apktool/)工具对classes.dex文件进行反汇编时，无法发现隐藏的方法，不过会有特定的现象发生，其实也是比较容易检测出来的。

在Dex文件格式分析中关于method的结构体是DexMethod，如果将methodIdx的值指向另一个method，同时修改相应的代码偏移量codeOff（accessFlags一般不需要修改），修改后续相应的methodIdx，则可以实现特定方法的隐藏。对Dex文件修改后需要重新计算Dex文件的SHA1值以及校验值，用来更新Dex文件。

隐藏方法的步骤如下：

       1、修改Dex文件中需要隐藏方法的DexMethod结构体，如图 4所示，图中隐藏了方法B。具体包括：

*      将DexMethod的methodIdx值设为0×0，相当于将原先的方法指向了前一个方法

*      访问标志符accessFlags一般不需要修改，在Dex文件格式里，directMethods和virtualMethods是分开的

*      将codeOffset设置为前一个方法的代码偏移地址

*      更新需隐藏方法的下一个方法的methodIdx，可以使用公式：next_method_idx=original_next_method_idx + original_method_idx

 [![apk_protect_4](http://static.sanwho.com/uploads/2013/12/apk_protect_4.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_4.png)

图 4 Dex方法隐藏

   2、重新计算Dex的SHA1哈希值和Adler校验值，并用以更新DexHeader，可以使用[DexFixer](http://pan.baidu.com/s/1jB5bu)修复classes.dex文件；

   3、重新打包生成APK文件：

*      将APK解压缩，提取其中出META-INF文件夹之外的所有文件

*      压缩成Zip格式文件

*      使用jarsigner或者其他工具对生成的Zip文件签名，后缀名修改成.apk。

隐藏的方法仍然需要在程序中进行调用，调用隐藏方法的步骤如下：

*      使用反射调用android.content.res.AssetManager.openNonAsset方法打开当前应用程序的classes.dex文件，将数据保存到内存中；还可以通过调用Context.getPackageCodePath()来获得当前应用程序对应的apk文件的路径，利用此路径构造ZipFile对象，进而获取classes.dex的ZipEntry，利用ZipFile的getInputStream(ZipEntry)方法获取classes.dex的数据流，核心代码如下所示；

```java
String apkPath = this.getPackageCodePath();

ZipFile apkfile = new ZipFile(apkPath);

ZipEntry dexentry = zipfile.getEntry("classes.dex");

InputStream dexstream = zipfile.getInputStream(dexentry);
```

* 修复Dex文件，将之前隐藏方法的DexMethod结构体恢复*

* 将修复后的Dex数据使用类加载器重新加载*

* 搜索被隐藏的方法*

* 调用被隐藏的方法*

需要注意的是，方法在Dex文件中是按方法名的字典序排序的，所以需要隐藏的方法如果是该类中所有方法排序第一个的话，那么methodIdx值是个绝对值，如果要隐藏的话就不是很方便，所以建议可以写个无用的方法，其方法名排序为第一个，让需要隐藏的方法重新指向该方法。

使用修改methodIdx的方法，让其指向另一个DexMethodId的结构体，如果使用baksmali进行反汇编，则会发现在一个类中有两个完全相同的函数。

那有没有更加隐蔽的手段来隐藏一个方法了？考虑到在DexClassData结构体中的DexClassDataHeader头部，其中directMethodsSize和virtualMethodsSize分别表示直接方法个数和虚方法个数，因此如果希望隐藏某个方法，可以通过将相应的directMethodsSize或virtualMethodsSize减1，同时将表示该需要隐藏方法的DexMethod结构体中的数据全部修改为0，这样就可以将该方法隐藏起来，使用baksmali反汇编时，不会显示出该方法的反汇编代码，具体可以参考[Hashdays 2012 Android Chanllenge](http://www.fortiguard.com/files/hashdayschallenge.pdf)。

当然，上述这两种隐藏方法，都没能隐藏掉DexMethodId结构体，这个结构体中包含了方法所属的类名、原型声明以及方法名，所以可以通过对比DexMethodId的个数和DexMethod结构体的个数来判断是否存在方法隐藏的问题。

### Dex完整性校验

classes.dex在Android系统上基本负责完成所有的逻辑业务，因此很多针对Android应用程序的篡改都是针对classes.dex文件的。在APK的自我保护上，也可以考虑对classes.dex文件进行完整性校验，简单的可以通过CRC校验完成，也可以检查Hash值。由于只是检查classes.dex，所以可以将CRC值存储在string资源文件中，当然也可以放在自己的服务器上，通过运行时从服务器获取校验值。基本步骤如下：

*      首先在代码中完成校验值比对的逻辑，此部分代码后续不能再改变，否则CRC值会发生变化

*      从生成的APK文件中提取出classes.dex文件，计算其CRC值，其他hash值类似

*      将计算出的值放入strings.xml文件中。

核心代码如下：



```java

String apkPath = this.getPackageCodePath();

Long dexCrc = Long.parseLong(this.getString(R.string.dex_crc));

try

{

    ZipFile zipfile = new ZipFile(apkPath);

    ZipEntry dexentry = zipfile.getEntry("classes.dex");

    if(dexentry.getCrc() != dexCrc){

        System.out.println("Dex has been modified!");

    }else{

        System.out.println("Dex hasn't been modified!");

    }

} catch (IOException e) {

             // TODO Auto-generated catch block

    e.printStackTrace();

}
```



但是上述的保护方式容易被暴力破解， 完整性检查最终还是通过返回true/false来控制后续代码逻辑的走向，如果攻击者直接修改代码逻辑，完整性检查始终返回true，那这种方法就无效了，所以类似文件完整性校验需要配合一些其他方法，或者有其他更为巧妙的方式实现？

### APK完整性校验

虽然Android程序的主要逻辑通过classes.dex文件执行，但是其他文件也会影响到整个程序的逻辑走向，以上述Dex文件校验为例，如果程序依赖strings.xml文件中的某些值，则修改这些值就会影响程序的运行，所以进一步可以整个APK文件进行完整性校验。但是如果对整个APK文件进行完整性校验，由于在开发Android应用程序时，无法知道完整APK文件的Hash值，所以这个Hash值的存储无法像Dex完整性校验那样放在strings.xml文件中，所以可以考虑将值放在服务器端。核心代码如下:


```java

 MessageDigest msgDigest = null;

    try {

      msgDigest = MessageDigest.getInstance("MD5")

      byte[] bytes = new byte[8192];

      int byteCount;

      FileInputStream fis = null;

      fis = new FileInputStream(new File(apkPath));

      while ((byteCount = fis.read(bytes)) &gt; 0)

         msgDigest.update(bytes, 0, byteCount);

       BigInteger bi = new BigInteger(1, msgDigest.digest());

       String md5 = bi.toString(16);

       fis.close();

      /*

       从服务器获取存储的Hash值，并进行比较

      */

      } catch (Exception e) {

         e.printStackTrace();

    }
```





### 动态加载

Java中可以使用反射技术来更加灵活地控制程序的运行，为Java运行时的行为提供了强大的支持。Android系统提供了DexClassLoader来支持在程序运行过程中动态加载包含classes.dex的.jar或者.apk文件，如果再结合Java反射技术，可以实现执行非应用程序部分的代码。利用动态加载技术，可以提供逆向分析的难度，在一定程度上可以保护APK自身的业务逻辑防止被破解。

DexClassLoader的构造函数原型如下：




1



```java

public DexClassLoader (String dexPath, String optimizedDirectory, String libraryPath, ClassLoader parent)

```



其中，dexPath为包含dex文件的.apk或者.jar路径，optimizedDirectory是优化后的dex文件的路径，libraryPath表示Native库的路径，parent是父类加载器。通过DexClassLoader实例化对象，调用loadClass加载需要调用的类，获得Class对象后，就可以进一步使用Java反射技术来调用相应的方法。如下：

```java

DexClassLoader classLoader = new DexClassLoader(apkPath, dexPath, null, getClassLoader());

    try {

    Class&lt;?&gt; mLoadClass = classLoader.loadClass("com.example.dexclassloaderslave.DexSlave");

    Constructor&lt;?&gt; constructor = mLoadClass.getConstructor(new Class[] {});

    Object dexSlave = constructor.newInstance(new Object[] {});

    Method sayHello = mLoadClass.getDeclaredMethod("sayHello", new Class[]{} );

    sayHello.setAccessible(true);

    sayHello.invoke(dexSlave, new Object[]{});

    } catch (Exception e)

   {

       e.printStackTrace();

    }


```



上述代码实现调用com.example.dexclassloaderslave.DexSlave类中的sayHello方法。



对于需要通过DexClassLoader被调用的.apk或者.jar文件的分发，可以将其放入Android项目的assets或者res目录下，也可以将其放在服务器端，在实际需要调用时通过网络获取文件。为了提高逆向的难度，可以对被调用的.apk或者.jar文件采取以下措施进行进一步的保护：

*      进行完整性校验，防止文件被篡改

*      进行加密处理，在调用加载前进行解密

*      对需要调用的函数相关信息使用通过网络获取的方式，而不是硬编码在代码中，可以真正实现动态调用，提高静态分析的难度

*      对于使用网络服务器分发的方式，注意对网络服务器地址的保护，不要以字符串硬编码的方式写在代码中，对下载请求也需要使用cookie等辅助识别的技术。

除了使用DexClassLoader类实现动态加载外，还可以使用dalvik.system.DexFile类实现Dex文件的加载，但是DexFile类提供的构造方法在实例化过程中需要在/data/davik-cache目录下生成相应的Dex文件，而/data/davik-cache目录对于一般应用程序是没有写权限的，所以在程序中无法实例化DexFile对象，也就无法调用DexFile.loadClass方法。所以需要通过反射调用DexFile类的openDex方法，具体可以参考[该代码](https://github.com/cryptax/dextools/blob/master/hidex/app/src/com/fortiguard/hideandseek/MrHyde.java)中invokeHidden函数。

## APK伪加密

APK实际上是Zip压缩文件，但是Android系统在解析APK文件时，和传统的解压缩软件在解析Zip文件时还是有所差异的，利用这种差异可以实现给APK文件加密的功能。Zip文件格式可以参考[MasterKey漏洞分析的一篇文章](http://justanapplication.wordpress.com/2013/07/13/the-great-android-security-hole-of-08-part-one-zip-files/)。在Central Directory部分的File Header头文件中，有一个2字节长的名为General purpose bit flags的字段，这个字段中每一位的作用可以参考[Zip文件格式规范](http://www.pkware.com/documents/casestudies/APPNOTE.TXT)的4.4.4部分，其中如果第0位置1，则表示Zip文件的该Central Directory是加密的，如果使用传统的解压缩软件打开这个Zip文件，在解压该部分Central Directory文件时，是需要输入密码的，如图 5所示。但是Android系统在解析Zip文件时并没有使用这一位，也就是说这一位是否置位对APK文件在Android系统的运行没有任何影响。一般在逆向APK文件时，会首先使用[apktool](https://code.google.com/p/android-apktool/)来完成资源文件的解析，dex文件的反汇编工作，但如果将Zip文件中Central Directory的General purpose bit flags第0位置1的话，apktool(version:1.5.2)将无法完成正常的解析工作，如图 6所示，但是又不会影响到APK在Android系统上的正常运行，如图 7所示。

 [![apk_protect_5](http://static.sanwho.com/uploads/2013/12/apk_protect_5.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_5.png)

图 5 传统解压缩软件需要输入密码进行解压缩

 [![apk_protect_6](http://static.sanwho.com/uploads/2013/12/apk_protect_6.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_6.png)

图 6 apktool解析伪加密的APK文件失败

       对APK文件进行伪加密可以使用[这个脚本](https://github.com/MindMac/Android-Fake-Encryption)，在Python的zipfile模块中，ZipInfo类中记录了Zip文件中相应的Central Directory的相关信息，包括General purpose bit flags，在ZipInfo类中属性为flag_bits，因此上述脚本中将需加密的APK文件的每个ZipInfo的flag_bits和1做或操作，实现在General purpose bit flags的第0位置1.

而需要去除这些伪加密的标志的话，可以使用[这个脚本](https://github.com/blueboxsecurity/DalvikBytecodeTampering/blob/master/unpack.py)。相关内容可以参考BlueBox之前提出的一个[Android Security Analysis Chanllenge.](http://bluebox.com/labs/android-security-challenge/)。

[![apk_protect_7](http://static.sanwho.com/uploads/2013/12/apk_protect_7.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_7.png)

图 7 伪加密的APK可以正常运行

 **Manifest Cheating**

AndroidManifest.xml是Android应用程序的配置文件，包含了包名、应用程序名称、申请的权限信息以及组件信息等。在Android应用程序开发，生成APK时，aapt会负责完成资源的打包，打包会将文本格式的XML资源文件编译成二进制格式的XML资源文件。将文本格式的XML文件转换成二进制格式，一方面通过字符串资源池的统一管理，减少文件体积；另一方面二进制格式的XML文件解析速度也会更快。在Android开发过程中，生成的R.java文件中包含了相应的资源类型、名称以及对应的id值。资源id是32bit的整型值，格式为:0xPPTTNNNN。其中PP表示使用该资源的包，TT代表该资源的类型，而NNNN是该类型中资源的名称。对于应用程序资源，PP值固定为7f，而对于被引用的系统资源包，其PP值为01。TT和NNNN一般是aapt按照资源出现的顺序生成的。更多分析可以参考罗升阳的[Android应用程序资源的编译和打包过程分析](http://blog.csdn.net/luoshengyang/article/details/8744683)。

[Manifest Cheating](http://bluebox.com/wp-content/uploads/2013/05/AndroidREnDefenses201305.pdf)的基本原理是，在AndroidManifest的&lt;application&gt;节点中插入一个未知id(如0×0)，名称为name的属性，其值可以是一个从未定义实现的Java类文件名。而对AndroidManifest的修改需要在二进制格式下进行，这样才能不会破坏之前aapt对资源文件的处理。由于是未知的资源id，在应用程序运行过程中，Android会忽略此属性。但是在使用apktool进行重打包时，首先会将AndroidManifest.xml转换为明文，进而会包含名称为name的属性，而相应的id信息会丢失，apktool重打包会重新进行资源打包处理，由于该name属性值是一个未实现的Java类，重打包后的应用程序在运行过程中，由于application节点中定义的类是先于所有其他组件运行的，若系统找不到对应的类，会出现运行时错误，Dalvik虚拟机会直接关闭。另外，也可以实现name属性值对应的Java类，若此类被调用，则表明被重打包了，可以采取进一步的措施。这样就可以起到保护自身APK的作用，防止被重打包。但是这种方法也很容易被绕过，只需要在经过apktool解码的AndroidManifest文件中，去掉在application节点中添加的name属性即可。整个过程如下：

*      将APK解压缩，提取其中的AndroidManifest.xml文件

*      使用[axml](https://code.google.com/p/axml/)工具，修改二进制的AndroidManifest.xml文件，在application节点下插入id未知(如0×0)，名为name的属性(值可以任意，只要不对应到项目中的类文件名即可，如some.class)

*      将除META-INF文件夹之外的文件压缩成zip文件，签名后生成.apk文件。

若是攻击者使用apktool重打包，运行重打包后的文件会出现如下运行时错误： [![apk_protect_8](http://static.sanwho.com/uploads/2013/12/apk_protect_8-710x209.png)](http://static.sanwho.com/uploads/2013/12/apk_protect_8.png)

图 8 使用Manifest Cheating重打包后APK文件运行时错误

## 调试器检测

在对APK逆向分析时，往往会采取动态调试技术，可以使用[netbeans+apktool](http://d-kovalenko.blogspot.com/2012/08/debugging-smali-code-with-apk-tool-and.html)对反汇编生成的smali代码进行动态调试。为了防止APK被动态调试，可以检测是否有调试器连接。Android系统在android.os.Debug类中提供了isDebuggerConnected()方法，用于检测是否有调试器连接。可以在Application类中调用isDebuggerConnected()方法，判断是否有调试器连接，如果有，直接退出程序。

除了isDebuggerConnected方法，还可以通过在AndroidManifest文件的application节点中加入android:debuggable=”false”使得程序不可被调试，这样如果希望调试代码，则需要修改该值为true，因此可以在代码中检查这个属性的值，判断程序是否被修改过，代码如下：



```java
   if(getApplicationInfo().flags&= ApplicationInfo.FLAG_DEBUGGABLE != 0){

       System.out.println("Debug");

       android.os.Process.killProcess(android.os.Process.myPid());

    }
```



##代码混淆##

使用Java编写的代码很容易被反编译，因此可以使用代码混淆的方法增加反编译代码阅读的难度。[ProGuard](http://proguard.sourceforge.net/)是一款免费的Java代码混淆工具，提供了文件压缩、优化、混淆和审核功能。在Eclipse+ADT开发环境下，每个Android应用程序项目目录下会默认生成project.properties和proguard-project.txt文件。如果需要使用ProGuard进行压缩以及混淆，首先需要在project.properties文件中去掉对如下语句的注释：


```
proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt
```



ProGuard的相关配置信息需要在proguard-project.txt文件中声明，在其中可以设置需要混淆和保留的类或方法。由于在某些情况下，ProGuard会错误地认为某些代码没有被使用，如在只在AndroidManifest文件中引用的类，从JNI中调用的方法等。对于这些情况，需要在proguard-project.txt文件中添加-keep命令，用来保留类或方法。关于ProGuard更加详细的配置项可以参考[ProGuard Manual](http://proguard.sourceforge.net/index.html#manual/index.html)。

除了使用ProGuard对Android代码进行混淆外，还可以使用[DexGuard](http://www.saikoa.com/dexguard)。DexGuard是特别针对Android的一款代码优化混淆的收费软件，提供代码优化混淆、字符串加密、类加密、Assets资源加密、隐藏对敏感API的调用、篡改检测以及移除Log代码。

关于代码混淆，还可以参考[Android:Game of Obfuscation](http://androidxref.com/files/bremer_chiossi_h2hc2013.pdf)。

## NDK

Android软件的开发主要使用Java语言，但是Android也提供了对本地语言C、C++的支持。借助JNI，可以在Java类中使用C语言库中的特定函数，或在C语言程序中使用Java类库。一般来说，如果代码中对处理速度有较高要求或者为了更好地控制硬件，抑或者为了复用既有的C/C++代码，都可以考虑通过JNI来实现对Native代码的调用。

由于逆向Native程序的汇编代码要比逆向Java汇编代码困难，因此可以考虑在关键代码部位使用Native代码，如注册验证，加解密操作等。一个可能的借助Native代码保护APK的方法是：将核心业务逻辑代码放入加密的.jar或者.apk文件中，在需要调用时使用Native代码进行解密，同时完成对解密后文件的完整性校验，不过不管是.jar还是.apk文件，解密后都会留在物理存储上，为了避免这种情况，可以使用反射技术直接调用dalvik.system.DexFile.openDex()方法，该方法接受classes.dex文件字节流返回DexFile对象。关于Native代码的编写，可以参考Google官方文档的[Android NDK](http://developer.android.com/tools/sdk/ndk/index.html)。

## 逆向工具对抗

在逆向分析Android应用程序时，一般会使用apktool，baksmali/smali，dex2jar，androguard，jdGUI以及IDA Pro等。因此可以考虑使得这些工具在反编译APK时出错来保护APK，这些工具大部分都是开源的，可以通过阅读其源代码，分析其在解析APK、dex等文件存在的缺陷，在开发Android应用程序时加以利用。可以参考Tim Strazzere的[Dex Education:Practicing Safe Dex](http://www.strazzere.com/papers/DexEducation-PracticingSafeDex.pdf)，相应的[Demo](https://github.com/strazzere/APKfuscator)，看雪上的[中文翻译](http://bbs.pediy.com/showthread.php?t=177114)，不过其中的很多技巧已经失效了。DexLabs的[Dalvik Bytecode Obfuscation on Android](http://dexlabs.org/blog/bytecode-obfuscation)介绍了垃圾字节码插入的技术。

## 总结

以上APK自我保护的技术并不能做到完全的保护作用，只是提高了逆向分析的难度，在实际运用中应该根据情况多种技术结合使用。这些技术其实很多来源于Android恶意代码，所以可以关注Android恶意代码中使用的一些技术来应用到自己开发的Android应用程序中。


---