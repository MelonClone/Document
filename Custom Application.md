# Watermelon Framework

## Custom Application Usage

### Dir
```
java
└ [packages]
  ...
  └ global
    └ consts
      └ Constants.java
    └ initiator
      └ [Initiators]
  MyApplication
```

## Application init

To initiate the BaseApplication create MyApplication.java in /global.
And the `MyApplication.java` have to implement `initate()` method.
In this method you should add initators into `Initializer`.
The initiator can be `DatabaseInitiator`, `SharedPreferenceInitiator`, `FontInitiator` or any of `Initializable` implementation.
It will be classfied by tag names.

```java
package this.is.my.packages;

import this.is.my.packages.global.consts.Constants;
// ...

public class MyApplication extends BaseApplication {

    public void initiate() {
        Initializer.addInitiator(new MyDatabaseInitiator());
        Initializer.addInitiator(new MySPInitiator());
        Initializer.addInitiator(new MyFontInitiator());
        Initializer.addInitiator(new MyNetworkInitiator());
    }
    
    // ...
    
}
```

```java
// (MyDatabaseInitiator.java)
public class MyDatabaseInitiator extends DatabaseInitiator {

    @Override
    public String getDatabaseName() {
        return Constants.DB_NAME;
    }

    @Override
    public MigrationContainer getMigrationContainer() {
        return new VersionMigration();
    }

    @Override
    public <D extends RoomDatabase> Class<D> getDatabaseClass() {
        return (Class<D>) AppDatabase.class;
    }
}

// (MySPInitiator.java)
public class MySPInitiator extends SharedPreferenceInitiator {

    @Override
    public String getSharedPreferenceName() {
        return Constants.SP_NAME;
    }
}

// (MyFontInitiator.java)
public class MyFontInitiator extends FontInitiator {

    @Override
    public String getFontLocation() {
        return Constants.FONT_LOCATION;
    }
}

// (MyNetworkInitiator.java)
public class MyNetworkInitiator implements Initializable {

    private boolean isInit = false;

    @Override
    public void init(Context context) {
        // Global http settings
        HttpManager.setBaseUrl(API_PROTOCOL + API_SERVER);
        isInit = true;
    }

    @Override
    public boolean isInit() {
        return isInit;
    }

    @Override
    public String getTag() {
        return "NETWORK";
    }
}

```

And use this class in manifest file.
