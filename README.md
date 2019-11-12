# Redemption Platform

## Introduction

Welcome to Iguama's Redemption Platform (from now on referred to as "The Platform") documentation. Our platform allows your customers to redeem their reward points in the digital merchant of their choice, in order to achieve that, there are some API requirements that need to be provided by any loyalty program interested in integrating to our platform.

## Overall flow

The overall process is composed of three steps:
* Integration
* Authentication
* Redemption
* Redemption Reversal

## Integration
### IOS Integration

* import code

Step 1: 

Start the IDE such as Xcode, copy the files in the file into the folder and import them into the project.

Step 2:

 Add import to where IGWebSiteSDK is required
 ```bash
 #import IGWebSiteSDK;
 ```
 * Configuration
 
 Open "AppDelegate.swif" file, and edit the following three parameters:
    
```bash
     func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {        
             // clientId、clientKey、sdkLogoImage、isSandbox
     IGInteractionMethod.config(clientId: "IOS_LATAMPASS_ID", clientKey: "IOS_LATAMPASS_KEY", sdkLogoImage: sdkLogoImge, isSandbox: false)
             return true
     }
```
 Parameters | Type/Required | Description
 ----- | ---- | -----------
     clientId | string/required | Loyal program’s identity Id for SDK.
     clientKey | string/required |  Loyal program’s secret key  for SDK
     sdkLogoImage | file/required | logo icon.
     isSandbox | string/required | True is for sandbox test, false is for prod evn.
 
  * User Authorization
  
a.user from app
   ```bash
  let interactionMethod = IGInteractionMethod(member_id: member_id, first_name: first_name, last_name: last_name, email: email, balance: balance, access_token: access_token)
  interactionMethod.login()
  ```

b.user from website
   ```bash
  let interactionMethod = IGInteractionMethod()
  interactionMethod.login()
  ```
  
  ### Android Integration
  
  * import code
  
  Step 1: 
  
  import com.iguama.redemptions.aar into libs folder in the merchant project, as shown in the figure below. Go to the project's Java Build Path and import com.iguama.redemptions.aar under libs. Select Order and Export, and check com.iguama.redemptions.aar.
  
  Step 2:
  
  In he build.gradle of your main project, add the following content to make the libs directory a dependent repository:
   ```bash
   allprojects {     
                  repositories {                
                      flatDir {             
                          dirs 'libs'         
                      }          
                   } 
              }
   ```
   
   Step 3:
   In the build.gradle of your App Module, add the following to the com.iguama.redemptions SDK as a project dependency:
   
```bash
      dependencies {    
                      compile (name: 'com.iguama.redemptions', ext: 'aar') 
      }
```
      
      
   * Configuration

Add the following declaration into file AndroidManifest.xml in the project:
    
 ```bash
       <meta-data
           android:name="com.iguama.client_id"
           android:value="${client_id}" />
       <meta-data
           android:name="com.iguama.secret_key"
           android:value="${secret_key}" />
       <meta-data
           android:name="com.iguama.is_qa"
           android:value="true" />
       <meta-data
           android:name="com.iguama.icon"
           android:resource="@mipmap/ic_logo" />
```

 Parameters | Type/Required | Description
 ----- | ---- | -----------
     com.iguama.client_id | string/required | Loyal program’s identity Id for SDK.
     com.iguama.secret_key | string/required |  Loyal program’s secret key  for SDK
     com.iguama.icon | file/required | logo icon.
     com.iguama.is_qa | string/required | True is for sandbox test, false is for prod evn.
   
  
   * User Authorization
    
  a.user from app
  
 ```bash
     IguamaSDK iguamaSDK = new IguamaSDK.Builder(getApplication())
                    .userId("userId")
                    .firstName("firstName")
                    .lastName("lastName")
                    .email("email")
                    .balance(10000)
                    .accessToken("xxxxxx")
                    .build();
                    
           iguamaSDK.launch();
 ```
  
  b.user from website
```bash
    IguamaSDK iguamaSDK = new IguamaSDK.Builder(getApplication());
                        
               iguamaSDK.launch();
 ```
  
  Parameters | Type/Required | Description
  ----- | ---- | -----------
  userId | string/required | User's member id.
  firstName | string/required | User's first name.
  lastName | string/optional | User's last name.
  email | string/optional | User's email.
  balance | int/required | User's available balance.
  accessToken | string/required | Access token need to be verified when redeem. 
     
## Authentication

The Plaform handles user authentication through OAuth2 authorization framework, so the authorization itself happens within the Loyalty program's website, The Platform interacts with loyalty program's technology to obtain the authorization grant needed to access the user's account. Further information about OAuth2 can be found [here](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2).

The complete authentication process is described below:

1. The Platform will redirect it's users to the loyalty program's website to start the authentication flow, within this URL redirect, we will send the following parameters:

Field | Type | Description
----- | ---- | -----------
client_id | String | The Platform's clientID (assigned by the loyalty program's platform)
redirect_uri | URI | URI where the service redirect's the user-agent after an authorization code is granted
scope | Enum (READ, REDEEM) | Specifies the level of access that The Platform is requesting 


2. The Platform will handle the callback request, this request should contain at least the following information:

Field | Required | Description
----- | -------- | -----------
authorization_code | Yes | Authorization code needed to obtain the access token

3. The Platform will request an access token from the Loyalty Program's API, by passing the authorization code along with authentication details, including the following information:

Below is an example of the API call to obtain the access token with `grant_type=AUTHORIZATION_CODE`

```bash
curl -X POST \
  https://api.loyaltyprogram.com/oauth/token \
  -H 'Content-Type: application/json' \
  -d '{
	"client_id": "$CLIENT_ID",
	"client_secret": "$CLIENT_SECRET",
	"grant_type": "AUTHORIZATION_CODE",
	"code": "$CODE"
}'
```

Field | Type | Description
----- | ---- | -----------
client_id | String | The Platform's clientID (assigned by the loyalty program's platform)
client_secret | String | The Platform's clientSecret (assigned by the loyalty program's platform)
code | String | The authorization code received in the previous step
grant_type | Enum (AUTHORIZATION_CODE, CLIENT_CREDENTIALS) | Specifies the grant_type used to request authorization, in this case, AUTHORIZATION_CODE will be sent.

**Expected response**

```json
{
    "access_token": "095240ff-d12a-4fd6-bc58-b16bac8e7bfd",
    "refresh_token": "095240ff-d12a-4fd6-bc58-b16bac8e7bfd",
    "expires_in": "3600000"
}
```

Field | Type | Description
----- | ---- | -----------
access_token | String | Token needed to interact with the Loyalty Program's API to request user's information.
refresh_token | String | Token that can be used to request a new access token, once the current one has expired.
expires_in | Integer | Time to Live of the issued token in miliseconds (Other time units can be used, though miliseconds is recommened).

4. Using the access token obtained in the previous step, The Platform will get the user's basic information from the Loyalty Program's API, below you will find an example cURL call and the minimum required information:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" "https://api.loyaltyprogram.com/me"
```

**Expected response**

```json
{
    "id": "123ABC098",
    "email": "john.doe@email.com",
    "first_name": "John",
    "last_name": "Doe",
    "available_balance": 1000
}
```

Field | Type | Description
----- | ---- | -----------
id | String | User's id within the loyalty program's platform
email | String | Member's email
first_name | String | User's first name.
last_name | String | User's last name.
available_balance | Integer | User's reward points balance.

After this API call, the authentication flow is complete.

## Redemption

Once the user has placed the order, The Platform will start the redemption process with goal of allocate the necessary funds to eCard provided.

Within our funds allocation process, we redeem the appropriated amount of points from the user's account.

In this process the redemption API is called.

Below is an example of the API call that The Platform will perform:

```bash
curl -X POST \
  https://api.loyaltyprogram.com/redemptions \
  -H 'Authorization: Bearer $ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
        "member_id": "123ABC098",
        "total": "1000",
        "description": "Amazon Order #1344-2342344234-2334",
        "transaction_id": "[our transaction ID]"
    }'
```

Field | Type | Description
----- | ---- | -----------
member_id | String | User's id within the loyalty program's platform
total | Integer | Total amount of reward points to redeem.
description | String | Redemption's description.
transaction | String | Our transaction id for reference.

**Expected response**
```json
{
    "id": "123ABC098"
}
```

Field | Type | Description
----- | ---- | -----------
id | String | Id of the redemption

## Redemption Reversal

The Platform may need to revert a redemption (partial refunds should be supported) in three cases:
1. The user requests it. (Restrictions apply)
2. The redemption is flagged as fraudulent.
3. The digital merchant issued a refund to the user's eCard, due to guarantee claim, post-order issues, etc.

In these cases, The Platform will perform the following steps:
1. Obtain an access token through the `$CLIENT_CREDENTIALS`provided during the merchant setup.
2. Call the redemption reveral API.

Below is an example of the API call to obtain the access token with `grant_type=CLIENT_CREDENTIALS`

```bash
curl -X POST \
  https://api.loyaltyprogram.com/oauth/token \
  -H 'Content-Type: application/json' \
  -d '{
	"client_id": "$CLIENT_ID",
	"client_secret": "$CLIENT_SECRET",
	"grant_type": "CLIENT_CREDENTIALS",
	"code": "$CLIENT_CREDENTIALS"
}'
```

Field | Type | Description
----- | ---- | -----------
client_id | String | The Platform's clientID (assigned by the loyalty program's platform)
client_secret | String | The Platform's clientSecret (assigned by the loyalty program's platform)
grant_type | Enum (AUTHORIZATION_CODE, CLIENT_CREDENTIALS) | Specifies the grant_type used to request authorization, in this case, CLIENT_CREDENTIALS will be sent.

```bash
curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" "https://api.loyaltyprogram.com/redemptions/:id?total=REWARDS_POINTS_REFUNDED"
```

Field | Type | Description
----- | ---- | -----------
id | String | Id of the redemption
total | Integer | Total of reward points to credit to the user's account.


Below is an example of the redemption refund API call that The Platform will perform:

```bash
curl -X DELETE -H "Authorization: Bearer ACCESS_TOKEN" "https://api.loyaltyprogram.com/redemptions/:id?total=REWARDS_POINTS_REFUNDED"
```

Field | Type | Description
----- | ---- | -----------
id | String | Id of the redemption
total | Integer | Total of reward points to credit to the user's account.

**Expected response**

```json
{
    "id": "095240ff-d12a-4fd6-bc58-b16bac8e7bfd"
}
```

Field | Type | Description
----- | ---- | -----------
id | String | Id of the redemption reversal operation, for reference purposes.

## Platform integration

There are two method available to integrate with our platform:
1. Google Chrome Extension
2. Mobile SDK (iOS & Android)

The `1.` it's the default one and the Google Chrome Extension will be provided by iguama to the loyalty program.

To integrate with our SDK, below we provide the instructions for each mobile OS.

### iOS integration

#### Import code

1. Start XCode, drag the file IguamaRedemptionsSDK.framework into your App directory in the project navigator panel.
2. Add the an `import` statement for iguama's binary framework at the of the file:

```swift
import IguamaRedemptionsSDK
```

#### Configuration
Open `AppDelegate.swif` file, and edit the following three parameters:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {        
IGInteractionMethod.config(clientId: “YOUR_CLIENT_ID”, clientKey: “YOUR_CLIENT_KEY”, sdkLogoImage: sdkLogoImge, isSandbox: false)
        return true
}
```

Field | Type | Description
----- | ---- | -----------
clientId | String/required | Loyalty program's identify for SDK.
clientKey | String/required | Loyalty program's secret key for SDK.
logoImage | File/required | Loyalty program's logo.
isSandbox | Boolean/required | Indicates if it should behave as test environment.

### Android integration

#### Import code
1. Import iguama_sdk.aar into libs folder within your project, as shown in the figure below. Go to the project's Java Build Path and import iguama_sdk.aar under libs. Select Order and Export, and check iguama_sdk.aar.

![Import iguama_sdk.aar into libs folder within your project](https://raw.githubusercontent.com/iguama/iguama.github.io/master/media/android-integration-figure-1.png)

2. Add the following content to your project's main build.gradle to make the libs directory a dependent repository:

```build.gradle
allprojects {     
    repositories {                
        flatDir {             
            dirs 'libs'         
        }          
    } 
}
```

3. In the build.gradle file of your app module, add the iguama_sdk as a project dependency:

```app/build.gradle
dependencies {    
  implementation (name: 'iguama_sdk', ext: 'aar') 
}
```

#### Configuration

Add the following snippet in your `AndroidManifest.xml` to config the SDK.

```AndroidManifest.xml
<meta-data
  android:name="com.iguama.client_id"
  android:value="${client_id}" />
<meta-data
  android:name="com.iguama.secret_key"
  android:value="${secret_key}" />
<meta-data
  android:name="com.iguama.is_sandbox"
  android:value="true" />
<meta-data
  android:name="com.iguama.logoImage"
  android:resource="@mipmap/ic_logo" />
```

Field | Type | Description
----- | ---- | -----------
clientId | String/required | Loyalty program's identify for SDK.
clientKey | String/required | Loyalty program's secret key for SDK.
logoImage | File/required | Loyalty program's logo.
isSandbox | Boolean/required | Indicates if it should behave as test environment.
