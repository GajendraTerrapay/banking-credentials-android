# TerraPay Banking SDK for Android

## Platform
* **Language/Environment**: Kotlin / Android SDK
* **Minimum SDK**: API 26 (Android 8.0)
* **Target SDK**: API 36 (Android 14)
* **JVM Target**: 11
* **Gradle (AGP)**: 8.13.2+

## Overview
The TerraPay Banking SDK enables seamless integration of virtual banking operations directly into your Android application.

### Key Capabilities
* **Credential Generation**: Dynamically issue virtual banking credentials for users.
* **Credential Retrieval**: Fetch details of existing banking instruments safely.
* **Lifecycle Management**: Suspend, reactivate, or close active credentials directly via client actions.
* **Core Mechanisms**: Automatic JWT token refreshing, real-time input verification, plug-and-play UI flows, and descriptive error handling wrappers.

⚠️ **Important:** To protect sensitive infrastructure, you must implement TerraPay's token exchange flow on your backend before initializing the client SDK.

---

## Installation

### Option 1: Remote Repository (Recommended)
Add the Git Maven repository to your project-level settings or build file (`dependencyResolutionManagement` in `settings.gradle.kts` or `build.gradle.kts`):

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://TerraPay.banking") } // Replace with actual Git Maven URL
    }
}
```

Then, add the dependency into your app-level `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.terrapay:banking-sdk:1.0.0")
}
```

### Option 2: Manual AAR Integration
1. Download the `TerraPayBankingSDK.aar` binary archive from the repository Releases section.
2. Place the file inside your application module's local directory: `app/libs/`.
3. Declare the local dependency in your app-level `build.gradle.kts`:

```kotlin
dependencies {
    implementation(files("libs/TerraPayBankingSDK.aar"))
}
```

---

## Authentication Setup

🔒 **Security Baseline:** Never expose plain text corporate API tokens inside client software distributions. Always broker upstream calls through an enterprise backend middleware.

### Upstream Reference Implementation
Your private backend server acts as a proxy to fetch short-lived session authorization profiles from TerraPay.

* **TerraPay Auth Endpoint:**: `POST https://dev-gateway.terrapay.com/v1/auth/token`


#### Required Headers
```http
Content-Type: application/json
x-reference-id: YOUR_REFERENCE_ID
x-app-code: YOUR_APP_CODE
```

#### Request Payload
```json
{
    "username": "your_service_account_username",
    "password": "your_secure_service_account_password",
    "resource": "TPXS"
}
```

#### Downstream API Response
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "accessTokenExpInSecs": 3600,
  "refreshTokenExpInSecs": 86400,
  "tokenType": "Bearer"
}
```

---

## Mobile App Integration

Your Android app must consume the token payload exposed by your own secure backend API route.

### Native Data Objects
Implement this parsing schema to deserialize the network payload into your client runtime context using Google Gson:

```kotlin
data class TokenResponse(
   val accessToken: String,
    val refreshToken: String,
    val accessTokenExpInSecs: Int,
    val refreshTokenExpInSecs: Int,
    val tokenType: String
)
```
## ⚠️ Security Guardrails

* **Never hardcode** TerraPay credentials inside your mobile application source code.
* **Always fetch** tokens dynamically from your secure backend infrastructure.
* **Contact support** via [sdk-support@terrapay.com](mailto:sdk-support@terrapay.com) to obtain your backend API credentials.

---

### Complete Integration Flow

Before triggering the SDK, implement a validation function to safely ensure both tokens are securely retrieved from your backend network layer:

```kotlin
/**
 * Validates and extracts backend authentication tokens.
 * Returns a Result containing the tokens or an exception if validation fails.
 */
fun validateAndGetTokens(): Result<Pair<String, String>> {
    val accessToken = getAccessTokenFromBackend()
    val refreshToken = getRefreshTokenFromBackend()

    return if (!accessToken.isNullOrBlank() && !refreshToken.isNullOrBlank()) {
        Result.success(Pair(accessToken, refreshToken))
    } else {
        Result.failure(IllegalArgumentException("Failed to fetch tokens from backend: Access or Refresh token is null or empty."))
    }    
}
```

### Step 1: Generate Credentials

### Configuration Parameters

| Parameter            | Required     | Validation Rule                                                |
|----------------------|--------------|----------------------------------------------------------------|
| dialCode             | Yes          | Must match the pattern `^\+\d+$`                               |
| currency             | Yes          | Valid ISO 4217 currency code                                   |
| appCode              | Yes          | Must not be empty                                              |
| partnerBic           | Yes          | Must not be empty                                              |
| primaryColor         | Optional     | if pass; Valid 6-digit hex code (e.g., `EC1B24`)               |
| secondaryColor       | Optional     | If pass; Valid 6-digit hex code (e.g., `FFFFFF`)               |
| termsConditionUrl    | Optional     | URL for terms and conditions                                   |
| accessToken          | Yes          | OAuth2 access token generated from partner backend             |
| refreshToken         | Yes          | OAuth2 refresh token generated from partner backend            |
| showCredentials      | Yes          | Must be boolean                                                |
| fullName             | Yes          | Must match the pattern `^[A-Za-z ]+$`                          |
| dateOfBirth          | Yes          | Must match the format `yyyy-MM-dd`                             |
| nationality          | Yes          | Valid ISO 3166-1 alpha-2 country code                          |
| phoneNumber          | Yes          | Digits only; length validated against country-specific rules   |
| country              | Yes          | Valid ISO 3166-1 alpha-2 country code                          |
| emailAddress         | Optional     | If pass; Valid email address if provided                       |
| kycIDNumber          | Yes          | Must not be empty                                              |
| kycIDType            | Yes          | Valid email address if provided                                |
| kycIDIssuedCountry   | Yes          | Valid ISO 3166-1 alpha-2 country code                          |
| kycIdIssuedDate      | Yes          | Must match the format `yyyy-MM-dd`                             |
| street               | Yes          | Must not be empty                                              |
| townName             | Yes          | Must not be empty                                              |
| state                | Yes          | Must not be empty                                              |
| postalCode           | Yes          | Must not be empty                                              |




```kotlin
LaunchedEffect(isCredentialClick) {
    if (!isCredentialClick) return@LaunchedEffect

    // 1. Define user personal demographics
    val personalInfo = TPPersonalInfo(
        fullName = "Gajendra Sharma",
        dateOfBirth = "1992-07-02",
        nationality = "KE",
        phoneNumber = "79696XXXX",
        emailAddress = "abc@gmail.com"
    )

    // 2. Define legal KYC identity documents
    val idDocument = TPIDDocument(
        kycIDNumber = "A0000000",
        kycIDType = "NATIONAL ID",
        kycIDIssuingCountry = "KE",
        kycIdIssuedDate = "2025-02-19"
    )

    // 3. Define residential address physical location attributes
    val address = TPAddress(
        street = "14 Road",
        townName = "Nairobi",
        state = "Kenya",
        postalCode = "00100",
        country = "UG"
    )

    // 4. Wrap configurations and security properties into a transaction payload
    val generateRequest = TPGenerateCredentialRequest(
        dialCode = "+XXX",
        currency = "UGX",
        partnerBic = "MMTSKXXX",
        accessToken = "qwFFHFHDHD12838448XXXXXXXXX",
        refreshToken = "ajsjdQ448XXXXXXXXX",
        primaryColor = "EC1B24",
        secondaryColor = "FFFFFF",
        showCredentials = true,
        personalInfo = personalInfo,
        idDocument = idDocument,
        address = address,
        termsConditionUrl = ""
    )

    // 5. Invoke the SDK UI sequence overlay
    TerraPayBanking.provisionCredentials(
        context = activity, // it should be Activity or ComponentActivity 
        config = generateRequest
    ) { result ->
        when (result) {
            is TPResult.success<*> -> {
                val credentials = result.response as TPGenerateCredentialResponse
                println("Account : \${credentials.accountNumber}")
            }
            is TPResult.cancelled -> {
                println("Operation Cancelled: \${result.message}")
            }
            is TPResult.failure -> {
                println("Operation Failed -> Code: result.error.errorCode, Message: {result.error.errorMessage}")
            }
        }
    }
}

```
### Step 2: Fetch Existing Credentials

### Configuration Parameters

| Parameter       | Required     | Validation Rule                                                |
|-----------------|--------------|----------------------------------------------------------------|
| dialCode        | Yes          | Must match the pattern `^\+\d+$`                               |
| phoneNumber     | Yes          | Digits only; length validated against country-specific rules   |
| appCode         | Yes          | Must not be empty                                              |
| partnerBic      | Yes          | Must not be empty                                              |
| accountNo       | Yes          | Must not be empty                                              |
| primaryColor    | Optional     | if pass; Valid 6-digit hex code (e.g., `EC1B24`)               |
| secondaryColor  | Optional     | If pass; Valid 6-digit hex code (e.g., `FFFFFF`)               |
| accessToken     | Yes          | OAuth2 access token generated from partner backend             |
| refreshToken    | Yes          | OAuth2 refresh token generated from partner backend            |
| showCredentials | Yes          | Must be boolean                                                |


```kotlin
LaunchedEffect(isFetchClick) {
    if (!isFetchClick) return@LaunchedEffect

    // 1. Configure the credential lookup parameters
    val fetchRequest = TPFetchCredentialRequest(
        dialCode = "+XXX",
        phoneNumber = "796XXXXXXX",
        bic = "TPDTXXXXXXXX",
        accountNo = "KE9441XXXXXXXXXXX",
        showcredential = true,
        accessToken = "qwFFHFHDHD12838448XXXXXXXXX",
        refreshToken = "ajsjdQ448XXXXXXXXX",
        primaryColor = "EC1B24",
        secondaryColor = "FFFFFF"
    )

    // 2. Invoke the SDK retrieval routine
    TerraPayBanking.fetchCredentials(
        context = activity,
        config = fetchRequest
    ) { result ->
        when (result) {
            is TPResult.success<*> -> {
                val credentials = result.response as TPFetchCredentialResponse
                println("Status: \${credentials.status}")
            }
            is TPResult.cancelled -> {
                println("Operation Cancelled: \${result.message}")
            }
            is TPResult.failure -> {
                println("Operation Failed -> Code: \${result.error.errorCode}, Message: \${result.error.errorMessage}")
            }
        }
    }
}
```
### Step 3: Update Credential Status

### Configuration Parameters

| Parameter       | Required     | Validation Rule                                              |
|-----------------|--------------|--------------------------------------------------------------|
| dialCode        | Yes          | Must match the pattern `^\+\d+$`                             |
| phoneNumber     | Yes          | Digits only; length validated against country-specific rules |
| appCode         | Yes          | Must not be empty                                            |
| partnerBic      | Yes          | Must not be empty                                            |
| primaryColor    | Optional     | if pass; Valid 6-digit hex code (e.g., `EC1B24`)             |
| secondaryColor  | Optional     | If pass; Valid 6-digit hex code (e.g., `FFFFFF`)             |
| accessToken     | Yes          | OAuth2 access token generated from partner backend           |
| refreshToken    | Yes          | OAuth2 refresh token generated from partner backend          |
| status          | Yes          | Must not be empty (`ACTIVE/INACTIVE`)                        |
| statusReason    | Yes          | Must not be empty                                            |
| modifiedBy      | Yes          | Must not be empty                                            |



```kotlin
LaunchedEffect(isUpdateClick) {
    if (!isUpdateClick) return@LaunchedEffect

    // 1. Configure the state management and lifecycle parameters
    val updateRequest = TPUpdateCredentialRequest(
        dialCode = "+XXX",
        phoneNumber = "796965XXX",
        bic = "AIRTUXXXXXX",
        status = "INACTIVE", // Acceptable states: SUSPENDED, CLOSED, ACTIVE, INACTIVE
        statusReason = "Mobile number recycled by telco",
        modifiedBy = "MPESA-KE-XXXX",
        accessToken = "qwFFHFHDHD12838448XXXXXXXXX",
        refreshToken = "ajsjdQ448XXXXXXXXX",
        primaryColor = "EC1B24",
        secondaryColor = "FFFFFF"
    )

    // 2. Invoke the SDK lifecycle update routine
    TerraPayBanking.updateCredentials(
        context = activity,
        config = updateRequest
    ) { result ->
        when (result) {
            is TPResult.success<*> -> {
                val credentials = result.response as TPUpdateCredentialResponse
                println("Status Updated: \${credentials.status}")
            }
            is TPResult.cancelled -> {
                println("Operation Cancelled: \${result.message}")
            }
            is TPResult.failure -> {
                println("Operation Failed -> Code: \${result.error.errorCode}, Message: \${result.error.errorMessage}")
            }
        }
    }
}
```

---


## Reference

### Credential Statuses
* **`ACTIVE`**: Account is functional and actively usable.
* **`CHURNED`**: The user has stopped using the service.
* **`SUSPENDED`**: Temporarily locked or deactivated.
* **`CLOSED`**: Account is permanently closed.

### ID Document Types
* **`PASSPORT`**: International passport document.
* **`NATIONAL_ID`**: Local national identity card.
* **`DRIVERS_LICENSE`**: Government-issued driver's license.


## Error Handling

SDK automatically validates inputs and provides descriptive errors:

```kotlin
TerraPayBanking.provisionCredentials(
    context = activity,
    config = generateRequest
) { result ->
    when (result) {
        is TPResult.success<*> -> {
            val credentials = result.response as TPGenerateCredentialResponse
            // Success - commit values locally or update UI states
            saveCredentials(credentials)
        }
        is TPResult.failure -> {
            // SDK provides distinct failure payloads containing code and explanation strings
            showAlert(code =result.error.errorCode, message = result.error.errorMessage)
        }
        is TPResult.cancelled -> {
            // User backed out of the workflow overlay execution natively — no action needed
            println("User explicitly abandoned the operational workflow: \${result.message}")
        }
    }
}

```

## Troubleshooting

### Unresolved reference / Module not found
Verify that you have correctly appended the custom repository URL inside your `settings.gradle.kts` configuration under the `dependencyResolutionManagement` routing block, or ensure that your locally downloaded `TerraPayBankingSDK.aar` binary archive is placed directly within the `/app/libs/` folder directory hierarchy.

### Thread / Main-Thread Exceptions
SDK actions containing UI components must be launched safely on the UI dispatcher. Wrap your functional triggers inside an asynchronous Jetpack Compose `LaunchedEffect(key)` loop or manually explicitly bridge your execution streams by specifying `Dispatchers.Main` context inside your lifecycle coroutine structures.

### Input Verification Blocks
The SDK validates fields locally before hitting public servers. If your parameters do not invoke responses, inspect your layout attributes to verify that hex color formatting excludes the `#` token and validation dates precisely mirror the `YYYY-MM-DD` specification schema.

---

## Support

* **Technical Support**: Direct your deployment inquiries to [dev-support@terrapay.com](mailto:dev-support@terrapay.com).
* **Documentation**: Browse deep-dive articles and technical resources at [https://docs.terrapay.com](https://docs.terrapay.com).
* **Issues**: Report unexpected framework runtime crashes or behavior variations using the repository GitHub Issues tab.

---

© 2026 TerraPay. All rights reserved.
