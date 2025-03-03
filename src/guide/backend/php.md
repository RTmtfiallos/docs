# PHP

This PHP implementation is compatible with PHP 5.6 and above. However, it is important to note that the curl library may not be available in older versions of PHP. Therefore, you may need to update your PHP installation if you are using an older version. A [register](https://docs.passwordless.dev/guide/api/#register-token) function might look something like:

```php
<?php

// Define the API secret key used for authentication with the remote server
$API_SECRET = "YOUR_API_SECRET";

// Function to generate a random integer value
function get_random_int() {
  // Multiply the random number generated by mt_rand() by 1e9 (one billion) and return the integer value
  return intval(1e9 * mt_rand());
}

// Function to create a token using the given alias
function create_token($alias) {
  global $API_SECRET; // Make the API secret accessible inside the function

  // Prepare the payload for the API request, including the user ID, username, and aliases
  $payload = array(
    "userId" => get_random_int(),
    "username" => $alias,
    "aliases" => array($alias)
  );

  // Initialize a cURL session
  $ch = curl_init();
  // Set the URL of the remote server
  curl_setopt($ch, CURLOPT_URL, "https://v4.passwordless.dev/register/token");
  // Indicate that this is a POST request
  curl_setopt($ch, CURLOPT_POST, 1);
  // Attach the JSON-encoded payload as the POST data
  curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
  // Set the HTTP headers, including the API secret and content type
  curl_setopt($ch, CURLOPT_HTTPHEADER, array("ApiSecret: $API_SECRET", "Content-Type: application/json"));
  // Return the response as a string instead of printing it
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
  // Optionally, disable SSL certificate verification (not recommended for production)
  // curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

  // Execute the cURL request and store the response
  $response = curl_exec($ch);

  // Check for cURL errors and print them if any
  if (curl_errno($ch)) {
      print("cURL error: " . curl_error($ch) . "\n");
  }

  // Get the HTTP response code
  $response_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);

  // Close the cURL session
  curl_close($ch);

  // Print the API response code
  print("passwordless api response: " . $response_code . "\n");

  // Decode the JSON response into an associative array
  $response_data = json_decode($response, true);
  print_r($response_data);

  // Check if the response code is 200 (Success) and print the received token
  if ($response_code == 200) {
    print("received token: " . $response_data["token"] . "\n");
  } else {
    // Handle or log any API error
    // Log or handle the error as needed in this block
  }

  // Return the response data
  return $response_data;
}

// Check if the script is executed with exactly one argument
if ($argc == 2) {
  // Get the alias from the command-line argument
  $alias = $argv[1];
  // Call the create_token function with the alias and store the response
  $response_data = create_token($alias);
} else {
  // Print the usage instruction if the script is not executed with the correct number of arguments
  print("Usage: create_token.php <alias>\n");
}
?>
```