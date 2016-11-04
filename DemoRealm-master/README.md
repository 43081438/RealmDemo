转自：https://github.com/RaphetS/DemoRealm
##Demo是以本地收藏为应用场景的，实现了对Realm的增删改查等操作，以及异步的增删改查操作，欢迎Star、Fork

##DownloadDemo
![image](https://github.com/43081438/RealmDemo/blob/master/DemoRealm-master/Screenshot/demoURL.png)


效果图

![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%A2%9E%E5%88%A0%E6%9F%A5.gif)
![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E6%94%B9.gif)

![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%88%A0.gif)
![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2.gif)


![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%85%B6%E4%BB%96%E6%9F%A5%E8%AF%A2.gif)
![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%BC%82%E6%AD%A5%E5%A2%9E%E5%88%A0.gif)

![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%BC%82%E6%AD%A5%E6%9B%B4%E6%96%B0.gif)
![image](https://github.com/RaphetS/DemoRealm/blob/master/Screenshot/%E5%BC%82%E6%AD%A5%E5%88%A0.gif)


# 目录 #

##1、realm简介
##2、环境配置
##3、在Application中初始化Realm
##4、创建实体
##5、增删改查
##6、异步操作
##7、数据迁移（版本升级）
## 一、Realm简介 ##
数据库Realm，是用来替代sqlite的一种解决方案，它有一套自己的数据库存储引擎，比sqlite更轻量级，拥有更快的速度，并且具有很多现代数据库的特性，比如支持JSON，流式api，数据变更通知，自动数据同步,简单身份验证，访问控制，事件处理，最重要的是跨平台，目前已有Java，Objective C，Swift，React-Native，Xamarin这五种实现。

本篇文章用的版本为Realm 2.0.2(官方文档)

文／RaphetS（简书作者）
原文链接：http://www.jianshu.com/p/28912c2f31db
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。

## 二、环境配置 ##
**（1） 在项目的build文件加上**

    buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "io.realm:realm-gradle-plugin:2.0.2"
    }
	}

**（2） 在app的build文件加上**

    apply plugin: 'realm-android'

## 三、初始化Realm ##
**（1） 在Application的oncreate()方法中Realm.init()**

    public class MyApplication extends Application {
	  @Override
	  public void onCreate() {
    	super.onCreate();
    	Realm.init(this);
		}
	}

**（2）在Application的oncreate()方法中对Realm进行相关配置**

**①使用默认配置**

    public class MyApplication extends Application {
		@Override
		public void onCreate() {
		    super.onCreate();
		    // The Realm file will be located in Context.getFilesDir() with name "default.realm"
		    Realm.init(this);
		    RealmConfiguration config = new RealmConfiguration.Builder().build();
		    Realm.setDefaultConfiguration(config);
			}
		}

**②使用自定义配置**

	public class MyApplication extends Application {
		@Override
		public void onCreate() {
		    super.onCreate();
		    Realm.init(this);
		    RealmConfiguration config = new  RealmConfiguration.Builder()
                                         .name("myRealm.realm")
                                         .deleteRealmIfMigrationNeeded()
                                         .build();
    		Realm.setDefaultConfiguration(config);
			}
		}

**（3）在AndroidManifest.xml配置自定义的Application**

    <application
		android:name=".MyApplication"
		...
	/>

## 四、创建实体 ##
**（1）新建一个类继承RealmObject**

	    public class Dog extends RealmObject {
		    private String name;
		    private int age;
		
		    @PrimaryKey
		    private String id;
		
		
		    public String getName() {
		        return name;
		    }
		
		    public void setName(String name) {
		        this.name = name;
		    }
		
		    public int getAge() {
		        return age;
		    }
		
		    public void setAge(int age) {
		        this.age = age;
		    }
		
		    public String getId() {
		        return id;
		    }
		
		    public void setId(String id) {
		        this.id = id;
		    }
		}

多对多的关系：

	public class Contact extends RealmObject {
	    public String name;
	    public RealmList<Email> emails;
	}

	public class Email extends RealmObject {
	    public String address;
	    public boolean active;
	}

**（2）其他相关说明**

**1、支持的数据类型：**
boolean, byte, short, int, long, float, double, String, Date and byte[]
在Realm中byte, short, int, long最终都被映射成long类型

**2、注解说明**

**@PrimaryKey**

①字段必须是String、 integer、byte、short、 int、long 以及它们的封装类Byte, Short, Integer, and Long

②使用了该注解之后可以使用copyToRealmOrUpdate()方法，通过主键查询它的对象，如果查询到了，则更新它，否则新建一个对象来代替。

③使用了该注解将默认设置@index注解

④使用了该注解之后，创建和更新数据将会慢一点，查询数据会快一点。

@Required
数据不能为null

@Ignore
忽略，即该字段不被存储到本地

@Index
为这个字段添加一个搜索引擎，这将使插入数据变慢、数据增大，但是查询会变快。建议在需要优化读取性能的情况下使用。

## 五、增 ##
**（1）实现方法一：事务操作**

    类型一 ：新建一个对象，并进行存储

    Realm realm=Realm.getDefaultInstance();
	realm.beginTransaction();
	User user = realm.createObject(User.class); // Create a new object
	user.setName("John");
	user.setEmail("john@corporation.com");
	realm.commitTransaction();


	类型二：复制一个对象到Realm数据库

	Realm realm=Realm.getDefaultInstance();
	User user = new User("John");
	user.setEmail("john@corporation.com");

	// Copy the object to Realm. Any further changes must happen on realmUser
	realm.beginTransaction();
	realm.copyToRealm(user);
	realm.commitTransaction();

**（2）实现方法二：使用事务块**

	Realm  mRealm=Realm.getDefaultInstance();
	final User user = new User("John");
	user.setEmail("john@corporation.com");
	
	mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {

            realm.copyToRealm(user);

            }
        });

## 六、删 ##
    Realm  mRealm=Realm.getDefaultInstance();

    final RealmResults<Dog> dogs=  mRealm.where(Dog.class).findAll();

        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {

                Dog dog=dogs.get(5);
                dog.deleteFromRealm();
                //删除第一个数据
                dogs.deleteFirstFromRealm();
                //删除最后一个数据
                dogs.deleteLastFromRealm();
                //删除位置为1的数据
                dogs.deleteFromRealm(1);
                //删除所有数据
                dogs.deleteAllFromRealm();
            }
        });

同样也可以使用同上的beginTransaction和commitTransaction方法进行删除


## 七、改 ##

    Realm  mRealm=Realm.getDefaultInstance();
	Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
	mRealm.beginTransaction();
	dog.setName(newName);
	mRealm.commitTransaction();

同样也可以用事物块来更新数据

## 八、查 ##
**（1）查询全部**

查询结果为RealmResults<T>，可以使用mRealm.copyFromRealm(dogs)方法将它转为List<T>

    public List<Dog> queryAllDog() {
        Realm  mRealm=Realm.getDefaultInstance();

        RealmResults<Dog> dogs = mRealm.where(Dog.class).findAll();

        return mRealm.copyFromRealm(dogs);
    }

**（2）条件查询**

    public Dog queryDogById(String id) {
        Realm  mRealm=Realm.getDefaultInstance();

        Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
        return dog;
    }

**常见的条件如下（详细资料请查官方文档）：**

**between(), greaterThan(), lessThan(), greaterThanOrEqualTo() & lessThanOrEqualTo()**

**equalTo() & notEqualTo()**

**contains(), beginsWith() & endsWith()**

**isNull() & isNotNull()**

**isEmpty() & isNotEmpty()**


**（3）对查询结果进行排序**

     /**
	  * query （查询所有）
     */
    public List<Dog> queryAllDog() {
        RealmResults<Dog> dogs = mRealm.where(Dog.class).findAll();
        /**
		  * 对查询结果，按Id进行排序，只能对查询结果进行排序
         */
        //增序排列
        dogs=dogs.sort("id");
        //降序排列
        dogs=dogs.sort("id", Sort.DESCENDING);
        return mRealm.copyFromRealm(dogs);
    }


**（4）其他查询**

sum，min，max，average只支持整型数据字段

    /**
     	*  查询平均年龄
     */
    private void getAverageAge() {
         double avgAge=  mRealm.where(Dog.class).findAll().average("age");
    }

    /**
     	*  查询总年龄
     */
    private void getSumAge() {
      Number sum=  mRealm.where(Dog.class).findAll().sum("age");
        int sumAge=sum.intValue();
    }

    /**
     	*  查询最大年龄
     */
    private void getMaxId(){
      Number max=  mRealm.where(Dog.class).findAll().max("age");
        int maxAge=max.intValue();
    }


## 九、异步操作 ##
大多数情况下，Realm的增删改查操作足够快，可以在UI线程中执行操作。但是如果遇到较复杂的增删改查，或增删改查操作的数据较多时，就可以子线程进行操作。

**（1）异步增：**

    private void addCat(final Cat cat) {
      RealmAsyncTask  addTask=  mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                realm.copyToRealm(cat);
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(mContext,"收藏成功");
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(mContext,"收藏失败");
            }
        });

    }

最后在销毁Activity或Fragment时，要取消掉异步任务

    @Override
    protected void onDestroy() {
        super.onDestroy();
       if (addTask!=null&&!addTask.isCancelled()){
            addTask.cancel();
        }
    }

**（2）异步删**

    private void deleteCat(final String id, final ImageView imageView){
      RealmAsyncTask  deleteTask=   mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                Cat cat=realm.where(Cat.class).equalTo("id",id).findFirst();
                cat.deleteFromRealm();

            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(mContext,"取消收藏成功");
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(mContext,"取消收藏失败");
            }
        });

    }

最后在销毁Activity或Fragment时，要取消掉异步任务

    @Override
    protected void onDestroy() {
        super.onDestroy();
       if (deleteTask!=null&&!addTask.isCancelled()){
            deleteTask.cancel();
        }
    }

**（3）异步改**

    RealmAsyncTask  updateTask=   mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                Cat cat=realm.where(Cat.class).equalTo("id",mId).findFirst();
                cat.setName(name);
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(UpdateCatActivity.this,"更新成功");

            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(UpdateCatActivity.this,"失败成功");
            }
        });

最后在销毁Activity或Fragment时，要取消掉异步任务

    @Override
    protected void onDestroy() {
        super.onDestroy();
       if (updateTask!=null&&!addTask.isCancelled()){
            updateTask.cancel();
        }
    }

**（4）异步查**

    RealmResults<Cat>   cats=mRealm.where(Cat.class).findAllAsync();
        cats.addChangeListener(new RealmChangeListener<RealmResults<Cat>>() {
            @Override
            public void onChange(RealmResults<Cat> element) {
               element= element.sort("id");
                List<Cat> datas=mRealm.copyFromRealm(element);

            }
        });

最后在销毁Activity或Fragment时，要取消掉异步任务

    @Override
    protected void onDestroy() {
        super.onDestroy();
        cats.removeChangeListeners();

    }

## 十、数据迁移（版本升级） ##
**方法一、删除旧版本的数据**

    RealmConfiguration config = new RealmConfiguration.Builder()
    .deleteRealmIfMigrationNeeded()
    .build()

**方法二、设置schema 版本和 migration，对改变的数据进行处理**

    RealmConfiguration config = new RealmConfiguration.Builder()
    .schemaVersion(2) // Must be bumped when the schema changes
    .migration(new Migration()) // Migration to run instead of throwing an exception
    .build()

**处理版本数据变化Migration**

	    public class Migration implements RealmMigration {
	
	    @Override
	    public void migrate(final DynamicRealm realm, long oldVersion, long newVersion) {
	        // During a migration, a DynamicRealm is exposed. A DynamicRealm is an untyped variant of a normal Realm, but
	        // with the same object creation and query capabilities.
	        // A DynamicRealm uses Strings instead of Class references because the Classes might not even exist or have been
	        // renamed.
	
	        // Access the Realm schema in order to create, modify or delete classes and their fields.
	        RealmSchema schema = realm.getSchema();
	
	        /************************************************
	            // Version 0
	            class Person
	                @Required
	                String firstName;
	                @Required
	                String lastName;
	                int    age;
	            // Version 1
	            class Person
	                @Required
	                String fullName;            // combine firstName and lastName into single field.
	                int age;
	        ************************************************/
	        // Migrate from version 0 to version 1
	        if (oldVersion == 0) {
	            RealmObjectSchema personSchema = schema.get("Person");
	
	            // Combine 'firstName' and 'lastName' in a new field called 'fullName'
	            personSchema
	                    .addField("fullName", String.class, FieldAttribute.REQUIRED)
	                    .transform(new RealmObjectSchema.Function() {
	                        @Override
	                        public void apply(DynamicRealmObject obj) {
	                            obj.set("fullName", obj.getString("firstName") + " " + obj.getString("lastName"));
	                        }
	                    })
	                    .removeField("firstName")
	                    .removeField("lastName");
	            oldVersion++;
	        }
	
	        /************************************************
	            // Version 2
	                class Pet                   // add a new model class
	                    @Required
	                    String name;
	                    @Required
	                    String type;
	                class Person
	                    @Required
	                    String fullName;
	                    int age;
	                    RealmList<Pet> pets;    // add an array property
	         ************************************************/
	        // Migrate from version 1 to version 2
	        if (oldVersion == 1) {
	
	            // Create a new class
	            RealmObjectSchema petSchema = schema.create("Pet")
	                    .addField("name", String.class, FieldAttribute.REQUIRED)
	                    .addField("type", String.class, FieldAttribute.REQUIRED);
	
	            // Add a new field to an old class and populate it with initial data
	            schema.get("Person")
	                .addRealmListField("pets", petSchema)
	                .transform(new RealmObjectSchema.Function() {
	                    @Override
	                    public void apply(DynamicRealmObject obj) {
	                        if (obj.getString("fullName").equals("JP McDonald")) {
	                            DynamicRealmObject pet = realm.createObject("Pet");
	                            pet.setString("name", "Jimbo");
	                            pet.setString("type", "dog");
	                            obj.getList("pets").add(pet);
	                        }
	                    }
	                });
	            oldVersion++;
	        }
	
	        /************************************************
	            // Version 3
	                class Pet
	                    @Required
	                    String name;
	                    int type;               // type becomes int
	                class Person
	                    String fullName;        // fullName is nullable now
	                    RealmList<Pet> pets;    // age and pets re-ordered (no action needed)
	                    int age;
	         ************************************************/
	        // Migrate from version 2 to version 3
	        if (oldVersion == 2) {
	            RealmObjectSchema personSchema = schema.get("Person");
	            personSchema.setNullable("fullName", true); // fullName is nullable now.
	
	            // Change type from String to int
	            schema.get("Pet")
	                .addField("type_tmp", int.class)
	                .transform(new RealmObjectSchema.Function() {
	                    @Override
	                    public void apply(DynamicRealmObject obj) {
	                        String oldType = obj.getString("type");
	                        if (oldType.equals("dog")) {
	                            obj.setLong("type_tmp", 1);
	                        } else if (oldType.equals("cat")) {
	                            obj.setInt("type_tmp", 2);
	                        } else if (oldType.equals("hamster")) {
	                            obj.setInt("type_tmp", 3);
	                        }
	                    }
	                })
	                .removeField("type")
	                .renameField("type_tmp", "type");
	            oldVersion++;
	        }
	    }
	}

