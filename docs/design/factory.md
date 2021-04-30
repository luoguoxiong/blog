# 工厂模式



> `继承同一父类、实现同一接口的子类`对象，由给定的多个类型参数创建具体的对象。
>
> 该模式用于封装和管理对象的创建，是一种创建型模式



## 一、工厂模式的实现

```typescript
interface Hello{
    sayHello:() => void;
}

enum Type {
    A,
    B
}

enum UserType{
    Teacher,
    Student
}

class TA implements Hello{
  sayHello(){
    console.log('TA sayhello');
  }
}

class TB implements Hello{
  sayHello(){
    console.log('TB sayhello');
  }
}

class SA implements Hello {
  sayHello() {
    console.log('Student A say hello');
  }
}

class SB implements Hello {
  sayHello() {
    console.log('Student B say hello');
  }
}
class AFactory {
    static list = new Map<UserType, Hello>([
      [UserType.Teacher, new TA()],
      [UserType.Student, new SA()],
    ])

    static getHello(occupation:UserType) {
      return AFactory.list.get(occupation);
    }
}

class BFactory {
    static list = new Map<UserType, Hello>([
      [UserType.Teacher, new TB()],
      [UserType.Student, new SB()],
    ])

    static getHello(occupation:UserType) {
      return BFactory.list.get(occupation);
    }
}

class HelloFactory {
    static list = new Map<Type, AFactory | BFactory>([
      [Type.A, AFactory],
      [Type.B, BFactory],
    ])

    static getType(type:Type) {
      return HelloFactory.list.get(type);
    }
}

HelloFactory.getType(Type.A).getHello(UserType.Teacher).sayHello();
HelloFactory.getType(Type.A).getHello(UserType.Student).sayHello();
HelloFactory.getType(Type.B).getHello(UserType.Teacher).sayHello();
HelloFactory.getType(Type.B).getHello(UserType.Student).sayHello();
```

