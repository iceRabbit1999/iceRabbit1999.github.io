# Definition

Restricts the instantiation of a class to one "single" instance

# Implementation

1. 

   ```java
   public class Singleton{
       private Singleton(){}
       
       private static final Singleton instance = new Singleton();
       
       public static Singleton getInstance(){
           return instance;
       }
   }
   ```

   

2. ```java
   public class Singleton{
       private Singleton(){}
       
       private static final Singleton instance;
       
       static{
           instance = new Singleton();
       }
       
       public static Singleton getInstance(){
           return instance;
       }
   }
   ```

   避免了多线程同步问题；没有达到懒加载的效果

3. ```java
   public class Singleton{
       private Singleton(){}
       
       private static final Singleton instance;
   
       public static Singleton getInstance(){
           if(instance == null){
               instance = new Singleton();
           }
           return instance;
       }
   }
   ```

   懒加载，但线程不安全

4. 加synchronized

   1. ```java
      public class Singleton{
          private Singleton(){}
          
          private static final Singleton instance;
      
          public static synchronized Singleton getInstance(){
              if(instance == null){
                  instance = new Singleton();
              }
              return instance;
          }
      }
      ```

5. ```java
   public class Singleton{
       private Singleton(){}
       
       private static final Singleton instance;
   
       public static Singleton getInstance(){
           if(instance == null){
              synchronized (Singleton.class){
                  if(instance == null){
                      instance = new Singleton();
                  }
              }
           }
           return instance;
       }
   }
   ```

   1. 线程安全、懒加载、效率高

   2. 可能出现空指针(在jvm指令重排之后)：解决 –> 加volatile

   3. ```java
      public class Singleton{
          private Singleton(){}
          
          private static volatile final Singleton instance;
      
          public static Singleton getInstance(){
              if(instance == null){
                 synchronized (Singleton.class){
                     if(instance == null){
                         instance = new Singleton();
                     }
                 }
              }
              return instance;
          }
      }
      ```

6. 





