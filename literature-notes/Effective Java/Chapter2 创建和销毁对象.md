---
"type:": literature-note
"title:": Chapter2 创建和销毁对象
"id:": 20250715080556
"created:": 2025-07-15T08:05:56
source:
  - book
url: https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/tree/dev/Chapter-2
tags:
  - literature-note
  - java
"processed:": false
related-notes: 
"archived:": false
---
# 1 用静态工厂方法代替构造器

## 1.1 优势

### 1.1.1 与构造器不同，静态工厂方法有名称
### 1.1.2 不需要在调用时创建对象
### 1.1.3 可以返回子类型
### 1.1.4 返回的对象可以根据参数动态调整
### 1.1.5 返回的对象可以在代码编写时不存在，在运行时动态加载
比如 SPI 机制
## 1.2 劣势
### 1.2.1 私有构造器不能被继承
### 1.2.2 IDE 中不好排查在何时实例化的
不好找
## 1.3 静态工厂方法命名规范

```java
Date d = Date.from(instant);
// 用在多个参数聚合
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
StackWalker luke = StackWalker.getInstance(options);
Object newArray = Array.newInstance(classObject, arrayLen);

// Get type or new type or just type.
FileStore fs = Files.getFileStore(path);
BufferedReader br = Files.newBufferedReader(path);
List<Complaint> litany = Collections.list(legacyLitany);
```

## 1.4 Reference

[Effective-Java-3rd-edition-Chinese-English-bilingual/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md at dev · clxering/Effective-Java-3rd-edition-Chinese-English-bilingual · GitHub](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/dev/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)

# 2 构造器参数多了考虑使用 Builder 模式
## 2.1 用多个不同参数的构造器创建复杂对象
参数如果太多，客户端难以使用，不好选定使用哪一个构造方法。
```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
    private final int servingSize; // (mL) required
    private final int servings; // (per container) required
    private final int calories; // (per serving) optional
    private final int fat; // (g/serving) optional
    private final int sodium; // (mg/serving) optional
    private final int carbohydrate; // (g/serving) optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
## 2.2 用 Setter 创建复杂对象
分到多个调用中去，可能处于不一致的状态
## 2.3 用 Builder 创建复杂对象
易于使用
```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
调用方：
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```
可以使用 Lombok 的 Builder

# 3 私有构造器和枚举实现单例模式

## 3.1 静态变量方法 

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```

## 3.2 静态工厂方法 

```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
```

## 3.3 防止反射破坏单例

以上两种方法都可通过反射创建打破单例，可以在构造器中加一层判断：

```java
private Elvis() {
	if (INSTANCE != null) {
		throw new IllegalStateException(); 
	}
}
```

## 3.4 防止序列化和反序列化破坏单例

如果使用序列化和反序列化，依然可以破坏单例。

1. 可以在所有实例字段加 ` transient ` 关键字，避免序列化
2. 重写 `readResolve` 方法避免反序列化破坏

```java
// readResolve method to preserve singleton property
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
```

## 3.5 使用枚举实现单例

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

