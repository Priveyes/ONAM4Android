# ONAM4Android

ONAM (Object Nested Access Management) is a lightweight ORM & Persistence API for Sqlite. ONAM4Android is designed to work seamlessly with android projects for ORM and data persistence.

### Prerequisites

Requires a Gradle based project in Android Studio with SQLite. Minimum supported SDK version is 14.

### Installing

Add Jitpack to gradle -- preferably in app.gradle

```
repositories {
    .....
    maven {
        url "https://jitpack.io"
    }
}
```

And
```
dependencies {
    compile 'com.github.basilgregory:ONAM4Android:2.1'
}
```


## How to integrate with your project - Based on demo application in code base

Lets consider the below database requirements.  
Database Name: **blog_db**  
Tables: **post, comment, user**.  


### @DB 

You need to define @DB only once in your launching Activity.

```
@DB(name = "blog_db",
        tables = {Post.class,Comment.class,User.class
}, version = 1)
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    ...
}
```

### Entity.init()

You need to call init() function in same class where @DB annotation is defined.

If @DB annonation is defined in Activity Class file.
```
@DB(name = "blog_db",
        tables = {Post.class,Comment.class,User.class
}, version = 1)
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Entity.init(this);
    }
    ...
}
```
Incase @DB annonation is defined in separate Class file.

```
@DB(name = "blog_db",
        tables = {Post.class,Comment.class,User.class
}, version = 1)
public class InitClass {

    public InitClass(Activity activity) {
        Entity.init(activity, this);
    }
    ...
}
```

For information on how to activate logs [Activate Logs](https://github.com/basilgregory/ONAM4Android/wiki/Logs) using log()

## Tables

**ONAM** provides 2 kinds of interfaces to take care of you all your ORM requirements. One an abstract class named **Entity** and a series of annotations that you need to integrate in your code.

The current **blog_db** database has 3 entities (or tables). So lets create 3 classes extending **Entity** class, and add the annotation **@Table** on each of them.

```
@Table
public class Post extends Entity {
    .....    
}

@Table
public class Comment extends Entity {
    .....    
}

@Table
public class User extends Entity {
    .....    
}
```

## Columns

## Primary Key
The primary key in ONAM for all tables are maintained internally (auto generated, incrementing), you will be able to access them using getId() method, but not overwrite or alter its value.

Now add columns as class fields.

```
@Table
public class Post extends Entity {
    private String title;
    private String post;
    private long createdAt;
    private List<User> followers;
    private List<Comment> comments;
    private User user;
    
    private transient String transientPost;

    .....
}
```

## Unique fields
You may set a field to be unique using @Column(unique= true) in its corresponding getter function, and if you try to insert or update a duplicate value, 'android.database.sqlite.SQLiteConstraintException: UNIQUE constraint failed' exception will be thrown. Even NULL value is allowed only once if unique= true is set.

## Transient fields

All fields will be converted to database columns. All fields with transient modifier *( in this case* private transient String transientPost *)* will be ommited out. You may use such fields to do your bidding at freewill.

Now generate getter and setters for all fields ( This is mandatory for all fields except transient fields).


## Mappings

### Types of mappings that are needed,
Post has ManyToOne mapping with User (as owner of post)  
Post has OneToMany mapping with Comment (as comments of post)  
Post has ManyToMany mapping with User (as followers of post)  

For more on entity mappings, see [wiki on Mappings](https://github.com/basilgregory/ONAM4Android/wiki/Entity-Mappings)


## Getters and Setters

You need to generate getters and setters of all fields that has to converted to columns.

```
    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getPost() {
        return post;
    }

    public void setPost(String post) {
        this.post = post;
    }

    public long getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(long createdAt) {
        this.createdAt = createdAt;
    }

    @OneToMany(referencedColumnName = "post_id", targetEntity = Comment.class)
    public List<Comment> getComments() {
        return fetch(this.comments,new Comment(){});
    }

    public void setComments(List<Comment> comments) {
        this.comments = comments;
    }

    @ManyToOne
    public User getUser() {
        return fetch(this.user,new User(){});
    }

    public void setUser(User user) {
        this.user = user;
    }

    @ManyToMany(tableName = "post_followers", targetEntity = User.class)
    public List<User> getFollowers() {
        return fetch(this.followers,new User(){});
    }

    public void setFollowers(List<User> followers) {
        this.followers = followers;
    }

    public String getTransientPost() {
        return transientPost;
    }

    public void setTransientPost(String transientPost) {
        this.transientPost = transientPost;
    }
```

In example code above, there are changes made to getter functions of related entities, these changes are mandatory for related entities.
```
    @ManyToOne
    public User getUser() {
        return fetch(this.user,new User(){});
    }

    @OneToMany(referencedColumnName = "post_id", targetEntity = Comment.class)
    public List<Comment> getComments() {
        return fetch(this.comments,new Comment(){});
    }
    
    @ManyToMany(tableName = "post_followers", targetEntity = User.class)
    public List<User> getFollowers() {
        return fetch(this.followers,new User(){});
    }
```
See [wiki on Mappings](https://github.com/basilgregory/ONAM4Android/wiki/Entity-Mappings) to know how to implement the same.


Custom table name and column names may be specified for table creation using @Column(name = "column_name") and @Table(name = "table_name") annotations.
See [migration docs](https://github.com/basilgregory/ONAM4Android/wiki/Migration) on how to implement the same.

## Save/ Update
You may save an entity by calling save() method. The decision to insert or update will be taken care of by ONAM. If you want to update an entity, be sure to fetch it first using any of the Query methods.

### Save
```
    Post post = new Post();
    post.setTitle(postTitle.getText().toString());
    post.setPost(postDescription.getText().toString());
    User registeredUser = User.find(User.class,1);
    post.setUser(registeredUser);
    post.save(); //Here insertion operation will be carried out
```
### Update
```
   User user = User.findByUniqueProperty(User.class, "name", "John Doe");
   user.setBio("Updated Bio");
   user.save(); //Here update operation will be carried out
```

## Delete

Calling delete() method will delete that particular row. It will return true/ false depending on the success of the operation
```
    
    User user = User.findByUniqueProperty(User.class, "name", "John Doe");
    if (user == null) return; //to make sure that user exists
    boolean deleteSuccess = user.delete();
```

## Truncate
Entity.truncate()  will delete all rows in a particular table. It will return true/ false depending on the success of the operation
```
    boolean truncateSuccess = Post.truncate(Post.class);
```


## Query rows

### Find by ID
In-order to find a row by its id,
```
   User user = User.find(User.class, 1);
```
This will return null, if the row doesn't exist.

### Check if a row exists/ Retrieve a single row.
```
   User user = User.findByUniqueProperty(User.class, "name", "John Doe");
```
This will return a single row if it exists with value "John Doe" in column "name".
Custom column names may be specified for tables using @Column(name = "column_name") annotation.
See [migration docs](https://github.com/basilgregory/ONAM4Android/wiki/Migration) on how to implement the same.

### Find by property

To find all rows with value "John Doe" in column "name".
```
   List<User> users = User.findByProperty(User.class, "name", "John Doe");
```

## Find All

To get all rows.
```
   List<User> users = User.findAll(User.class);
```

## WHERE Clause
Incase, you need to specify a custom where clause.

Use where clause, use:
```
   String nameOfUser = "John Doe";
   String whereClause = "name == " + nameOfUser;
   List<User> users = User.findByProperty(User.class, whereClause);
```


See [query docs](https://github.com/basilgregory/ONAM4Android/wiki/Query-from-table) to find how to add limit, orderby clauses.

## Lifecycle Events

You can define methods that will be executed before create, before update, after create or after update using corresponding annotations.
See [Lifecycle docs](https://github.com/basilgregory/ONAM4Android/wiki/Lifecycle) on how to implement the same.

## JSON parser

We have added support to convert Entity objects/delegate objects to JSONObject/JSONArray.

```
    JSONObject postObject = Entity.toJSON(post);
```
were 'post' is object of **Post** Entity, and this will return a JSONObject with the values mapped to corresponding fields.  
For detailed information on how to specify the fields to be marked for JSON convertion [JSONParser Docs](https://github.com/basilgregory/ONAM4Android/wiki/JSON-Parser)


### Contributing

Please read [CONTRIBUTING.md](https://github.com/basilgregory/ONAM4Android/blob/master/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/basilgregory/ONAM4Android/tags). 

### Authors

* **Robin Alex Panicker** - [BasilGregory Software Labs Pvt Ltd](https://github.com/rp-bg) 
* **Don Peter** - [BasilGregory Software Labs Pvt Ltd](https://github.com/dp-bg)

### Contact

email: onam@basilgregory.com

### License

This project is licensed under the MIT License - see the [LICENSE.md](https://github.com/basilgregory/ONAM4Android/blob/master/LICENSE) file for details

