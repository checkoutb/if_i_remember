Design  Pattern
2021年12月6日
10:06

https://www.runoob.com/design-pattern/design-pattern-tutorial.html

https://design-patterns.readthedocs.io/zh_CN/latest/index.html



[[toc]]


=================================

# 设计模式原则

对接口编程而不是对实现编程
优先使用对象组合而不是继承


|     |     |
| --- | --- |
| 开闭原则 | 对扩展开放，对修改关闭 |
| 里氏代换原则 | 任何基类可以出现的地方，子类一定可以出现 |
| 依赖倒转原则 | 针对接口编程，依赖于抽象而不依赖于具体 |
| 接口隔离原则 | 使用多个隔离的接口，比使用单个接口要好。降低类之间的耦合度 |
| 迪米特法则/最少知道原则 | 一个实体应该尽量少地与其他实体发生相互作用，使得系统功能模块相对独立。 |
| 合成复用原则 | 尽量使用合成/聚合的方式，而不是使用继承 |
|单一职责|。|


# 设计模式分类

|     |     |     |
| --- | --- | --- |
| 创建型模式 | 提供在创建对象时隐藏创建逻辑的方式，不使用new直接创建。使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。 | 工厂模式，抽象工厂模式，单例模式，建造者模式，原型模式 |
| 结构型模式 | 关注类和对象的组合，继承的概念被用来组合接口和定义组合对象获得新功能的方式 | 适配器模式，桥接模式，过滤器模式，组合模式，装饰器模式，外观模式，享元模式，代理模式 |
| 行为型模式 | 关注  对象之间的通信 | 责任链，命令，解释器，迭代，中介者，备忘录，观察者，状态，空对象，策略，模板，访问者 |
| J2EE模式 | 关注表示层，由Sun Java Center  鉴定的 | MVC，业务代表，组合实体，数据访问对象，前端控制器，拦截过滤器，服务定位器，传输对象 |

![计算机生成了可选文字: 保存迭代状态 创建组合 给增加职 Decorator 改变外表 改变内容 共車组合 Flyweight 車策略 避免滞后 枚举子女 使用组合命令 定义遍历 增加操作 定义语法 定义链 共享终结符 共車．态 定义算法步骤 加操作 TemplateMethod ChainofR腸Il 经常使用 Factoryueth0d 动态地配置工厂 单个实例 单个实例 引let 图 用工厂方法实现 设计模式之间的关系](../_resources/7f9f9733395a41b284356384cb947035.jpg)

=================================

# 工厂模式

定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使得创建过程延迟到子类进行。

一个产品一个工厂。

public class ShapeFactory {

   //使用 getShape 方法获取形状类型的对象
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }
}

。。new的3个类  都是  Shape的  子类。
。。但是这里哪有  让其子类自己决定实例化哪一个工厂类。？

。。只是为了  将对象  的种类  压缩到  某几种。

=================================

# 抽象工厂模式

围绕一个超级工厂创建其他工厂。
工厂的工厂。

接口负责创建一个  相关对象的工厂，不需要显示指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

public class AbstractFactoryPatternDemo {
   public static void main(String[] args) {
      //获取形状工厂
      AbstractFactory shapeFactory = FactoryProducer.getFactory("SHAPE");
      //获取形状为 Circle 的对象
      Shape shape1 = shapeFactory.getShape("CIRCLE");
      //调用 Circle 的 draw 方法
      shape1.draw();
      //获取形状为 Rectangle 的对象
      Shape shape2 = shapeFactory.getShape("RECTANGLE");
      //调用 Rectangle 的 draw 方法
      shape2.draw();
      //获取形状为 Square 的对象
      Shape shape3 = shapeFactory.getShape("SQUARE");
      //调用 Square 的 draw 方法
      shape3.draw();

      //获取颜色工厂
      AbstractFactory colorFactory = FactoryProducer.getFactory("COLOR");
      //获取颜色为 Red 的对象
      Color color1 = colorFactory.getColor("RED");
      //调用 Red 的 fill 方法
      color1.fill();
      //获取颜色为 Green 的对象
      Color color2 = colorFactory.getColor("Green");
      //调用 Green 的 fill 方法
      color2.fill();
      //获取颜色为 Blue 的对象
      Color color3 = colorFactory.getColor("BLUE");
      //调用 Blue 的 fill 方法
      color3.fill();
   }
}

。。感觉没有什么用啊。。

=================================

# 单例模式

单例类只有一个实例
单例类必须自己创建  自己的唯一实例
单例类必须给所有其他对象提供  这一实例

public class SingleObject {
   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();
   //让构造函数为 private，这样该类就不会被实例化
    private SingleObject(){}
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }
   public void showMessage(){
      System.out.println("Hello World!");
   }
}

懒汉式，线程不安全，延迟初始化
public class Singleton {
    private static Singleton instance;
    private Singleton (){}

    public static Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
    }
}

懒汉式，线程安全，延迟初始化
public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static synchronized Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
    }
}

饿汉式，安全，立即初始化。。。。一般使用这种
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton (){}
    public static Singleton getInstance() {
    return instance;
}
}

双重锁校验  DCL double checked locking，安全，延迟
public class Singleton {
    private volatile static Singleton singleton;
    private Singleton (){}
    public static Singleton getSingleton() {
    if (singleton == null) {
        synchronized (Singleton.class) {
        if (singleton == null) {
            singleton = new Singleton();
        }
        }
    }
    return singleton;
    }
}

登记式/静态内部类，安全，延迟。。。。。。  需要延迟加载，就使用这种。
public class Singleton {
    private static class SingletonHolder {
    private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton (){}
    public static final Singleton getInstance() {
    return SingletonHolder.INSTANCE;
    }
}

枚举，安全，立即
public enum Singleton {
    INSTANCE;
    public void whateverMethod() {
    }
}

一般情况下，不建议  使用第一，第二的  懒汉式，  建议使用  第三种  饿汉式。  如果需要  延迟加载，使用  第五种。如果涉及  反序列化创建对象，尝试第六种。

=================================

# 建造者模式

使用多个简单的对象(？感觉是步骤啊)一步一步构造出一个复杂对象。

将复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

主要解决：有时  面临着  一个  复杂对象的  创建工作，它的各个部分是由一定的算法构成，由于需求的变化，这个复杂对象  的各个部分经常面临剧烈变化，但是  将它们组合在一起的  算法  相对稳定。

。。部分
public class Pepsi extends ColdDrink {
   @Override
   public float price() {
      return 35.0f;
   }
   @Override
   public String name() {
      return "Pepsi";
   }
}
。。组合
public class Meal {
   private List<Item> items = new ArrayList<Item>();

   public void addItem(Item item){
      items.add(item);
   }
   public float getCost(){
      float cost = 0.0f;
      for (Item item : items) {
         cost += item.price();
      }
      return cost;
   }
   public void showItems(){
      for (Item item : items) {
         System.out.print("Item : "+item.name());
         System.out.print(", Packing : "+item.packing().pack());
         System.out.println(", Price : "+item.price());
      }
   }
}
。。builder
public class MealBuilder {
   public Meal prepareVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new VegBurger());
      meal.addItem(new Coke());
      return meal;
   }
   public Meal prepareNonVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new ChickenBurger());
      meal.addItem(new Pepsi());
      return meal;
   }
}

。。我认为的  budiler  和这个有点区别，我以为的是内部builder类  +  链式调用，最后build()  。

。。这里是  固定下来。  不同的部分  根据一定的算法  组合。  组合确定的，后续也可以添加的。。。这和  工厂模式  差不多啊。  隐藏了  创建的过程。

=================================

# 原型模式

创建重复对象，保证性能。

实现一个原型接口，用于创建当前对象的  克隆。

但直接创建对象的代价比较大时，采用这种模式。比如，创建一个对象  需要从数据库里  获得数据，那么可以缓存  这个对象，下一次返回它的clone。

一个对象多个修改者，并且修改者会修改值。

通过拷贝  现有对象  生成  新对象，浅拷贝Cloneable接口，  重写，深拷贝是Serializable  读取二进制流。

。实现Cloneable接口
   public Object clone() {
      Object clone = null;
      try {
         clone = super.clone();
      } catch (CloneNotSupportedException e) {
         e.printStackTrace();
      }
      return clone;
   }

public class ShapeCache {

   private static Hashtable<String, Shape> shapeMap
      = new Hashtable<String, Shape>();

   public static Shape getShape(String shapeId) {
      Shape cachedShape = shapeMap.get(shapeId);
      return (Shape) cachedShape.clone();
   }

   // 对每种形状都运行数据库查询，并创建该形状
   // shapeMap.put(shapeKey, shape);
   // 例如，我们要添加三种形状
   public static void loadCache() {
      Circle circle = new Circle();
      circle.setId("1");
      shapeMap.put(circle.getId(),circle);

      Square square = new Square();
      square.setId("2");
      shapeMap.put(square.getId(),square);

      Rectangle rectangle = new Rectangle();
      rectangle.setId("3");
      shapeMap.put(rectangle.getId(),rectangle);
   }
}

public class PrototypePatternDemo {
   public static void main(String[] args) {
      ShapeCache.loadCache();

      Shape clonedShape = (Shape) ShapeCache.getShape("1");
      System.out.println("Shape : " + clonedShape.getType());

      Shape clonedShape2 = (Shape) ShapeCache.getShape("2");
      System.out.println("Shape : " + clonedShape2.getType());

      Shape clonedShape3 = (Shape) ShapeCache.getShape("3");
      System.out.println("Shape : " + clonedShape3.getType());
   }
}

。。就是一个  cache，并且  cache  返回的是  一个  克隆后的  新对象。
。。cache  一般是  不可修改的值。  如果  值会被修改，那就  clone + prototype

=================================

# 适配器模式

作为  2个  不兼容的接口  之间的  桥梁。

结构型，它结合了  两个  独立接口的  功能。

涉及到一个  单一的类，该类负责加入  独立的  或  不兼容的  接口功能。

将一个  类的  接口  转换成  另一个  接口。使得原本因为  接口不兼容  而不能一起工作的  类  可以一起工作。

适配器对接口进行了转换，所以增加了复杂度，如果适配器过多，应该直接重构。

由于java单继承，所以最多只能适配一个  适配者类，而且目标类  必须是  抽象类。。。？

适配器类  不是在设计时添加，  而是为了  解决  正在使用的  2个项目的问题。

![计算机生成了可选文字: <<lnterface>> MediaPlayer +pla¥0v00 AdapterPatternDemo +main()void <<lnterface>> AdvancedMediaPlayer +pla¥V0《void +playMp4():void implements MediaAdapter -advancedMediaPlayer: AdvancedMediaPlaver +MediaAdapter():void +pla¥0v00 VlcPlayer +playVLC():void +pla¥Mp4()Old 亻吏总 Mp4Player +playVLC():void +pla¥Mp4()Oid implements AudioPlayer 亻吏总-mediaAdapter:MediaAdapter +pla¥0：v00 WWW.runOOb.COm](../_resources/60093e066be644478ca404d6387b8936.png)

。。最终要使用  AudioPlayer，来播放  mp3,vlc,mp4.

。。这个好像还多了一层  AudioPlayer  。。MediaAdapter  已经能够在  使用MediaPlayer的地方使用了，并且实际效果是  vlc，mp4。

。。当你要把一个接口A  转为另一个接口B (主系统使用B，外部系统使用A)(数据从A到B/A生成数据，B使用数据)时，  需要  一个实现接口B  的类C，C里有一个  A属性。  把接口A的对象放到C里，C实现B的方法，并且实现的时候  调用A对象。

public class MediaAdapter implements MediaPlayer {

   AdvancedMediaPlayer advancedMusicPlayer;

   public MediaAdapter(String audioType){
      if(audioType.equalsIgnoreCase("vlc") ){
         advancedMusicPlayer = new VlcPlayer();
      } else if (audioType.equalsIgnoreCase("mp4")){
         advancedMusicPlayer = new Mp4Player();
      }
   }

   @Override
   public void play(String audioType, String fileName) {
      if(audioType.equalsIgnoreCase("vlc")){
         advancedMusicPlayer.playVlc(fileName);
      }else if(audioType.equalsIgnoreCase("mp4")){
         advancedMusicPlayer.playMp4(fileName);
      }
   }
}

public class AudioPlayer implements MediaPlayer {

   MediaAdapter mediaAdapter;

   @Override
   public void play(String audioType, String fileName) {

      //播放 mp3 音乐文件的内置支持
      if(audioType.equalsIgnoreCase("mp3")){
         System.out.println("Playing mp3 file. Name: "+ fileName);
      }
      //mediaAdapter 提供了播放其他文件格式的支持
      else if(audioType.equalsIgnoreCase("vlc")
         || audioType.equalsIgnoreCase("mp4")){
         mediaAdapter = new MediaAdapter(audioType);
         mediaAdapter.play(audioType, fileName);
      }
      else{
         System.out.println("Invalid media. "+
            audioType + " format not supported");
      }
   }
}

=================================

# 桥接模式

用于把  抽象化  和实现化  解耦，使得两者可以独立变化。

。。？更像是，2个维度变化，一个维度单独抽取成类A，  类A  和  另一个维度的  属性  聚合成  类B，  这样的话，  就能  独立变化了。

public class Circle extends Shape {
   private int x, y, radius;

   public Circle(int x, int y, int radius, DrawAPI drawAPI) {
      super(drawAPI);
      this.x = x;
      this.y = y;
      this.radius = radius;
   }

   public void draw() {
      drawAPI.drawCircle(radius,x,y);
   }
}

public class BridgePatternDemo {
   public static void main(String[] args) {
      Shape redCircle = new Circle(100,100, 10, new RedCircle());
      Shape greenCircle = new Circle(100,100, 10, new GreenCircle());

      redCircle.draw();
      greenCircle.draw();
   }
}
。。圆的参数，颜色

=================================

# 过滤器模式

允许开发人员使用不同的标准来过滤一组对象，通过逻辑运算  以解耦的方式把它们连接起来。
结合多个标准来获得  单一标准。

![计算机生成了可选文字: <<lnterface>> +meetCnter30七sPSO邝。 implements implements AndCritena 一ch「iaC「「ia 一0t0CiaCfitera +meetCriteria()凵s0SO邝。 Person -genderStri壟 -martalS%tLEStrlng +Person() +getAgeCString +getGender()String +g引Ma「1刳Stat凵(t)3ring OrCriteria +c「「iaC「门3 -0t0C「it急Cla +meetCfiteria()凵sPSO邝。 CriteriaFemaIe +me引C「冂30七sPSO邝。 使用 CritenaMaIe +meetCrterla()Llst<Person> CriteriaPatterDemo man():void](../_resources/b6ae24d873154010a76e267ce6529166.png)

public class OrCriteria implements Criteria {

   private Criteria criteria;
   private Criteria otherCriteria;

   public OrCriteria(Criteria criteria, Criteria otherCriteria) {
      this.criteria = criteria;
      this.otherCriteria = otherCriteria;
   }

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> firstCriteriaItems = criteria.meetCriteria(persons);
      List<Person> otherCriteriaItems = otherCriteria.meetCriteria(persons);

      for (Person person : otherCriteriaItems) {
         if(!firstCriteriaItems.contains(person)){
           firstCriteriaItems.add(person);
         }
      }
      return firstCriteriaItems;
   }
}

   public static void main(String[] args) {
      List<Person> persons = new ArrayList<Person>();

      persons.add(new Person("Robert","Male", "Single"));
      persons.add(new Person("John","Male", "Married"));
      persons.add(new Person("Laura","Female", "Married"));
      persons.add(new Person("Diana","Female", "Single"));
      persons.add(new Person("Mike","Male", "Single"));
      persons.add(new Person("Bobby","Male", "Single"));

      Criteria male = new CriteriaMale();
      Criteria female = new CriteriaFemale();
      Criteria single = new CriteriaSingle();
      Criteria singleMale = new AndCriteria(single, male);
      Criteria singleOrFemale = new OrCriteria(single, female);

      System.out.println("Males: ");
      printPersons(male.meetCriteria(persons));

      System.out.println("\nFemales: ");
      printPersons(female.meetCriteria(persons));

      System.out.println("\nSingle Males: ");
      printPersons(singleMale.meetCriteria(persons));

      System.out.println("\nSingle Or Females: ");
      printPersons(singleOrFemale.meetCriteria(persons));
   }

。。标准的  组装。  工具类。过滤。

=================================

# 组合模式

用于把一组相似的对象当做一个单一的对象。
依据树形结构来组合对象，用来表示部分以及整体层次。
创建了对象组的树形结构。

将对象组合成树形结构以表示  部分-整体  的层次结构。
使得用户  对单个对象  和组合对象  的使用  具有一致性。

使用树形结构，模糊了简单元素和复杂元素的概念，使得客户程序可以像处理简单元素一样来处理复杂元素，从而使得客户程序与  复杂元素的  内部结构  解耦。

public class Employee {
   private String name;
   private String dept;
   private int salary;
   private List<Employee> subordinates;

}

public class CompositePatternDemo {
   public static void main(String[] args) {
      Employee CEO = new Employee("John","CEO", 30000);
      Employee headSales = new Employee("Robert","Head Sales", 20000);
      Employee headMarketing = new Employee("Michel","Head Marketing", 20000);

      Employee clerk1 = new Employee("Laura","Marketing", 10000);
      Employee clerk2 = new Employee("Bob","Marketing", 10000);

      Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
      Employee salesExecutive2 = new Employee("Rob","Sales", 10000);

      CEO.add(headSales);
      CEO.add(headMarketing);

      headSales.add(salesExecutive1);
      headSales.add(salesExecutive2);

      headMarketing.add(clerk1);
      headMarketing.add(clerk2);

      //打印该组织的所有员工
      System.out.println(CEO);
      for (Employee headEmployee : CEO.getSubordinates()) {
         System.out.println(headEmployee);
         for (Employee employee : headEmployee.getSubordinates()) {
            System.out.println(employee);
         }
      }
   }
}

。。这个不是还是  要知道  复杂结构吗。。。我以为是  Employee  提供一个  printSelfAndSub方法，只要你调用这个printSelfAndSub，就会  自动print  自己  和  sub。

=================================

# 装饰器模式

允许向一个现有的对象  添加新的功能，同时又不改变其结构。

创建一个  装饰类，用来包装原有的类，并且在保持类方法签名完整性的前提下，提供额外的功能。

动态地给一个对象添加一些额外的职责。
就增加功能来说，装饰器模式  比  生成子类  更灵活。

public abstract class ShapeDecorator implements Shape {
   protected Shape decoratedShape;

   public ShapeDecorator(Shape decoratedShape){
      this.decoratedShape = decoratedShape;
   }

   public void draw(){
      decoratedShape.draw();
   }
}

public class RedShapeDecorator extends ShapeDecorator {
   public RedShapeDecorator(Shape decoratedShape) {
      super(decoratedShape);
   }

   @Override
   public void draw() {
      decoratedShape.draw();
      setRedBorder(decoratedShape);
   }

   private void setRedBorder(Shape decoratedShape){
      System.out.println("Border Color: Red");
   }
}

public class DecoratorPatternDemo {
   public static void main(String[] args) {

      Shape circle = new Circle();
      ShapeDecorator redCircle = new RedShapeDecorator(new Circle());
      ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle());
      //Shape redCircle = new RedShapeDecorator(new Circle());
      //Shape redRectangle = new RedShapeDecorator(new Rectangle());
      System.out.println("Circle with normal border");
      circle.draw();

      System.out.println("\nCircle of red border");
      redCircle.draw();

      System.out.println("\nRectangle of red border");
      redRectangle.draw();
   }
}
。。shape  只有  矩形，圆形，  现在增加了  draw  这个功能。  动态扩展。

=================================

# 外观模式

隐藏系统的复杂性，向客户端提供一个  客户端可以访问系统  的接口。

向现有系统添加一个  接口，来隐藏系统的复杂性。
涉及一个  单一的类，该类提供了  客户端请求的  简化方法  和对现有系统类方法的委托调用。

为子系统中的一组接口提供一个  一致的界面，外观模式定义了  一个高层接口，这个接口使得这一子系统更加容易使用。

public class ShapeMaker {
   private Shape circle;
   private Shape rectangle;
   private Shape square;

   public ShapeMaker() {
      circle = new Circle();
      rectangle = new Rectangle();
      square = new Square();
   }

   public void drawCircle(){
      circle.draw();
   }
   public void drawRectangle(){
      rectangle.draw();
   }
   public void drawSquare(){
      square.draw();
   }
}

public class FacadePatternDemo {
   public static void main(String[] args) {
      ShapeMaker shapeMaker = new ShapeMaker();

      shapeMaker.drawCircle();
      shapeMaker.drawRectangle();
      shapeMaker.drawSquare();
   }
}

=================================

# 享元模式

减少创建对象的数量，减少内存占用和提高性能。

尝试重用现有的同类对象，如果没有找到匹配的对象，则创建新对象。

实例：String，数据库连接池
。。cache。。。

场景：有大量相似对象；需要缓冲池

注意划分  外部状态  和内部状态，否则可能引起线程安全问题。
必须有一个工程对象加以控制。

public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();

   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);

      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}

public class FlyweightPatternDemo {
   private static final String colors[] =
      { "Red", "Green", "Blue", "White", "Black" };
   public static void main(String[] args) {

      for(int i=0; i < 20; ++i) {
         Circle circle =
            (Circle)ShapeFactory.getCircle(getRandomColor());
         circle.setX(getRandomX());
         circle.setY(getRandomY());
         circle.setRadius(100);
         circle.draw();
      }
   }
   private static String getRandomColor() {
      return colors[(int)(Math.random()*colors.length)];
   }
   private static int getRandomX() {
      return (int)(Math.random()*100 );
   }
   private static int getRandomY() {
      return (int)(Math.random()*100);
   }
}

=================================

# 代理模式

一个类代表另一个类的功能。

创建具有现有对象的对象，以便向外界提供功能接口。

和适配器的区别：适配器模式  主要改变所考虑对象的接口，而代理不能改变所代理类的接口
装饰器区别：装饰器为了  增强功能，代理为了加以控制。

public class RealImage implements Image {

   private String fileName;

   public RealImage(String fileName){
      this.fileName = fileName;
      loadFromDisk(fileName);
   }

   @Override
   public void display() {
      System.out.println("Displaying " + fileName);
   }

   private void loadFromDisk(String fileName){
      System.out.println("Loading " + fileName);
   }
}

public class ProxyImage implements Image{

   private RealImage realImage;
   private String fileName;

   public ProxyImage(String fileName){
      this.fileName = fileName;
   }

   @Override
   public void display() {
      if(realImage == null){
         realImage = new RealImage(fileName);
      }
      realImage.display();
   }
}

   public static void main(String[] args) {
      Image image = new ProxyImage("test_10mb.jpg");

      // 图像将从磁盘加载
      image.display();
      System.out.println("");
      // 图像不需要从磁盘加载
      image.display();
   }

。。。RealImage  构造器访问  磁盘。。。
。。单例，cache/享元，代理。。
。。而且，RealImage  也只需要访问一次  磁盘。。。

=================================

# 责任链模式

为请求创建一个接收者对象的链。

这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。

通常每个接收者都包含对另一个接收者的引用，如果一个对象不能处理这个请求，那么就把这个请求传递给下一个接收者。

避免将发送者  和  接收者耦合在一起，让多个对象都有可能接受请求，将这些对象连接成一条链，并且沿着这条链传递请求。

public abstract class AbstractLogger {
   public static int INFO = 1;
   public static int DEBUG = 2;
   public static int ERROR = 3;

   protected int level;

   //责任链中的下一个元素
   protected AbstractLogger nextLogger;

   public void setNextLogger(AbstractLogger nextLogger){
      this.nextLogger = nextLogger;
   }

   public void logMessage(int level, String message){
      if(this.level <= level){
         write(message);
      }
      if(nextLogger !=null){
         nextLogger.logMessage(level, message);
      }
   }

   abstract protected void write(String message);

}

public class ConsoleLogger extends AbstractLogger {

   public ConsoleLogger(int level){
      this.level = level;
   }

   @Override
   protected void write(String message) {
      System.out.println("Standard Console::Logger: " + message);
   }
}

public class ChainPatternDemo {
   private static AbstractLogger getChainOfLoggers(){

      AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
      AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
      AbstractLogger consoleLogger = new ConsoleLogger(AbstractLogger.INFO);

      errorLogger.setNextLogger(fileLogger);
      fileLogger.setNextLogger(consoleLogger);

      return errorLogger;
   }

   public static void main(String[] args) {
      AbstractLogger loggerChain = getChainOfLoggers();

      loggerChain.logMessage(AbstractLogger.INFO, "This is an information.");

      loggerChain.logMessage(AbstractLogger.DEBUG,
         "This is a debug level information.");

      loggerChain.logMessage(AbstractLogger.ERROR,
         "This is an error information.");
   }
}

=================================

# 命令模式

数据驱动的  设计模式。

请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并将该命令传递给相应的对象，该对象执行命令。

将请求包装成对象，从而使你可以用不同的请求对客户进行参数化。

public class BuyStock implements Order {
   private Stock abcStock;

   public BuyStock(Stock abcStock){
      this.abcStock = abcStock;
   }

   public void execute() {
      abcStock.buy();
   }
}

public class SellStock implements Order {
   private Stock abcStock;

   public SellStock(Stock abcStock){
      this.abcStock = abcStock;
   }

   public void execute() {
      abcStock.sell();
   }
}

public class Broker {
   private List<Order> orderList = new ArrayList<Order>();

   public void takeOrder(Order order){
      orderList.add(order);
   }

   public void placeOrders(){
      for (Order order : orderList) {
         order.execute();
      }
      orderList.clear();
   }
}

   public static void main(String[] args) {
      Stock abcStock = new Stock();

      BuyStock buyStockOrder = new BuyStock(abcStock);
      SellStock sellStockOrder = new SellStock(abcStock);

      Broker broker = new Broker();
      broker.takeOrder(buyStockOrder);
      broker.takeOrder(sellStockOrder);

      broker.placeOrders();
   }

=================================

# 解释器模式  Interpreter Pattern

提供了  评估语言的语法或表达式的方式。
实现一个表达式接口，该接口解释一个特定的上下文。

用于SQL解析，符号处理引擎等

给定一个语言，定义它的文法表示，并定义一个解释器。

使用场景少，Java中碰到可以使用  expression4J

=================================

# 迭代器模式

用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。

=================================

# 中介者模式

用于降低多个对象和类之间的  通信复杂度。

提供一个中介类，该类通常处理不同类之间的通信，并支持松耦合。

用一个中介对象封装  一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其松耦合，而且可以独立地改变它们之间的交互。

主要解决：对象与对象之间存在大量的关联关系，这样会导致系统的结构变得很复杂，并且，如果一个对象发生改变，我们也需要跟踪与之相关联的对象，同时做出相应的处理。

多个类相互耦合，形成网状结构。使用中介者模式，改成  星形结构。

从1对多，变成了1对1

中介者会庞大，变得复杂，难以维护。

=================================

# 备忘录模式

保存一个对象的某个状态，以便在适当的时候恢复对象。

在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。

ctrl+z  撤销。数据库事务，游戏存档

消耗资源，如果类的成员变量过多，会占用比较大的资源。

public class Originator {
   private String state;
   public void setState(String state){
      this.state = state;
   }
   public String getState(){
      return state;
   }
   public Memento saveStateToMemento(){
      return new Memento(state);
   }
   public void getStateFromMemento(Memento Memento){
      state = Memento.getState();
   }
}

public class CareTaker {
   private List<Memento> mementoList = new ArrayList<Memento>();

   public void add(Memento state){
      mementoList.add(state);
   }

   public Memento get(int index){
      return mementoList.get(index);
   }
}

   public static void main(String[] args) {
      Originator originator = new Originator();
      CareTaker careTaker = new CareTaker();
      originator.setState("State #1");
      originator.setState("State #2");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #3");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #4");

      System.out.println("Current State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(0));
      System.out.println("First saved State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(1));
      System.out.println("Second saved State: " + originator.getState());
   }

=================================

# 观察者模式

当对象间存在一对多关系时，就使用观察者模式。比如，当一个对象被修改时，会自动通知依赖它的对象。

主要解决：一个对象状态的改变给其他对象通知的问题。

如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
如果观察者和  观察目标之间有  循环依赖的话，观察目标会导致  循环调用，可能导致系统崩溃。
观察者模式，只是让观察者知道  观察目标发生了变换，无法知道  怎么发生变化。

使用场景：
一个抽象模型有2个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以独立地改变和复用
一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度
一个对象必须通知其他对象，而并不知道这些对象是谁
需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象…  可以使用观察者模式创建一种  链式触发机制。

注意事项：
Java中已经有了对观察者模式的支持类。(java.util.Observable  ?)
避免循环引用
如果顺序执行，某一观察者错误将导致系统卡壳，一般采用异步方式。

public class Subject {

   private List<Observer> observers
      = new ArrayList<Observer>();
   private int state;

   public int getState() {
      return state;
   }

   public void setState(int state) {
      this.state = state;
      notifyAllObservers();
   }

   public void attach(Observer observer){
      observers.add(observer);
   }

   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }
}

public class OctalObserver extends Observer{

   public OctalObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }

   @Override
   public void update() {
     System.out.println( "Octal String: "
     + Integer.toOctalString( subject.getState() ) );
   }
}

public class ObserverPatternDemo {
   public static void main(String[] args) {
      Subject subject = new Subject();

      new HexaObserver(subject);
      new OctalObserver(subject);
      new BinaryObserver(subject);

      System.out.println("First state change: 15");
      subject.setState(15);
      System.out.println("Second state change: 10");
      subject.setState(10);
   }
}

=================================

# 状态模式

类的行为是基于它的状态改变的。

创建表示各种状态的对象  和一个行为随着状态对象改变而改变的  context  对象。

允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。

使用场景：
行为随着状态改变而改变的场景
条件，分支语句的代替者。

public interface State {
   public void doAction(Context context);
}

public class StartState implements State {

   public void doAction(Context context) {
      System.out.println("Player is in start state");
      context.setState(this);
   }

   public String toString(){
      return "Start State";
   }
}

public class StopState implements State {

   public void doAction(Context context) {
      System.out.println("Player is in stop state");
      context.setState(this);
   }

   public String toString(){
      return "Stop State";
   }
}

public class Context {
   private State state;

   public Context(){
      state = null;
   }

   public void setState(State state){
      this.state = state;
   }

   public State getState(){
      return state;
   }
}

public class StatePatternDemo {
   public static void main(String[] args) {
      Context context = new Context();

      StartState startState = new StartState();
      startState.doAction(context);

      System.out.println(context.getState().toString());

      StopState stopState = new StopState();
      stopState.doAction(context);

      System.out.println(context.getState().toString());
   }
}

。。状态和context  分离。。。这里只是  状态有用，  context  除了保存下状态，没有任何作用啊。

。。感觉是  应该  state  中  有对于  context  的操作，  context  保存了state，  当  state  改变的时候，context.execute()  也会改变(直接调用  state的方法，state的又操作context，比如stopState里面就会调用  context.stop(), startState里面调用  context.start())。。

。。。擦，是策略模式。。

=================================

# 空对象模式

用一个空对象  取代  null

public class CustomerFactory {

   public static final String[] names = {"Rob", "Joe", "Julie"};

   public static AbstractCustomer getCustomer(String name){
      for (int i = 0; i < names.length; i++) {
         if (names[i].equalsIgnoreCase(name)){
            return new RealCustomer(name);
         }
      }
      return new NullCustomer();
   }
}

   public static void main(String[] args) {

      AbstractCustomer customer1 = CustomerFactory.getCustomer("Rob");
      AbstractCustomer customer2 = CustomerFactory.getCustomer("Bob");
      AbstractCustomer customer3 = CustomerFactory.getCustomer("Julie");
      AbstractCustomer customer4 = CustomerFactory.getCustomer("Laura");

      System.out.println("Customers");
      System.out.println(customer1.getName());
      System.out.println(customer2.getName());
      System.out.println(customer3.getName());
      System.out.println(customer4.getName());
   }

=================================

# 策略模式

一个类的行为或其算法可以在运行时更改。
创建表示各种策略的对象  和  一个行为随着策略对象  改变而改变的  context对象。策略对象改变context对象的执行算法。

定义一系列的算法，把它们一个个封装起来，使得它们可以互相替换。

主要解决：在有多种算法相似的情况下，使用if else  所带来的复杂和难以维护。

public interface Strategy {
   public int doOperation(int num1, int num2);
}

public class OperationSubtract implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 - num2;
   }
}

public class Context {
   private Strategy strategy;

   public Context(Strategy strategy){
      this.strategy = strategy;
   }

   public int executeStrategy(int num1, int num2){
      return strategy.doOperation(num1, num2);
   }
}

public class StrategyPatternDemo {
   public static void main(String[] args) {
      Context context = new Context(new OperationAdd());
      System.out.println("10 + 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationSubtract());
      System.out.println("10 - 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationMultiply());
      System.out.println("10 * 5 = " + context.executeStrategy(10, 5));
   }
}

=================================

# 模板模式

一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需重写方法实现，但调用将以抽象类中定义的方式进行。

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方式使得子类可以不改变一个算法的结构  即可重定义该算法的某些特定步骤。

为防止恶意操作，一般模板方法都加上final关键词。

public abstract class Game {
   abstract void initialize();
   abstract void startPlay();
   abstract void endPlay();

   //模板
   public final void play(){

      //初始化游戏
      initialize();

      //开始游戏
      startPlay();

      //结束游戏
      endPlay();
   }
}

public class Football extends Game {

   @Override
   void endPlay() {
      System.out.println("Football Game Finished!");
   }

   @Override
   void initialize() {
      System.out.println("Football Game Initialized! Start playing.");
   }

   @Override
   void startPlay() {
      System.out.println("Football Game Started. Enjoy the game!");
   }
}

public class TemplatePatternDemo {
   public static void main(String[] args) {

      Game game = new Cricket();
      game.play();
      System.out.println();
      game = new Football();
      game.play();
   }
}

=================================

# 访问者模式

使用一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。

根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。

主要将  数据结构和数据操作分离。

何时使用：需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中。

在被访问的类中添加一个  对外提供接待访问者的接口。
在数据基础类中有一个方法接受访问者，将自身引用传入访问者。

public interface ComputerPart {
   public void accept(ComputerPartVisitor computerPartVisitor);
}

public class Keyboard  implements ComputerPart {

   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}

public class Computer implements ComputerPart {

   ComputerPart[] parts;

   public Computer(){

      parts = new ComputerPart[] {new Mouse(), new Keyboard(), new Monitor()};

   }

   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      for (int i = 0; i < parts.length; i++) {
         parts[i].accept(computerPartVisitor);
      }
      computerPartVisitor.visit(this);
   }
}

public class ComputerPartDisplayVisitor implements ComputerPartVisitor {

   @Override
   public void visit(Computer computer) {
      System.out.println("Displaying Computer.");
   }

   @Override
   public void visit(Mouse mouse) {
      System.out.println("Displaying Mouse.");
   }

   @Override
   public void visit(Keyboard keyboard) {
      System.out.println("Displaying Keyboard.");
   }

   @Override
   public void visit(Monitor monitor) {
      System.out.println("Displaying Monitor.");
   }
}

public class VisitorPatternDemo {
   public static void main(String[] args) {

      ComputerPart computer = new Computer();
      computer.accept(new ComputerPartDisplayVisitor());
   }
}

=================================

# MVC模式

|     |     |
| --- | --- |
| Model | 代表一个存储数据的对象或Java  POJO。也可以带有逻辑，在数据变化时更新控制器 |
| View | 代表  模型包含的数据的可视化 |
| Controller | 作用于模型和视图。控制数据  流向  模型对象，并在数据变化时更新视图。它使得视图与模型分离开。 |

=================================

# 业务代表模式

用于对  表示层和业务层解耦。基本上用来  减少通信  或对表示层中的业务层代码的  远程查询功能。

业务层中有以下实体：

|     |     |
| --- | --- |
| 客户端 | 表示层代码可以是JSP，servlet，或UI  Java代码 |
| 业务代表 | 一个为客户端实体提供的入口类，它提供了对业务服务方法的访问 |
| 查询服务 | 查询服务对象负责获取相关的业务实现，并提供业务对象对业务代表对象的访问 |
| 业务服务 | 业务服务接口。实现了该业务服务的实体类，提供了实际的业务实现逻辑 |

![Client. BusinessDelegate. BusinessSetvice. LookUpSetvice, JMSSetvice EJ8Setvice  BusinessDelegatePattemDemo BusinessDelegate Client  BusinessDelegatePattemDemo  BusinessLookup  + doProc  Business Service  + doProc  implements  EJsservice  + doProc  + man():void  BusinessDelegate  ookup : ausnessLookup  - service : ausnessServlce  - serviceType : Stnng  + doTas  im lements  JMS Service  Client  - service : Bus nessDelegate  + Client(8usnessDeegate)  + doTask():vod  wwwmnoo&ccm](../_resources/2f0a11bdc23b4ab59c009381b03bd6fd.png)

=================================

# 组合实体模式

用在EJB  持久化机制中。

一个组合实体是一个EBJ实体bean，代表了对象的图解。当更新一个组合实体时，内部依赖对象 beans 会自动更新，因为它们是由 EJB 实体 bean 管理的。

以下是组合实体 bean 的参与者。

|     |     |
| --- | --- |
| 组合实体（Composite Entity） | 它是主要的实体 bean。它可以是粗粒的，或者可以包含一个粗粒度对象，用于持续生命周期。 |
| 粗粒度对象（Coarse-Grained Object） | 该对象包含依赖对象。它有自己的生命周期，也能管理依赖对象的生命周期。 |
| 依赖对象（Dependent Object） | 依赖对象是一个持续生命周期依赖于粗粒度对象的对象。 |
| 策略（Strategies） | 策略表示如何实现组合实体。 |

![coarseGrainedObject  CompositeEntityPatternDemo , Client  Client  -enti : Com IteEnt  +printData() : void  +setData() : void  CompositeEntityPatternDemo  +main() void  uses  CompositeEntity  -c o: CoarseGrainedOb•ect  +getData() : String\]  +setData() : void  Coa rseG ra inedObject  -objectl : Dependentobjectl  -object2 : DependentObject2  +getData() : String[\]  +setData : void  uses  Dependentobjectl  -data : String  +getData() : String  +setData() : void  uses  DependentObject2  -data : String  +getData() : String  +setData() : void](../_resources/b55c59ab8f674fb0a3c8a0e14d3a9a73.png)

=================================

# 数据访问对象模式
Data  Access Object Pattern

用于把低级的数据访问API或操作  从高级的业务服务中分离出来。

|     |     |
| --- | --- |
| 数据访问对象接口（Data Access Object Interface） | 该接口定义了在一个模型对象上要执行的标准操作。 |
| 数据访问对象实体类（Data Access Object concrete class） | 该类实现了上述的接口。该类负责从数据源获取数据，数据源可以是数据库，也可以是 xml，或者是其他的存储机制。 |
| 模型对象/数值对象（Model Object/Value Object） | 该对象是简单的 POJO，包含了 get/set 方法来存储通过使用 DAO 类检索到的数据。 |

![StudentDao  DaoPattemDemo , StudentDao  Student  -name: String  -rollNo : int  +8etName(): String  void  +getRollNoO: int  +setRoIlNo() : String  StudentDac  uses  : List  +updateStudentlO : void  +deIeteStudent(): void  +addStudentO: void  Implements  StudentDaolmpl  -students: List  +StudentDaolmpl()  +8etAllStudents() : List  void  +deleteStudent(): void  •addStudent(): void  DaopatternDemo  *main') : void](../_resources/b6df5d8d0e704590962726616228761c.png)

=================================

# 前端控制器模式

用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。
该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。

以下是这种设计模式的实体。

|     |     |
| --- | --- |
| 前端控制器（Front Controller） | 处理应用程序所有类型请求的单个处理程序，应用程序可以是基于 web 的应用程序，也可以是基于桌面的应用程序。 |
| 调度器（Dispatcher） | 前端控制器可能使用一个调度器对象来调度请求到相应的具体处理程序。 |
| 视图（View） | 视图是为请求而创建的对象。 |

![HomeView-*U  FrontControllerPattemDemo , FrontController  Frontcontroller  -dispatcher: Dispatcher  +FrontController()  +isAuthenticuser() : void  +trackRequest() : void  : void  FrontControllerPatternDemo  +main() : void  Dispatcher  -studentVlew: StudentView  -homeView : HomeView  *Dispatcher')  uses  HomeVlew  +show() : void  uses  StudentVlew  +show() : void](../_resources/a841d21bc28e4410895ac40c78bae030.png)

=================================

# 拦截过滤器模式

用于对应用程序的请求或响应做一些预处理/后处理。定义过滤器，并在把请求传给实际目标应用程序之前应用在请求上。

过滤器可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。

以下是这种设计模式的实体。

|     |     |
| --- | --- |
| 过滤器（Filter） | 过滤器在请求处理程序执行请求之前或之后，执行某些任务。 |
| 过滤器链（Filter Chain） | 过滤器链带有多个过滤器，并在 Target 上按照定义的顺序执行这些过滤器。 |
| Target | Target 对象是请求处理程序。 |
| 过滤管理器（Filter Manager） | 过滤管理器管理过滤器和过滤器链。 |
| 客户端（Client） | Client 是向 Target 对象发送请求的对象。 |

![FiltetChain. FilterManager, Target, Client  InterceptingFilterDemo Client  FilterManager  -chain : FilterChan  + Filteflanager()  FilterChain  + titers : List  - target Target  + addFiIter():vöd  + execute()vod  + setTarget():v01d  Target  + execute()vod  Client  + tilterManager : Filtelvlanager  + setFterManager():v01d  + sendRequest():void  Filter  + execute():void  Implement  InterceptingFilterDemo  + main():void  NW"" runcc.com  DebugFilter  AuthenticationFilter  + exec  + exec](../_resources/4a5f6e5ae2b240f3bcfea1dbd98e03c6.png)

=================================

# 服务定位器模式

用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能。

以下是这种设计模式的实体。

|     |     |
| --- | --- |
| 服务（Service） | 实际处理请求的服务。对这种服务的引用可以在 JNDI 服务器中查找到。 |
| Context / 初始的 Context | JNDI Context 带有对要查找的服务的引用。 |
| 服务定位器（Service Locator） | 服务定位器是通过 JNDI 查找和缓存服务来获取服务的单点接触。 |
| 缓存（Cache） | 缓存存储服务的引用，以便复用它们。 |
| 客户端（Client） | Client 是通过 ServiceLocator 调用服务的对象。 |

![ServiceLocator. InitialContext. Cache. Service Service 1 Service2  ServiceLocatorPattemDemo  + main():void  ServiceLocator  - cache : Cache  InitialContext  •vww.nunoob.cotn  Cache  + services : List  + Cache()  <<lnterface>>  Service  + execute():vold  implements  Servicel  + getName():void  + execute():vod  Service2  + getName():void  + execute():vod](../_resources/2513985031f94da395db701ed8190ac7.png)

=================================

# 传输对象模式

用于从客户端向服务器一次性传递带有多个属性的数据。传输对象也被称为数值对象。传输对象是一个具有 getter/setter 方法的简单的 POJO 类，它是可序列化的，所以它可以通过网络传输。它没有任何的行为。服务器端的业务类通常从数据库读取数据，然后填充 POJO，并把它发送到客户端或按值传递它。对于客户端，传输对象是只读的。客户端可以创建自己的传输对象，并把它传递给服务器，以便一次性更新数据库中的数值。

以下是这种设计模式的实体。

|     |     |
| --- | --- |
| 业务对象（Business Object） | 为传输对象填充数据的业务服务。 |
| 传输对象（Transfer Object） | 简单的 POJO，只有设置/获取属性的方法。 |
| 客户端（Client） | 客户端可以发送请求或者发送传输对象到业务对象。 |

![Student80 StudentVO ,  Transfet0bjectPattemDemo , Student80 Student  StudentBO  + getAlIStudents():List  + updateStudent():v01d  + deleteStudent():v01d  + addStudent():v01d  StudentVO  - name : String  - rollNo : Int  + StudentVO()  + getName():String  + getRoIINo():int  + setRoIINo():String  TransferPatternDemo  •vww.nunoob.cotn](../_resources/638bba323aa44ce08264b2766a09736d.png)

=================================

=================================

已使用 Microsoft OneNote 2016 创建。