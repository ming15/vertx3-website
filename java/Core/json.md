# JSON

不像其他语言,JAVA并没有一等类来支持`JSON`,因此Vert.x提供了下面俩个类让`JSON`的使用更加简便

## JSON objects

`JsonObject`表示一个`JSON`对象。

`JsonObject`基本上只是一个`string key`和`value`的一个映射,`value`可以是`JSON`支持的数据类型的一种(`string, number, boolean`)

同时`JSON`对象还支持`null`值

###Creating JSON objects

如果使用默认的`JsonObject`构造器创建出来的就是一个空`JSON`对象

You can create a JSON object from a string JSON representation as follows:
你也可以使用一个`String`表示的`JSON`来创建一个`JsonObject`对象。
```
String jsonString = "{\"foo\":\"bar\"}";
JsonObject object = new JsonObject(jsonString);
```

###Putting entries into a JSON object

我们可以直接使用`put`方法向`JsonObject`中添加元素
```
JsonObject object = new JsonObject();
object.put("foo", "bar").put("num", 123).put("mybool", true);
```

###Getting values from a JSON object

我们可以直接使用`get...`方法从`JsonObject`中获取某个值。
```
String val = jsonObject.getString("some-key");
int intVal = jsonObject.getInteger("some-other-key");
```

###Encoding the JSON object to a String

你可以直接使用`encode`方法将某个对象编码成字符串形式

## JSON arrays

`JsonArray`表示的是`JSON`数组

`JSON`数组就是`JSON value`的一个序列

`JSON`数组还可以包含`null`值


### Creating JSON arrays

如果使用默认的`JsonArray`构造器创建出来的就是一个空`JSON`数组对象

你也可以使用一个`String`表示的`JSON`来创建一个`JsonArray`对象。
```
String jsonString = "[\"foo\",\"bar\"]";
JsonArray array = new JsonArray(jsonString);
```

你可以直接使用`add`方法向一个`JsonArray`中添加元素
```
JsonArray array = new JsonArray();
array.add("foo").add(123).add(false);
```

###Getting values from a JSON array

同样的你可以使用`get...`方法直接从`JsonArray`获取元素
```
String val = array.getString(0);
Integer intVal = array.getInteger(1);
Boolean boolVal = array.getBoolean(2);
```

### Encoding the JSON array to a String

你可以直接使用`encode`方法将`JsonArray`编码成`String`