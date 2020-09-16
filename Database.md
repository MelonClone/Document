# Watermelon Framework

## Database Usage

### Dir
```
java
└ [packages]
  └ domain
    └ [domain_name]
      └ dao
        └ [daos] // for DAO
      └ domain
        └ [domain_items]
      └ model
        └ [data_source] // for DAO executor
        └ [models] // for DB & SharedPreference
        ...
      ...
    ...
  └ global
    └ db
      └ version
        └ AppDatabase.java // 
        └ VersionMigration.java //
        ...
      ...
    ...
  ...
```

## Database init

To initiate the Room database create AppDatabase.java in /global/db/version. 
And the `MyApplication.java` have to initiate `DatabaseInitiator.java`.

```java
import this.is.my.packages.global.db.version.AppDatabase;

public class MyApplication extends BaseApplication {

    public void initiate() {
        Initializer.addInitiator(new MyDatabaseInitiator());
        // ...
    }
    // ...
}

```


## Database Migration
If your app need migration to the next version. Create `MyMigrationContainer.java` at /global/db/version as the `AppDatabase.java`.
And add the container in BaseApplication.

```java
import this.is.my.packages.global.db.version.MyMigrationContainer;

public class MyDatabaseInitiator extends BaseApplication {
    // ...
    @Override
    public MigrationContainer getMigrationContainer() {
        return new MyMigrationContainer();
    }
}
```

```java
public class MyMigrationContainer extends MigrationContainer {

    private static List<Migration> migrationList = new ArrayList<>();

    private static void init() {
        migrationList.add(MIGRATION_1_2);
    }

    public MyMigrationContainer() {
        init();
    }

    @Override
    public List<Migration> getMigrationList() {
        return this.migrationList;
    }

    static final Migration MIGRATION_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            database.execSQL("CREATE TABLE `my_table` (" +
                    "`id` INTEGER, " +
                    "`name` TEXT, " +
                    "`age` INTEGER, " +
                    "PRIMARY KEY(`id`))");
        }
    };
    //...
}
```

## Database CRUD

Add DAOs

```java
@Dao
public interface UserDao extends BaseDao {
    @Query("SELECT * FROM user_table LIMIT 1")
    User getUser();

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertUser(User user);
}
```

and make dao caller

```java
public class LocalUserDataSource implements UserDataSource {

    private final UserDao userDao;

    public LocalUserDataSource(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void insertOrUpdateUser(User user, DatabaseCallback daoResultCallback) {
        DaoCallback.execute(new DaoCallback() {
            public Domain execute() {
                userDao.insertUser(user);
                return null;
            }

            public void postExecute(Domain d) {
                Message msg = new Message();
                msg.arg1 = INSERT_USER;
                daoResultCallback.callback(msg);
            }
        });
    }

    @Override
    public void getUserInfo(DatabaseCallback daoResultCallback) {
        DaoCallback.execute(new DaoCallback() {
            public Domain execute() {
                return userDao.getUser();
            }

            public void postExecute(Domain d) {
                Message msg = new Message();
                msg.arg1 = GET_USER;
                msg.obj = d;
                daoResultCallback.callback(msg);
            }
        });
    }
}
```

then you can use this way

```java
public class LoginViewModel extends MelonCloneBaseViewModel {
    // ...
    private UserDataSource userDataSource;

    @Override
    protected void init() {
        // ...
        userDataSource = new LocalUserDataSource(((AppDatabase) DBHelper.getInstance().getDB()).userDao());
    }

    public void updateUser(User user) {
        userDataSource.insertOrUpdateUser(user, message -> {
            // TODO LiveData control
        });
    }
}
```