---
title: CsvHelper 使用手册
date: 2018-09-04
tags:
---
# CsvHelper 使用手册

## 导引

### 安装

在包管理控制台

```shell
PM> Install-Package CsvHelper
```

.NET CLI 控制台

```shell
>dotnet add package CsvHelper
```

### 读一个 CSV 文件

首先创建一个这样的 CSV 文件

```
ID,Name
1,one
2,two
```

做一个类的定义如下

```csharp
public class Foo
{
    public int ID{get;set;}
    public string Name{get;set;}
}
```

如果我们创建的类的属性名能够匹配目标 CSV 文件的表头，那么我们就无需任何配置的读取这个文件。

```csharp
using (var reader = new StreamReader("Path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    var records = csv.GetRecords<Foo>();
}
```

这个 `GetRecords<T>`  方法将会返回一个 `IEnumerable<T>`将会`yield`  records。这也就意味着当你在反复查询记录的时候一次只能返回一条，即仅仅只有文件的一小部分会被读到内存中。不过要小心的是，如果你做了任何关于 LINQ 投影的事情，就像调用 `.ToList()`，整个文件都会被读入到内存中。 `CsvReader`只能向前走，所以当你想要运行任何 LINQ 查询来防范你的数据，你需要知道如果这样做的话，整个文件会被加载到内存中。

加入我们对前面的 CSV 文件做了一点点改变，使它与之前的属性不是完全匹配了。

```
id,name
1,one
2,two
```

在此次改动中，我们把名字都用小写字母替代了。由于我们之前设置的属性名能是驼峰式的，这样我们就可以改变属性头与表格头的匹配方式了。

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    csv.Configuration.PrepareHeaderForMatch = (string header,int index) => header.ToLower();
    var records = csv.GetRecords<Foo>();
}
```

使用配置 `PrepareHeaderForMatch`，我们就能够实现不同名称之间的配对。头名和属性名都包含在 `PrepareHeaderForMatch`函数中。当 reader 需要使用属性名来设置头名的时候，他们将会匹配。你还能够使用这个函数来做一些其他的事情，比如说空格或者其他的一些字符。

那我们再来看看如果我们去掉 CSV 文件的头名怎么破吧。

```
1,one
2,two
```

首先我们需要告诉 reader 文件中已经没有头记录了，配置如下

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    csv.Configuration.HasHeadRecord = false;
    var record = csv.GetRecords<Foo>();
}
```

CsvReader 将会使用类中属性的位置作为索引点。但是这有一个问题，你不能再依靠 .NET 中类成员的顺序了。解决方法就是将这个属性映射到 CSV 文件的特定位置。

一种方法就是用属性映射。

```csharp
public class Foo
{
    [Index(0)]
    public int Id{get;set;}
    
    [Index(1)]
    public string Name{get;set;}
}
```

这个 `IndexAttribute`允许你指定你想要使用属性的位置。

你还可以使用名字作为映射，让我们使用前面的小写头部的例子来看看怎么使用名字匹配。

```csharp
public class Foo
{
    [Name("id")]
    public int Id{get;set;}
    
    [Name("name")]
    public string Name{get;set;}
}
```

有许多的属性你同样可以使用。

如果我们无法操作我们做匹配的这个类来增加我们需要的属性怎么办？在这个例子中，我们会使用 `ClassMap`做一次流利的匹配。

```csharp
public class FooMap:ClassMap<Foo>
{
    public FooMap()
    {
		Map(M => m.Id).Name("id");
		Map(m => m.Name).Name("name");
    }
}
```

为了来使用映射，我们需要注册配置。（也就是写一下配置方法）

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    csv.Configuration.RegisterClassMap<FooMap>();
    var records = csv.GetRecords<Foo>();
}
```

推荐创造一个类映射，因为这样的话使用 CsvHelper 会更加强大。

### 写一个 CSV 文件

现在让我们来看一看怎么写一个 CSV 文件吧，这跟读基本上是一样的，只是顺序相反。

跟之前读文件一样，我们用一样的类定义。

```csharp
public class Foo
{
    public int Id{get;set;}
    public string Name{get;set;}
}
```

然后我们创建一些这样的记录（Records）

```csharp
var records = new List<Foo>
{
	new Foo{Id=1,Name="one"},
	new Foo{Id=2,Name="two"},
};
```

我们可以无需配置的把这些记录写到文件中。

```csharp
using (var writer = new StreamWriter("Path\\to\\file.csv"))
using (var csv = new CsvWriter(writer))
{
    csv.WriteRecords(records);
}
```

这个 `WriteRecords`方法将会把所有的记录都写到文件中，在你完成写数据之后，你应该调用 `writer.Flush()`来确保写入器内部缓冲区中的所有数据都已被刷新到文件中。在 `using`块中的缓存器会自动被清空，因此我们并不需要刻意的去处理这个块。使用 `using`块来包含 `IDisposable`对象是一种比较好的方式，这个对象会在 `using`块退出之后 自己做出相应的处理（在我们这个例子中会自动清除缓存）。

记得我们是不能在 .NET 里面依赖属性的顺序的吗？如果我们写一个有标头的类的时候，这并不重要，我们只需要在后面使用标头即可。如果我们想要将标头安置在 CSV 文件的相应位置的时候，我们就需要指定索引来保证它的顺序，所以当你在写入 CSV 文件数据的时候设置索引是一个好习惯。

```csharp
public class FooMap:classMap<Foo>
{
	public FooMap()
    {
    	Map(m=>m.Id).Index(0).Name("id");
    	Map(m=>m.Name).Index(1).Name("name");
    }
}
```

## 举例说明

### 预备知识

下面是使用 CsvHelper 需要知道预备知识。

这里有一些关于 .NET 的基础知识是你使用 CsvHelper 前需要知道的，微软有一个[很棒的文档](https://docs.microsoft.com/zh-cn/dotnet/)能够帮助你学习更多。

#### 使用和释放

不论何时你什么时候创建一个 `IDisposable` 的对象，你都要在使用资源后释放它，大多数类使用非托管资源来是实现 `IDisposable`，这也就意味着在 `System.IO`命名空间里的类都需要被释放。

最好的练习释放对象的方法就是使用 `using`块中写代码，因为在 `using` 块中，资源会快速自动的被处理。

```csharp
using (var stream = new MemoryStream())
{
    //在这里使用 stream 对象
}
//stream 对象将会在这里被快速的处理
```

如果你在后面也需要用到这个对象，并且稍后会释放它，那么 使用`using`会帮你做一些错误处理，因此使用 `using`相比于直接调用 `Dispose`依旧是一个更好的选择。但是关于这个目前仍然有一些争论，因为有人觉得它没有展现出使用意图。

```csharp
var stream = new MemoryStream();
//之后其他部分的代码
using (stream){	}
```

#### 读写文件

`System.IO.File`组件包含有打开文件进行读写的方法。

```csharp
using (var stream = File.OpenRead("path\\to\\file.csv"))
{
    
}

using (var stream = File.OpenWrite("path\\to\\file.csv"))
{
    
}
```

这些东西同样返回一个 `FileStream`来为操作文件进行服务。加入我们的数据是文本型的，我们就需要 `StreamReader`和 `StreamWriter`来读写文本。

```csharp
using (var stream = File.OpenRead("path\\to\\file.csv"))
using (var reader = new StreamReader(stream))    
{
    
}

using (var stream = File.OpenRead("path\\to\\file.csv"))
using (var writer = new StreamWriter(stream))    
{
    
}
```

`StreamReader`和 `StreamWriter`都有一种简便写法。

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
{
    
}
using (var writer = new StreamWriter("path\\to\\file.csv"))
{
    
}
```

因为 CsvHelper 并不知道你文件数据的具体编码，所以当你有一个特殊的编码的时候。你需要再你的流（stream）里面指定它。

```csharp
using (var reader = new StremaReader("path\\to\\file.csv"),Encoding.UTF8)
{
    
}
using (var writer = new StreamWriter("path\\to\\file.csv",Encoding.UTF8))
{
    
}
```

`CsvReader` 和 `CsvWriter`的构造函数分别是 `TextReader` 和 `TextWriter`，`TextReader` 和 `TextWriter`都是读写文本的一个抽象类。`StreamReader`和 `StreamWriter`都继承自 `TextReader` 和 `TextWriter`，因此我们也把它们和`CsvReader，CsvWriter`一起使用。

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
}

using (var writer = new StreamWriter("path\\to\\file.csv"))
using (var csv = new CsvWriter(writer))
{    
}
```

### 读 CSV 文件

#### 获取类记录

将 CSV 插入到对应类的对象中。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

#### 获取动态记录

将 CSV  插入到 `dynamic`对象中，由于无法判断属性的类型，所以动态对象上的所有属性都是字符串。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        var records = csv.GetRecords<dynamic>();
    }
}
```

#### 获取匿名类型的记录

如果你需要将 CSV 插入到匿名类型的对象中，仅仅只要提供匿名类型的定义即可。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        var anonymousTypeDefinition = new
        {
            Id = default(int),
            Name = string.Empty
        };
        var records = csv.GetRecords(anonymousTypeDefinition);
    }
}
```

#### 枚举类型记录

将 CSV 转换为一个类对象，该对象可在枚举的每次迭代中重用。每个枚举将生成给定的记录，但只生成映射的成员。如果你提供一个映射却没有映射其中的一个成员，那么该成员就不会得到当前行的数据。值得注意的是，对于你在工程中调用的任何方法会被强制返回一个 `IEnumerable` 的值，就像方法 `ToList()` ，你会得到一个与所有记录的实例与你在 CSV 中最后一条记录相对应的的列表。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        var record = new Foo();
        var records = csv.EnumerateRecords(record);
        foreach (var r in records)
        {
            // r is the same instance as record.
        }
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

#### 手动读取

因为一些原因，不去配置一个与你的类定义一一对应的映射会变得更加容易，只需要再多纪杭代码就可手动实现行的读取。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        var records = new List<Foo>();
        csv.Read();
        csv.ReadHeader();
        while (csv.Read())
        {
            var record = new Foo
            {
                Id = csv.GetField<int>("Id"),
                Name = csv.GetField("Name")
            };
            records.Add(record);
        }
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

#### 读取多样的数据集

因为某些原因，CSV 里面会包含多类型的混合数据集。就像这样读就没有什么问题了，当你检索数据的时候，你需要相应的改变类的类型。

数据

```
FooId,Name
1,foo

BarId,Name
07a0fca2-1b1c-4e44-b1be-c2b05da5afc7,bar
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.IgnoreBlankLines = false;
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.Configuration.RegisterClassMap<BarMap>();
        var fooRecords = new List<Foo>();
        var barRecords = new List<Bar>();
        var isHeader = true;
        while (csv.Read())
        {
            if (isHeader)
            {
                csv.ReadHeader();
                isHeader = false;
                continue;
            }

            if (string.IsNullOrEmpty(csv.GetField(0)))
            {
                isHeader = true;
                continue;
            }

            switch (csv.Context.HeaderRecord[0])
            {
                case "FooId":
                    fooRecords.Add(csv.GetRecord<Foo>());
                    break;
                case "BarId":
                    barRecords.Add(csv.GetRecord<Bar>());
                    break;
                default:
                    throw new InvalidOperationException("Unknown record type.");
            }
        }
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Bar
{
    public Guid Id { get; set; }
    public string Name { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id).Name("FooId");
        Map(m => m.Name);
    }
}

public sealed class BarMap : ClassMap<Bar>
{
    public BarMap()
    {
        Map(m => m.Id).Name("BarId");
        Map(m => m.Name);
    }
}
```

#### 读取多种记录类型

如果你的 CSV 文件中每行都有不同的记录类型，你应该读基于行的类型。

数据

```
A,1,foo
B,07a0fca2-1b1c-4e44-b1be-c2b05da5afc7,bar
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.HasHeaderRecord = false;
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.Configuration.RegisterClassMap<BarMap>();
        var fooRecords = new List<Foo>();
        var barRecords = new List<Bar>();
        while (csv.Read())
        {
            switch (csv.GetField(0))
            {
                case "A":
                    fooRecords.Add(csv.GetRecord<Foo>());
                    break;
                case "B":
                    barRecords.Add(csv.GetRecord<Bar>());
                    break;
                default:
                    throw new InvalidOperationException("Unknown record type.");
            }
        }
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Bar
{
    public Guid Id { get; set; }
    public string Name { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id).Index(1);
        Map(m => m.Name).Index(2);
    }
}

public sealed class BarMap : ClassMap<Bar>
{
    public BarMap()
    {
        Map(m => m.Id).Index(1);
        Map(m => m.Name).Index(2);
    }
}
```

### 写 CSV 文件

#### 写类对象

例子

```csharp
void Main()
{
    var records = new List<Foo>
    {
        new Foo { Id = 1, Name = "one" },
    };

    using (var writer = new StreamWriter("path\\to\\file.csv"))
    using (var csv = new CsvWriter(writer))
    {
        csv.WriteRecords(records);
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}

```

输出

```
Id,Name
1,one
```

#### 写动态类对象

例子

```csharp
void Main()
{
    var records = new List<dynamic>();

    dynamic record = new ExpandoObject();
    record.Id = 1;
    record.Name = "one";
    records.Add(record);

    using (var writer = new StringWriter())
    using (var csv = new CsvWriter(writer))
    {
        csv.WriteRecords(records);

        writer.ToString().Dump();
    }
}
```

输出

```
Id,Name
1,one
```

#### 写匿名类型对象

例子

```csharp
void Main()
{
    var records = new List<object>
    {
        new { Id = 1, Name = "one" },
    };

    using (var writer = new StreamWriter("path\\to\\file.csv"))
    using (var csv = new CsvWriter(writer))
    {
        csv.WriteRecords(records);
    }
}
```

输出

```
Id,Name
1,one
```

### 配置

#### 类映射

##### 映射属性

映射到属性。这个将会把类的属性映射到 CSV 数据的标头名字上，映射需要在配置中被注册，例子等价于一点也不使用类映射，用标头来匹配类名。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {        
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }    
    public string Name { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
    }
}
```

##### 通过名称映射

通过标头名映射到属性，如果你的属性名不匹配你的类名，那么你就可以通过名字来映射到属性。

数据

```
Column1,Column2
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id).Name("ColumnA");
        Map(m => m.Name).Name("ColumnB");
    }
}
```

##### 通过可替换的名映射

多标头名映射至属性，如果你有一个可以改变的标头名，你就可以指定多种头名。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id).Name("TheId", "Id");
        Map(m => m.Name).Name("TheName", "Name");
    }
}
```

##### 映射复制名

映射已经复制标头名的属性，有时候你复制了头名，这时候会通过标题名称索引来处理。name 索引是标头名称出现次数的索引，而不是标头位置的索引。

数据

```
Id,Name,Name
1,first,last
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string FirstName { get set; }
    public string LastName { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.FirstName).Name("Name").NameIndex(0);
        Map(m => m.LastName).Name("Name").NameIndex(1);
    }
}
```

##### 通过索引映射

通过标头的索引位置映射属性，如果你的数据不包含标头，你就可以使用索引来映射数据。不能通过 .NET 类的属性的顺序来，所以如果你没有使用名称进行映射，确保你指定了索引。

数据

```
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.HasHeaderRecord = false;
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id).Index(0);
        Map(m => m.Name).Index(1);
    }
}
```

##### 自动映射

自动映射，如果你没有提供映射配置，你可以直接调用在类中的自动配置，组件会自动帮你创建一个映射。这对于你有比较多数量的属性而言是一个很好的方式，因为他会自动帮你正确的设置好，你需要做的就是对代码做一些小改变。

数据

```
Id,The Name
1,one
```

例子

```csharp
void Main()
{       
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        AutoMap();
        Map(m => m.Name).Name("The Name");
    }
}
```

##### 忽略属性

忽略映射属性，当你使用自动映射类方法的时候，每个属性都会被映射，如果那里有你不想要映射的属性，你就能够直接忽视他们了。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{       
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsDirty { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        AutoMap();
        Map(m => m.IsDirty).Ignore();
    }
}
```

##### 常数值

对于特定的属性设置常值，你能够设置常值属性而不是映射到域。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{       
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        var records = csv.GetRecords<Foo>();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsDirty { get; set; }
}

public sealed class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.IsDirty).Constant(true);
    }
}
```

##### 类型转换

使用指定的类型转换，如果你需要从非标准的 .NET 类型转换或者转换到非标准的 .NET 数据，你能够提供一个类型转化来使用属性。

数据

```
Id,Name,Json
1,one,"{ ""Foo"": ""Bar"" }"
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.GetRecords<Foo>().ToList().Dump();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Json Json { get; set; }
}

public class Json
{
    public string Foo { get; set; }
}

public class JsonConverter<T> : DefaultTypeConverter
{
    public override object ConvertFromString(string text, IReaderRow row, MemberMapData memberMapData)
    {
        return JsonConvert.DeserializeObject<T>(text);
    }

    public override string ConvertToString(object value, IWriterRow row, MemberMapData memberMapData)
    {
        return JsonConvert.SerializeObject(value);
    }
}

public class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.Json).TypeConverter<JsonConverter<Json>>();
    }
}
```

##### 内联类型转化

转化到一个内联类型，如果你不想要写一个满的 `ITypeConverter`实现，你能够创建一个可以实现功能的函数。

###### 读

数据

```
Id,Name,Json
1,one,"{ ""Foo"": ""Bar"" }"
```

例子

```
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.GetRecords<Foo>().ToList().Dump();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Json Json { get; set; }
}

public class Json
{
    public string Foo { get; set; }
}

public class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.Json).ConvertUsing(row => JsonConvert.DeserializeObject<Json>(row.GetField("Json")));
    }
}
```

###### 写

例子

```csharp
void Main()
{
    var records = new List<Foo>
    {
        new Foo { Id = 1, Name = "one" }
    };

    using (var writer = new StreamWriter("path\\to\\file.csv"))
    using (var csv = new CsvWriter(writer))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.WriteRecords(records);

        writer.ToString().Dump();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Json Json { get; set; }
}

public class Json
{
    public string Foo { get; set; }
}

public class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.Json).ConvertUsing(o => JsonConvert.SerializeObject(o));
    }
}
```

输出

```
Id,Name,Json
1,one,"{""Id"":1,""Name"":""one"",""Json"":null}"
```

##### 映射选项

属性一定要存在才能映射，如果你有数据不确定是不是有标头，你需要制作映射选项。

数据

```
Id,Name
1,one
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.GetRecords<Foo>().ToList().Dump();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTimeOffset? Date { get; set; }
}

public class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.Date).Optional();
    }
}
```

##### 验证

验证一个域值，如果你想要确保你的数据符合一些标准，你能够验证它。

数据

```
Id,Name
1,on-e
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.Configuration.RegisterClassMap<FooMap>();
        csv.GetRecords<Foo>().ToList().Dump();
    }
}

public class Foo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTimeOffset? Date { get; set; }
}

public class FooMap : ClassMap<Foo>
{
    public FooMap()
    {
        Map(m => m.Id);
        Map(m => m.Name).Validate(field => !field.Contains("-"));
    }
}
```

#### 属性

大部分的配置都能通过使用属性类映射被完成，[完整的可获得的属性列表](https://joshclose.github.io/CsvHelper/api/CsvHelper.Configuration.Attributes/)。

数据

```
Identifier,name,IsBool,Constant
1,one,yes,a
2,two,no,b
```

例子

```csharp
void Main()
{
    using (var reader = new StreamReader("path\\to\\file.csv"))
    using (var csv = new CsvReader(reader))
    {
        csv.GetRecords<Foo>().ToList().Dump();
    }
}

public class Foo
{
    [Name("Identifier")]
    public int Id { get; set; }

    [Index(1)]
    public string Name { get; set; }

    [BooleanTrueValues("yes")]
    [BooleanFalseValues("no")]
    public bool IsBool { get; set; }

    [Constant("bar")]
    public string Constant { get; set; }

    [Optional]
    public string Optional { get; set; }

    [Ignore]
    public string Ignored { get; set; }    
}
```

#### 类型转换

待补...

#### 数据表

使用 CsvHelper 去加载数据表是非常频繁的事情，因此我直接将其集成了一个功能。

CsvDataReader 实现  `IDataReader`方法。，这也就意味着它仅有前向数据读取的所有功能，所以真的不必要去直接使用这个类而不用 `CsvReader`， `CsvDataReader`要求有 `CsvReader`的实例并且在内部使用来完成其功能。

使用 CsvHelper 来加载一个 `DataTable`是比较简单的，默认的，表格中的所有列将会以字符串的形式被加载。当读取器已经做好了实例化的准备，你只需要在创建 CsvDataReader实例前做一些配置即可。

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    // Do any configuration to `CsvReader` before creating CsvDataReader.
    using (var dr = new CsvDataReader(csv))
    {        
        var dt = new DataTable();
        dt.Load(dr);
    }
}
```

如果你想要指定行和行的类型，数据表也可以进行自动的类型转换。

```csharp
using (var reader = new StreamReader("path\\to\\file.csv"))
using (var csv = new CsvReader(reader))
{
    // Do any configuration to `CsvReader` before creating CsvDataReader.
    using (var dr = new CsvDataReader(csv))
    {        
        var dt = new DataTable();
        dt.Columns.Add("Id", typeof(int));
        dt.Columns.Add("Name", typeof(string));

        dt.Load(dr);
    }
}
```

### 应用程序接口（API）

#### 命名空间（CsvHelper Namespace）

##### 类（Classes）

|                                                              |                                                              |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [BadDataException](https://joshclose.github.io/CsvHelper/api/CsvHelper/BadDataException) | Represents errors that occur due to bad data.                |
| [CsvDataReader](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvDataReader) | Provides a means of reading a CSV file forward-only by using CsvReader. |
| [CsvFieldReader](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvFieldReader) | Reads fields from a `System.IO.TextReader` .                 |
| [CsvHelperException](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvHelperException) | Represents errors that occur in CsvHelper.                   |
| [CsvParser](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvParser) | Parses a CSV file.                                           |
| [CsvReader](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvReader) | Reads data that was parsed from `CsvHelper.IParser` .        |
| [CsvSerializer](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvSerializer) | Defines methods used to serialize data into a CSV file.      |
| [CsvWriter](https://joshclose.github.io/CsvHelper/api/CsvHelper/CsvWriter) | Used to write CSV files.                                     |
| [Factory](https://joshclose.github.io/CsvHelper/api/CsvHelper/Factory) | Creates CsvHelper classes.                                   |
| [FieldValidationException](https://joshclose.github.io/CsvHelper/api/CsvHelper/FieldValidationException) | Represents a user supplied field validation failure.         |
| [HeaderValidationException](https://joshclose.github.io/CsvHelper/api/CsvHelper/HeaderValidationException) | Represents a header validation failure.                      |
| [MissingFieldException](https://joshclose.github.io/CsvHelper/api/CsvHelper/MissingFieldException) | Represents an error caused because a field is missing in the header while reading a CSV file. |
| [ObjectResolver](https://joshclose.github.io/CsvHelper/api/CsvHelper/ObjectResolver) | Creates objects from a given type.                           |
| [ParserException](https://joshclose.github.io/CsvHelper/api/CsvHelper/ParserException) | Represents errors that occur while parsing a CSV file.       |
| [ReaderException](https://joshclose.github.io/CsvHelper/api/CsvHelper/ReaderException) | Represents errors that occur while reading a CSV file.       |
| [ReadingContext](https://joshclose.github.io/CsvHelper/api/CsvHelper/ReadingContext) | CSV reading state.                                           |
| [RecordBuilder](https://joshclose.github.io/CsvHelper/api/CsvHelper/RecordBuilder) | Builds CSV records.                                          |
| [ReflectionExtensions](https://joshclose.github.io/CsvHelper/api/CsvHelper/ReflectionExtensions) | Extensions to help with reflection.                          |
| [ValidationException](https://joshclose.github.io/CsvHelper/api/CsvHelper/ValidationException) | Represents a user supplied validation failure.               |
| [WriterException](https://joshclose.github.io/CsvHelper/api/CsvHelper/WriterException) | Represents errors that occur while writing a CSV file.       |
| [WritingContext](https://joshclose.github.io/CsvHelper/api/CsvHelper/WritingContext) | CSV writing state.                                           |

##### 接口（Interfaces）

|                                                              |                                                              |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [IFactory](https://joshclose.github.io/CsvHelper/api/CsvHelper/IFactory) | Defines methods used to create CsvHelper classes.            |
| [IFieldReader](https://joshclose.github.io/CsvHelper/api/CsvHelper/IFieldReader) | Defines methods used to read a field in a CSV file.          |
| [IObjectResolver](https://joshclose.github.io/CsvHelper/api/CsvHelper/IObjectResolver) | Defines the functionality of a class that creates objects from a given type. |
| [IParser](https://joshclose.github.io/CsvHelper/api/CsvHelper/IParser) | Defines methods used the parse a CSV file.                   |
| [IReader](https://joshclose.github.io/CsvHelper/api/CsvHelper/IReader) | Defines methods used to read parsed data from a CSV file.    |
| [IReaderRow](https://joshclose.github.io/CsvHelper/api/CsvHelper/IReaderRow) | Defines methods used to read parsed data from a CSV file row. |
| [ISerializer](https://joshclose.github.io/CsvHelper/api/CsvHelper/ISerializer) | Defines methods used to serialize data into a CSV file.      |
| [IWriter](https://joshclose.github.io/CsvHelper/api/CsvHelper/IWriter) | Defines methods used to write to a CSV file.                 |
| [IWriterRow](https://joshclose.github.io/CsvHelper/api/CsvHelper/IWriterRow) | Defines methods used to write a CSV row.                     |

##### 枚举（Enums）

|                                                              |                  |
| :----------------------------------------------------------- | :--------------- |
| [Caches](https://joshclose.github.io/CsvHelper/api/CsvHelper/Caches) | Types of caches. |