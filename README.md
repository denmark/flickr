# Go Flickr API Library

## Overview

This project is a Go library designed to simplify interaction with the Flickr API. It provides a convenient way for Go developers to integrate Flickr's vast photo-sharing platform into their applications. The library handles authentication, makes requests to various API endpoints, and manages responses and errors.

## Features

*   **Authentication:** Supports OAuth 1.0a for secure authentication with the Flickr API.
*   **Comprehensive API Coverage:** Implements a wide range of Flickr API methods, including:
    *   Photos (uploading, searching, metadata)
    *   Photosets (creating, managing)
    *   People (user information, photos)
    *   And more...
*   **Error Handling:** Provides robust error handling to manage API errors gracefully.
*   **Helper Utilities:** Includes utility functions to assist with common tasks.
*   **Examples:** Comes with example code to demonstrate usage.
*   **Tests:** Includes a test suite to ensure reliability.

## Who is this for?

This library is for Go developers who want to:

*   Build applications that leverage Flickr's photo and video hosting capabilities.
*   Automate interactions with their Flickr account.
*   Access and display Flickr content within their Go projects.

## Installation

To install the Flickgo library, use `go get`:

```bash
go get github.com/azerpas/flickgo
```

Then, you can import it into your Go project:

```go
import (
	"fmt"
	"github.com/azerpas/flickgo"
)

func main() {
	// Example: Create a new client (actual usage may vary)
	// client := flickgo.NewClient("YOUR_API_KEY", "YOUR_API_SECRET")
	// Or, if you need to authenticate a user:
	// userClient, err := flickgo.NewUserClient("ACCESS_TOKEN", "ACCESS_TOKEN_SECRET")
	// if err != nil {
	// 	log.Fatal(err)
	// }
	// fmt.Println("Flickgo client initialized (example)")
	fmt.Println("Refer to the 'Authentication' and 'Usage' sections, and the 'examples/' directory for detailed usage.")
}
```
*(Please refer to the official Flickr API documentation for obtaining API keys and access tokens.)*

## Authentication (OAuth 1.0a)

This library uses OAuth 1.0a to authenticate users and authorize access to their Flickr account. You will need an API Key and API Secret from Flickr, which you can obtain by registering an application on the [Flickr App Garden](https://www.flickr.com/services/apps/create/).

The authentication process typically involves the following steps:

1.  **Initialize a Client**: Create a `FlickrClient` with your API Key and API Secret.
2.  **Get a Request Token**: Use `GetRequestToken` to obtain a request token and secret from Flickr.
3.  **User Authorization**: Redirect the user to an authorization URL obtained via `GetAuthorizeUrl`. The user logs into Flickr and authorizes your application. Flickr then provides an OAuth Verifier code (since this library uses an "oob" - out-of-band - callback, the user will need to copy this code and provide it back to your application).
4.  **Get an Access Token**: Exchange the request token, request token secret, and the OAuth Verifier for an access token and secret using `GetAccessToken`.
5.  **Authenticated Client**: Once you have the access token and secret, you can use them to make authenticated API calls on behalf of the user. The `FlickrClient` will store these tokens after a successful call to `GetAccessToken`.

Here's a conceptual example of the command-line authentication flow:

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"

	"github.com/azerpas/flickgo"
)

func main() {
	apiKey := "YOUR_FLICKR_API_KEY"
	apiSecret := "YOUR_FLICKR_API_SECRET"

	if apiKey == "YOUR_FLICKR_API_KEY" || apiSecret == "YOUR_FLICKR_API_SECRET" {
		log.Fatal("Please replace YOUR_FLICKR_API_KEY and YOUR_FLICKR_API_SECRET with your actual credentials.")
	}

	// 1. Initialize a client
	client := flickgo.NewFlickrClient(apiKey, apiSecret)
	// For a desktop app, flickgo uses an out-of-band (oob) callback for simplicity.
	// client.CallbackUrl = "oob" // GetRequestToken sets this by default

	// 2. Get a Request Token
	fmt.Println("Getting request token...")
	requestToken, err := flickgo.GetRequestToken(client)
	if err != nil {
		log.Fatalf("Error getting request token: %v", err)
	}
	if requestToken.OAuthProblem != "" {
		log.Fatalf("OAuth problem when getting request token: %s", requestToken.OAuthProblem)
	}

	fmt.Printf("Request Token: %s\nRequest Token Secret: %s\n", requestToken.OauthToken, requestToken.OauthTokenSecret)

	// 3. User Authorization
	authorizeURL, err := flickgo.GetAuthorizeUrl(client, requestToken)
	if err != nil {
		log.Fatalf("Error getting authorize URL: %v", err)
	}
	fmt.Printf("\nGo to the following URL to authorize the application:\n%s\n\n", authorizeURL)
	fmt.Print("Enter the OAuth Verifier code from Flickr: ")
	verifier, _ := bufio.NewReader(os.Stdin).ReadString('\n')
	verifier = string(verifier[:len(verifier)-1]) // Remove newline

	// 4. Get an Access Token
	fmt.Println("Getting access token...")
	accessToken, err := flickgo.GetAccessToken(client, requestToken, verifier)
	if err != nil {
		log.Fatalf("Error getting access token: %v", err)
	}
	if accessToken.OAuthProblem != "" {
		log.Fatalf("OAuth problem when getting access token: %s", accessToken.OAuthProblem)
	}

	fmt.Printf("\nAuthentication successful!\n")
	fmt.Printf("Access Token: %s\n", accessToken.OAuthToken)
	fmt.Printf("Access Token Secret: %s\n", accessToken.OAuthTokenSecret)
	fmt.Printf("User NSID: %s\n", accessToken.UserNsid)
	fmt.Printf("Username: %s\n", accessToken.Username)

	// 5. Authenticated Client
	// The 'client' object is now authenticated and can be used to make API calls.
	// Its OAuthToken, OAuthTokenSecret, and Id fields are populated.
	client.OAuthToken = accessToken.OAuthToken
	client.OAuthTokenSecret = accessToken.OAuthTokenSecret
	client.Id = accessToken.UserNsid
	fmt.Printf("\nClient is now authenticated for user: %s (%s)\n", client.Id, accessToken.Username)
}
```
**Note on Permissions**: The `GetAuthorizeUrl` function in this library currently requests "delete" permissions. You might need to modify the library if you require different permission levels (e.g., "read" or "write").

After obtaining the `OAuthToken` and `OAuthTokenSecret`, you should store them securely. For subsequent sessions, you can initialize a `FlickrClient` and directly set its `OAuthToken` and `OAuthTokenSecret` fields (along with `ApiKey` and `ApiSecret`) to make authenticated calls without repeating the OAuth dance.

## Usage

Once you have an authenticated `flickgo.FlickrClient`, you can use it to interact with various Flickr API endpoints. The library is structured into modules (like `photos`, `photosets`, `people`) that provide specific functionalities.

Here are a few examples:

### Initializing a Client (with existing tokens)

If you have already completed the OAuth flow and stored the tokens:

```go
import (
	"github.com/azerpas/flickgo"
	// Import specific modules you need, e.g.:
	// "github.com/azerpas/flickgo/photos"
	// "github.com/azerpas/flickgo/photosets"
)

func getAuthenticatedClient() *flickgo.FlickrClient {
	apiKey := "YOUR_FLICKR_API_KEY"
	apiSecret := "YOUR_FLICKR_API_SECRET"
	oauthToken := "YOUR_STORED_OAUTH_TOKEN"
	oauthTokenSecret := "YOUR_STORED_OAUTH_TOKEN_SECRET"
	// userNsid := "YOUR_STORED_USER_NSID" // Optional: client.Id can be set if needed

	client := flickgo.NewFlickrClient(apiKey, apiSecret)
	client.OAuthToken = oauthToken
	client.OAuthTokenSecret = oauthTokenSecret
	// client.Id = userNsid // Set the user's NSID on the client
	return client
}
```

### Getting Information About a Photo

To get details for a specific photo, use the `photos.GetInfo` function:

```go
import (
	"fmt"
	"log"
	"github.com/azerpas/flickgo"
	"github.com/azerpas/flickgo/photos"
)

func main() {
	client := getAuthenticatedClient() // Your function from above

	photoID := "some_photo_id" // Replace with an actual photo ID
	// The 'secret' argument can be empty for public photos or if you are the owner.
	photoInfoResp, err := photos.GetInfo(client, photoID, "")
	if err != nil {
		log.Fatalf("Error getting photo info: %v", err)
	}

	if photoInfoResp.Stat == "ok" {
		fmt.Printf("Photo Title: %s\n", photoInfoResp.Photo.Title)
		fmt.Printf("Description: %s\n", photoInfoResp.Photo.Description)
		fmt.Printf("Views: %d\n", photoInfoResp.Photo.Views)
	} else {
		log.Fatalf("Flickr API error getting photo info: %s (Code: %d)", photoInfoResp.Message, photoInfoResp.Code)
	}
}
```

### Uploading a Photo

To upload a photo, use the `flickgo.UploadFile` function. You can specify parameters like title, description, and tags using `flickgo.NewUploadParams`.

```go
import (
	"fmt"
	"log"
	"github.com/azerpas/flickgo"
)

func main() {
	client := getAuthenticatedClient() // Your function from above

	filePath := "path/to/your/photo.jpg" // Replace with the actual path to your image

	params := flickgo.NewUploadParams()
	params.Title = "My Awesome Photo"
	params.Description = "Uploaded with flickgo!"
	params.IsPublic = true // Make it public
	params.Tags = []string{"go", "flickr", "api", "test"}

	uploadResp, err := flickgo.UploadFile(client, filePath, params)
	if err != nil {
		// Note: The error from UploadFile might be a general error.
		// The UploadResponse itself can also contain an error message from Flickr.
		log.Fatalf("Error uploading photo: %v. Response: %+v", err, uploadResp)
	}

	if uploadResp.Stat == "ok" {
		fmt.Printf("Photo uploaded successfully! Photo ID: %s\n", uploadResp.ID)
	} else {
		log.Fatalf("Flickr API error during upload: %s (Code: %d)", uploadResp.Message, uploadResp.Code)
	}
}
```

### Exploring Other Functionality

This library provides various other functions within its modules:
*   **`github.com/azerpas/flickgo/photosets`**: For managing photo albums (e.g., `photosets.GetList`, `photosets.GetInfo`, `photosets.Create`).
*   **`github.com/azerpas/flickgo/people`**: For user-related information.
*   ...and more.

Browse the subdirectories of `github.com/azerpas/flickgo` to discover available modules and functions. The function signatures and required parameters will guide you in their usage.

## Examples Directory

The `examples/` directory in this repository contains more extensive sample applications.

**Note:** Some examples in the `examples/` directory might directly use components or client initialization patterns from the `github.com/denmark/flickr` library. While `flickgo` utilizes `github.com/denmark/flickr` for some underlying API call executions, you should primarily interact with the `flickgo.FlickrClient` and the functions/modules provided directly by `github.com/azerpas/flickgo` (like `flickgo.UploadFile`, `flickgo/photos.GetInfo`, etc.) as shown in the "Usage" section above.

## Contributing

We welcome contributions to the `flickgo` library! Whether it's reporting a bug, suggesting an improvement, or submitting new code, your help is appreciated.

### Reporting Issues

*   If you encounter a bug or have a feature request, please open an issue on the [GitHub Issues page](https://github.com/azerpas/flickgo/issues).
*   Provide as much detail as possible, including steps to reproduce the bug, expected behavior, and actual behavior.

### Making Changes

1.  **Fork the Repository**: Start by forking the `azerpas/flickgo` repository to your own GitHub account.
2.  **Create a Branch**: Create a new branch in your fork for your changes. Choose a descriptive branch name (e.g., `feature/add-new-endpoint` or `fix/oauth-bug`).
    ```bash
    git checkout -b your-branch-name
    ```
3.  **Implement Your Changes**: Make your modifications to the code.
    *   Follow existing coding conventions and style.
    *   Ensure your code is well-commented, especially for new functionalities.
    *   If you add new features, consider adding corresponding examples or updating existing ones.
4.  **Run Tests**: Before submitting your changes, ensure all tests pass. The project includes a script to run tests:
    ```bash
    ./runtests.sh
    ```
    If you add new functionality, please include relevant unit tests.
5.  **Commit Your Changes**: Commit your changes with a clear and concise commit message.
    ```bash
    git commit -m "feat: Add support for XYZ endpoint"
    ```
6.  **Push to Your Fork**: Push your branch to your fork on GitHub.
    ```bash
    git push origin your-branch-name
    ```
7.  **Submit a Pull Request (PR)**: Open a pull request from your branch to the `master` branch (or the appropriate target branch) of the `azerpas/flickgo` repository.
    *   Provide a clear description of your changes in the PR.
    *   Reference any related issues.

Thank you for contributing to `flickgo`!

## License

This project is licensed under the MIT License. For the full license text, please see the [LICENSE.md](LICENSE.md) file.
