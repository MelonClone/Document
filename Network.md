# Watermelon Framework

## Network

### Dir
```
java
└ [packages]
  └ domain
    └ [some domain]
      └ model
        └ [repositoies]
        ...
  └ utils
    └ network
      └ services
        └ [service]
  ...
```

[ViewModel] <--RepoCallback--> [Model(Repository)] <--Callback--> [HttpManager] <--Retrofit-----------> [network.services.Service]
                                                                                <--HttpUrlConnection--> [Target URL, Method]

## Usage 

### Repository
#### Path
/pacakgePath/domain/[domainName]/model

#### Desc
This http network class.
By HttpManager which is manage http libraries the Repository request http api.
Also the request callback respose data.
If the viewModel who called this repository need response data, call a repoCallbackwith NetworkState.
If you want change http library you should edit the HttpManager to use that you want.

[ViewModel] <-> [Model(Repository)] <-> [HttpManager]

#### Code

```java
public class SomeRepository implements Repository {

    public void getSomeInfo(Some someObject, final RepoCallback<Some> repoCallback) {
        HttpManager.createService(SomeService.class)
            .getSome(someObject.getId())
            .enqueue(new Callback<Some>() {
                @Override
                public void onResponse(@NonNull Call<Some> call, @NonNull Response<Some> response) {
                    if (response.isSuccessful()) {
                        repoCallback.success(response.body());
                    } else {
                        repoCallback.fail(new NetworkState(response.code(), response.message()));
                    }

                    @Override
                    public void onFailure(@NonNull Call<Some> call, @NonNull Throwable t) {
                        repoCallback.fail(new NetworkState(0, t.toString()));
                    }
                }
            });
    }
}
```

</br>
- - -
</br>

### Service
#### Path
/packagePath/utils/network/services

#### Desc
Services are list of api end-points.
If use retrofit it should be interface and describe the retrofit spec.
If use HttpUrlConnection it should explain url, method, headers and the other http spec.

[HttpManager] <-> [network.services.Service]
      ↕
[HTTP Server]

There are two types of Service
1. Retrofit2 Service
2. HttpURLConnection Service


#### Code (Retrofit2)

If you want to use Retrofit services then extend interface `NetworkService`

```java
public interface SampleRetrofitUserService extends NetworkService {
    // ...
    @Headers({
            "Accept: application/json"
    })
    @GET(MUSIC_PRESET+"/{music_id}")
    Call<User> getUser(
            @Path("user_id") String userId
    );
}
```

then you can use it this way

```java
public class UserRepository implements Repository {
    // ...
    public void getUserById(User user, final RepoCallback<Message> repoCallback) {
        HttpManager.createService(SampleRetrofitUserService.class)
                .getUser(user.getUserId())
                .enqueue(new Callback<Message>() {
                    @Override
                    public void onResponse(@NonNull Call<Message> call, @NonNull Response<Message> response) {
                        if (response.isSuccessful()) {
                            repoCallback.success(response.body());
                            return;
                        }
                        repoCallback.fail(new NetworkState(response.code(), response.message()));
                    }

                    @Override
                    public void onFailure(@NonNull Call<Message> call, @NonNull Throwable t) {
                        repoCallback.fail(new NetworkState(0, t.toString()));
                    }
                });
    }
}
```


#### Code (HttpURLConnection)

If you want use HttpURLConnection extends class HttpConnectionServiceImpl

```java
public class SampleHttpUserService extends HttpConnectionServiceImpl {
    // ...
    public HttpSender.Call<User> getUser() {
        getHttpSender().setMethod(HttpConnector.HttpMethod.GET);
        return callback ->
                getHttpSender()
                    .call(connection -> {
                        InputStream is = null;
                        BufferedReader br = null;
                        StringBuilder sb = new StringBuilder();

                        try {
                            connection.setDoOutput(false);

                            if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                                is = connection.getErrorStream();
                            } else {
                                is = connection.getInputStream();
                            }

                            br = new BufferedReader(new InputStreamReader(is, "UTF-8"));

                            String line = null;

                            while ((line = br.readLine()) != null) {
                                sb.append(line);
                            }
                        } catch (UnsupportedEncodingException e) {
                            e.printStackTrace();
                        } catch (IOException e) {
                            e.printStackTrace();
                        } finally {
                            try {
                                if (br != null) br.close();
                                if (is != null) is.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }

                        return User.parse(sb.toString());
                    }, callback);
    }
}
```

then you can use this way.
It same as retrofit but the `getUser()` returns it's own caller and uses own callback.

```java
public class UserRepository implements Repository {
    // ...
    public void getUserById(User user, final RepoCallback<Message> repoCallback) {
        HttpManager.createService(SampleHttpUserService.class)
                .getUser()
                .execute(new HttpSender.Callback<Message>() {
                    @Override
                    public void onResponse(HttpURLConnection response, User user) {
                        repoCallback.success(user);
                    }
                });
    }
}
```