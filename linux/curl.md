## Curl

- Developers often find themselves needing a tool that seamlessly facilitates HTTP requests, allowing them to communicate with servers, test endpoints, and debug applications. One such tool that has stood the test of time is **cURL**. Short for _Client URL_.

- **cURL** is a command-line utility that enables users to transfer data with various protocols.

- cURL is a robust and versatile tool that supports a wide range of protocols, including HTTP, HTTPS, FTP, FTPS, SCP, SFTP, LDAP, and more.

- It is available on most Unix-like operating systems, including Linux, macOS, and even Windows, making it accessible to developers across different platforms.

- **cURL** is open-source software, maintained and actively developed by a community of contributors.

### Basic Usage

```bash
  curl https://example.com
```

### Common Use Cases

#### API Testing

Here, cURL sends a GET request to fetch user data from the API endpoint.

```bash
  curl https://api.example.com/users/1
```

#### File Transfer

This command downloads a file from the specified URL.

```bash
  curl -O https://example.com/file.zip
```

#### Authentication

This command cURL includes basic authentication credentials with the request.

```bash
  curl -u username:password https://api.example.com/resource
```

#### Verbose Output

The -v flag enables verbose output, showing detailed information about the request and response.

```bash
  curl -v https://example.com
```

#### curl virtual host in Nginx

This command uses header flag to indicate that it want the file server for example.com

```bash
curl --header "Host: example.com" localhost
```

#### HTTP POST Request

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name":"John","age":30}' https://api.example.com/users
```

Here, cURL sends a POST request with JSON data to create a new user.

#### Follow Redirects

cURL automatically follows HTTP redirects with the -L flag

```bash
curl -L https://example.com
```

cURL automatically follows HTTP redirects with the -L flag.

#### Authentication with header & URL encoded body

```bash
curl --location 'https://napi.neosecurities.com/oauth2/token' \
--header 'Authorization: Basic Z0N0WjFVNWZmbGhPMG03Nk5VMU5waHZ0Zko0YTpjZlhDMXVHS1dYcFZnSkJtUFZNRkRJV05MUm9h' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'username=sinus009' \
--data-urlencode 'password=copycat@123'
```

This cURL command sends a POST request to the specified URL https://napi.neosecurities.com/oauth2/token with the following details:

- The `Authorization` header is set with a Basic Authentication token. It seems like the token is base64 encoded, consisting of a client ID and client secret separated by a colon (:)

- The request body (`--data-urlencode`) includes the parameters required for OAuth 2.0 authentication. The --data-urlencode flag in cURL is used to specify data to be sent with a POST request where the data needs to be URL-encoded. This is particularly useful when sending form data or parameters in the body of an HTTP request.

- `grant_type=password`: This indicates that the client is sending the user's username and password directly for authentication.
- username: The username for authentication, which seems to be sinus009.

- `password`: The password corresponding to the username, which seems to be copycat@123

This command will make a request to obtain an access token using the provided credentials. If the authentication is successful, the server will respond with an access token that can be used to access protected resources.

#### Examining cURL response in terminal

To view individual parameters in a JSON response in the terminal, you can use various command-line tools available in Unix-like operating systems.

**Using `grep` and `awk`**

While grep and awk are not JSON-specific tools, they can still be used to extract specific parameters from a JSON response.

```bash
curl https://api.example.com/data | grep 'parameter1' | awk -F '"' '{print $4}'
```

This command sends a GET request to https://api.example.com/data, receives a JSON response, then uses grep to filter lines containing parameter1 and awk to extract the value of parameter1 using double quotes as the field delimiter.

#### Using Python

Python comes pre-installed on most Unix-like systems and provides a built-in module called json.tool for formatting JSON data.

```bash
curl https://api.example.com/data | python -m json.tool
```

This command sends a GET request to https://api.example.com/data, receives a JSON response, and then uses Python's json.tool module to pretty-print the JSON response, making it easier to read and analyze.

Overall, cURL is widely supported in the API development ecosystem and serves as a valuable tool for interacting with APIs in various scenarios.By mastering cURL, developers can streamline their workflows and interact with web services with confidence and efficiency.

#### References

- https://blog.gopenai.com/exploring-curl-a-versatile-command-line-tool-for-http-requests-7cd4570556c5
