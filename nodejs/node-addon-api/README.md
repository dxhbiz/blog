### Nodejs与C/C++互传结构体

### 解决方案
- Nodejs和C/C++使用Buffer作为输入与输出
- Nodejs接入ref、ref-struct实现Buffer转换成js对象


### 示例
- 结构定义

C/C++结构体定义
```cpp
typedef struct {
    char name[32];
    uint8_t age;
    uint8_t grade;
} student_t;
```

Nodejs定义
```nodejs
const ref = require('ref-napi');
const Struct = require('ref-sturct-di')(ref);
const refArray = require('ref-array-di')(ref);

const student = Struct({
    name: refArray(ref.types.char, 32),
    age: ref.types.uint8,
    grade: ref.types.uint8,
});
```
- C/C++封装

```cpp
// 本例使用EventEmitter实现
/**
 * 初始化emitFn
 * Napi::Function emit =
 *    info.This().As<Napi::Object>().Get("emit").
 *    As<Napi::Function>();
 * this->emitFn = Napi::Persistent(emit);
**/

student_t student;
std::vector<char> bytes(sizeof(student_t));
memcpy(bytes.data(), &student, sizeof(student_t));

this->emitFn.Call(
    info.This(),
    {Napi::String::New(env, "data"),  Napi::Buffer<char>::Copy(env, bytes.data(), bytes.size())});
```

- Nodejs解析
```nodejs
event.on('data', (buf) => {
    const s = new student(buf);
    const name = Buffer.from(s.name.buffer);
    const age = s.age;
    const grade = s.grade;
});
```


### 参考文档
- [Node addon 示例](https://github.com/nodejs/node-addon-examples)
- [ref-napi](https://github.com/node-ffi-napi/ref-napi)
- [ref-struct-di](https://github.com/node-ffi-napi/ref-struct-di)
- [ref-array-di](https://github.com/node-ffi-napi/ref-array-di)