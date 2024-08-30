# :warning: This repository is no longer maintained :warning:

# Dolby.io Platform Token Service

A simple token server function to secure the Dolby.io API, suitable for Native Mobile (iOS and Android) and web applcations. We'll use Netlify for this example.  This sample application creates a simple token server to secure your application calls to the Dolby.io API.

[![Deploy To Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/dolbyio-samples/comms-sdk-platform-token-service)

Click the **Deploy to Netlify** button; Authenticate with GitHub and then supply your application's unique **identifier** and your Dolby.io Consumer key and secret. These values will be used to setup the **Environment** variables on the Netlify server.   This configuration uses that unique app identifier as a gate for the web service.  All requests to this serverless endpoint will require the request to be a POST and contain a header called **appidentifier** that matches the values you entered for the environmental variable. 

Suggested Identifiers:

iOS:
Use the bundle identifier.
 
Android:
Use the package identifier.

## How to use the unique identifier to help secure the token service


Within your native application code you'll use a POST request with the SAME appidentifier that you configured for this token service in the request headers. In the case of iOS native apps, you'll use the bundle identifier which is unique to each app. Similarly for Android, you'll use the package identifier.

## iOS

```Swift
// AppDelegate.swift  iOS Example
// this function fetches the token by a post request with the app identifier, 
// if matching on the service then a token is returned.
    func fetchSecureToken(completion: @escaping (_ token: String?) -> Void) {
        let serverURL = "<ENTER THE URL TO YOUR DEPLOYED TOKEN SERVICE>"  
        let bundleID = Bundle.main.infoDictionary?["CFBundleIdentifier"] as! String
        
        let url = URL(string: serverURL)!
        var request = URLRequest(url: url)
        let headers = [
            "appidentifier": bundleID
        ]
        request.allHTTPHeaderFields = headers
        request.httpMethod = "POST"
        request.timeoutInterval = 60

        
        let task = URLSession.shared.dataTask(with: request) { data, _, _ in
            if let data = data,
               let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any],
               let accessToken = json["access_token"] as? String {
                completion(accessToken)
            } else {
                completion(nil)
            }
        }
        task.resume()
        
    }

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Fetch access token.
        fetchSecureToken { accessToken in
            guard let token = accessToken else { return }
            
            // Voxeet SDK OAuth initialization.
            VoxeetSDK.shared.initialize(accessToken: token) { (refreshClosure, isExpired) in
                // VoxeetSDK calls this closure when the token needs to be refreshed.
                self.fetchSecureToken { accessToken in
                    guard let token = accessToken else { return }
                    // Call the SDK’s refresh closure with the new token
                    refreshClosure(token)
                }
            }
        }
        VoxeetSDK.shared.notification.push.type = .none
        VoxeetSDK.shared.conference.defaultBuiltInSpeaker = true
        VoxeetSDK.shared.conference.defaultVideo = false
        return true
    }
    
```
## Android

```kotlin
// MainActivity.kt  Android Example
// this function fetches the token by a post request with the package identifier, 
// if matching on the service then a token is returned.
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Fetch access token.
        fetchSecureToken { accessToken ->
            // Voxeet SDK OAuth initialization.
            val token = accessToken ?: return@fetchSecureToken
            VoxeetSDK.initialize(token) { _, callback ->
                fetchSecureToken { accessToken ->
                    // Call the SDK’s refresh closure with the new token
                    val token = accessToken ?: return@fetchSecureToken
                    callback.ok(token)
                }
            }
        }
    }

    private fun fetchSecureToken(callback: (token: String?) -> Unit) {
        val serverURL = "<ENTER THE URL TO YOUR DEPLOYED TOKEN SERVICE>"
        // using Volley to initialize a queue
        val queue = Volley.newRequestQueue(this@MainActivity)

        // setting up secure POST request with package identifier in header
        var request = object : JsonObjectRequest(
            Request.Method.POST, serverURL, null,
            Response.Listener { response ->
                val token = response.get("access_token")
                Log.d("MainActivity", "Access token is: $token")
                callback(token as String)
            },
            Response.ErrorListener {
                Log.e("MainActivity", "Request failed: ${it.localizedMessage}")
                callback(null)
            }) {
            override fun getHeaders(): MutableMap<String, String> {
                val headers = HashMap<String, String>()
                headers["appidentifier"] = applicationContext.packageName
                return headers
            }
        }

        queue.add(request)
    }
}
```

For this Android fetch request example, we used the [Volley HTTP networking library](https://google.github.io/volley/). 

In order to test, make sure to add the Volley dependency in your ``build.gradle(:app)`` file.
```gradle
dependencies {
    implementation 'com.android.volley:volley:${version}'
}
```
## Web  

For web deployment, you'd deploy this code as a function hosted on the same domain of the web app, use "web" as the package identifier value and the function will use the hosted domain for the identifier. Your web application would be deployed to the WWW folder in this example.

  
  
## Install

 If you followed along with the Click to deploy to Netlify, your token service should be live and running.  The code was also cloned to your GitHub Account.
 
 If you want to make further edits to the token service code- pull down the newly cloned repo to continue developing the token server code. 
 
Run the following command to install the dependencies:
 
```bash
 npm install
```
  

## Run
 
Run the Netlify live server with the command :
 
```bash
npm run live
```
*This deploys a preview of the project and a tunnel to push live changes to that preview. 
You can make a change and refresh your site to see the results.*
