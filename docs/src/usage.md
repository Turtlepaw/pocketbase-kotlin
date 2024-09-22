# Usage

### Client

The `PocketbaseClient` is the class used to access the Pocketbase API.

```kotlin
// Creates a new pocketbase client with the given URL
// The client is used to access everything in the Pocketbase API
val client = PocketbaseClient({
    protocol = URLProtocol.HTTP // or URLProtocol.HTTPS (recommended for production)
    host = "localhost"
    port = 8090 // (optional)
})
```

### Logging in

Logging the client in allows it to call methods that require Pocketbase authentication.
The current login token is saved and applied to every api request.

#### Auth Store

If you want to persistently store the token, you can extend `BaseAuth` and store the token using `SharedPreferences` or another database:

```kotlin
// Using SharedPreferences
const val TOKEN_KEY = "token"
class AuthStore(var sharedPreferences: SharedPreferences) : BaseAuthStore(null) {
    init {
        this.token = sharedPreferences.getString(TOKEN_KEY, null)
    }

    override fun save(token: String?) {
        super.save(token)
        sharedPreferences.edit()
            .putString(TOKEN_KEY, token)
            .apply()
    }

    override fun clear() {
        super.clear()
        sharedPreferences.edit()
            .putString(TOKEN_KEY, null)
            .apply()
    }
}
```

```kotlin
// Set store in client settings
val client = PocketbaseClient({
    //...
}, store = AuthStore(sharedPreferences))
```

#### Logging in via username/email and password

```kotlin
// Logging in as an admin
client.admins.authWithPassword("email", "password").token

// Using the collection auth service to log in
// This authenticates a user rather than an admin
// The user auth service is the same as collection auth, but it uses the "users" collection
client.records.authWithPassword<User>("collectionName", "email", "password").token
client.records.authWithUsername<User>("collectionName", "username", "password").token
```

#### Logging in via OAuth

```kotlin
//You can also use oauth2
client.records.authWithOauth2<User>("collectionName", "provider", "code", "codeVerifier", "redirectUrl").token
```

#### Logging in via token

```kotlin
client.login { token = "xxx" }
```

#### Logging out

```kotlin
client.logout()
```

### Using the Pocketbase API

```kotlin
// We are using this class to hold records created in the collection "people".
// Each record has a name (required), age (required number) and a pet optional.
// If a value is optional in the collection's schema make sure the kotlin type is nullable and defaults to null.
// All record classes must be @Serializable and extend Record.
@Serializable
data class PersonRecord(val name: String, val age: Int, val pet: String? = null): Record()

val recordToCreate = PersonRecord("Sky", 4)
// Generics are used to define the return type of API calls, just make sure that the record class you created matches the collection's schema.
val response = client.records.create<PersonRecord>("people", Json.encodeToString(recordToCreate))
// You can now use the data from the response.
val collectionId = response.collectionId
```

#### Types

| Pocketbase Type                   | Kotlin Type  |
| --------------------------------- | ------------ |
| Plain text, email, rich text, Url | `String`     |
| Number                            | `Int`        |
| Bool                              | `Boolean`    |
| Select, relation                  | `List<T>`    |
| Json                              | `JsonObject` |

### Congratulations

**All done!** Now that you have a Pocketbase client set up and know how to log in, the rest is easy!
Simply follow the [Pocketbase Web API Docs](https://pocketbase.io/docs/api-records/) and our internal KDocs to find your
way around the Pocketbase API.
There are a few exceptions for which features could not be made to match the other SDKs, for a guide on them go to
our [caveats page](caveats.md)
