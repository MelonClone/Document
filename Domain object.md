# Watermelon Framework

## Domain tree Usage

### Dir
```
java
└ [packages]
  └ domain
    └ [some domain]
      └ dao
        └ [Room Dao Classes]
      └ domain
        └ [Domain Objects]
      └ model
        └ [data_source] // for DAO executor
        └ [models] // for DB & SharedPreference
      └ view
        └ activity
          └ [Activities extends BaseActivity] // for Activity, Event Emitter
        └ adapter
          └ [List Adapters] // Such as recyclerview adapter or View pager adapter
        └ fragment
          └ [Fragments and Views] // Each fragment have their own Lifecycle. 
      └ viewmodel
        └ [View models extends BaseViewModel] // 
  ...
```

[View] <-> [ViewModel] <--Callback--> [Model(DataSource)] <--Callback--> [Dao] <--Domain--> [RoomDatabase]

## Usage 

### DAO
#### Path
/pacakgePath/domain/[domainName]/dao

#### Desc
This dao class is room DAO.
If you use room database. The MainApplication will initiate RoomDatabase.class. And AppDatabase which your RoomDatabase class will find the DAOs.

The DataSource of Model have some DAO objects. That will call the Room by annotation.

[App init] -> [Room init]

[Model(DataSource)] <-> [Dao] <-> [RoomDatabase]

</br>
- - -
</br>

### Domain
#### Path
/pacakgePath/domain/[domainName]/domain

#### Desc
POJO class
implements `org.watermelon.framework.global.model.domain.Domain`, `androidx.room.Entity`

the Domain is handled by viewModel. 

[ViewModel] <--Callback--> [Model(DataSource)] <--Callback--> [Dao] <--Domain--> [RoomDatabase]

```java
public class SomeDomain implements Domain {
    // Entity init
}
```

</br>
- - -
</br>

### Model
#### Path
/pacakgePath/domain/[domainName]/model

#### Desc
There are 3 types model data model, Repository (implements `org.watermelon.framework.global.model.repository.Repository`) and DataSource

The Repository is HTTP Model. So they are send http messages to the server and callback to viewModel the answers.
The DataSource is Database Model. So it call the local database(SQLite, Room) also callback to viewModel the result of database as well.
The Model is abstracted class. If you need some static datas by files or anything.

</br>
- - -
</br>

### View (Activity)
#### Path
/pacakgePath/domain/[domainName]/view/activity

#### Desc
Mother View class. It sharing its lifecycle and context.
Activity is actual screen (users can see).
It is emmited events (Such as a click, a drag or a pinch etc...) by users.
The View have viewModels and Fragments.

[UserInteraction] <-> [View] <-> [ViewModel]
                             <-> [Fragment, CustumView, Adapter]

#### Code

```java
public class SomeActivity extends BaseActivity {
    
    @Override
    protected void layoutInit() {
        // Layout binding
    }

    @Override
    protected void toolbarInit() {
        // Override if need toolbar
    }

    @Override
    protected void viewInit() {
        // View behavior, state init
    }

    @Override
    protected void viewModelInit() {
        // ViewModel init
    }

    @Override
    protected void listenerInit() {
        // EventListener init
    }
}
```

</br>
- - -
</br>

### View (Adapter)
#### Path
/pacakgePath/domain/[domainName]/view/adapter

#### Desc
Adapter is view but it has own view layout.
Also the layout can be changed without activity control.
The Activity hand over thier viewModels to adapter.
If adapter need data they can use viewModel.

[View] <-> [Adapter] <-> [ViewModel]
                     <-> [Fragment, CustumView]

</br>
- - -
</br>

### View (Fragment)
#### Path
/pacakgePath/domain/[domainName]/view/fragment

#### Desc
Fragment is slice of activity.
It is shown on each layout(or view) by activity.
This has own lifecycle but it exist only inside of activity.

[Activity] <-> [Fragment] <-> [ViewModel]
                          <-> [Adapter]


#### Code

```java
public class SearchMainFragment extends BaseFragment {

    // Views
    RecyclerView newestMusicView;
    NewestMusicAdapter newestMusicAdapter;
    UltraViewPager rankingViewPager;

    // ViewModels
    NewestMusicViewModel newestMusicViewModel;
    RankingViewModel rankingViewModel;

    @Override
    protected View viewContainerInit(LayoutInflater inflater, ViewGroup container) {
        return inflater.inflate(R.layout.some_layout, container, false);
    }

    @Override
    public void setParentViewModel(ViewModel... viewModels) {
        for (ViewModel viewModel : viewModels) {
            // ViewModel init
            if (viewModel instanceof NewestMusicViewModel) {
                newestMusicViewModel = (NewestMusicViewModel) viewModel;
            }
        }
    }

    @Override
    protected void layoutInit(View view) {
        // Layout binding
    }


    @Override
    protected void viewInit() {
        // View behavior, state init
    }

    @Override
    protected void viewModelInit() {
        // ViewModel init
        rankingViewModel = new ViewModelProvider(this).get(RankingViewModel.class);

        newestMusicViewModel.getList().observe(this, list -> {
            // Do something
        });

        rankingViewModel.getList().observe(this, list -> {
            // Do something
        });
    }

    @Override
    protected void listenerInit() {
        // EventListener init
    }
}
```

</br>
- - -
</br>

### ViewModel
#### Path
/pacakgePath/domain/[domainName]/viewmodel

#### Desc
Main buiseness logic will be here.
the View event process. Model calling. Callback control etc.
This ViewModel extends androidx.lifecycle.ViewModel. So it has own lifecycle. Also it can be shared with the other views.

 [View] <-> [ViewModel] <-- Callback --> [Model]
 [View] <->

#### Code

```java
public class LoginViewModel extends BaseViewModel<{Auth object}> {

    // LiveDatas
    private MutableLiveData<AuthState> loginInfoText;

    // models
    private UserRepository userRepository;
    private UserDataSource userDataSource;

    // ...


    @Override
    protected void beforeInit() {
        super.beforeInit();
    }

  
    @Override
    protected void init() {
        loginInfoText = new MutableLiveData<>();
        userDataSource = new UserDataSource()
        userRepository = UserRepository.getInstance();
    }

    // ...
}
```